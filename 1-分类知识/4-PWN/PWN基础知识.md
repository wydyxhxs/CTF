### checksec
```python
pwn checksec XXX
```
![](https://cdn.jsdelivr.net/gh/wydyxhxs/images/pic/20250723091757090.png)


1. **Arch**: 显示目标二进制文件的架构类型。在此例中为 `i386-32-little`，意味着该文件是32位的小端架构。
    
2. **RELRO**: 表示“读取只读重定位”。这是一种防止二进制文件在加载时被恶意修改的技术。`Partial RELRO` 表示部分启用了此保护机制。
    
3. **Stack**: 显示栈保护状态。`No canary found` 意味着栈保护机制没有启用，可以被利用进行缓冲区溢出攻击。
    
4. **NX**: 表示“不可执行”位（No-eXecute）。`NX enabled` 表示启用了此保护，防止代码在数据区域（如栈）执行，从而减少缓冲区溢出的风险。
    
5. **PIE**: 可执行文件是否启用了位置独立执行（Position Independent Executable）。`No PIE` 表示该二进制没有启用此保护，攻击者可以利用已知地址进行攻击。
    
6. **SHSTK**: 栈保护状态。`Enabled` 表示启用了栈保护。
    
7. **IBT**: 表示启用了“无限制分支目标跟踪”（Indirect Branch Tracking）。`Enabled` 表示启用此保护，增加对间接跳转的控制，防止利用跳转表攻击。
    
8. **Stripped**: 表示可执行文件是否去除了符号表。`No` 表示没有去除符号表，这通常有助于调试和逆向分析。