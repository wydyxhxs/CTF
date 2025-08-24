# reverse & pwn

## UPX

### 工具脱壳

```bash
upx -d xxx.exe
```

### 手动脱壳

#### ESP定律

个人认为这个方法是最方便，最快捷的，掌握这个方法基本的UPX壳闭眼脱

##### 原理

ESP 定律的原理在于利用程序中堆栈平衡来快速找到 OEP.

由于在程序自解密或者自解压过程中, 不少壳会先将当前寄存器状态压栈, 如使用`pushad`, 在解压结束后, 会将之前的寄存器值出栈, 如使用`popad`. 因此在寄存器出栈时, 往往程序代码被恢复, 此时硬件断点触发. 然后在程序当前位置, 只需要少许单步操作, 就很容易到达正确的 OEP 位置.

1. 程序刚载入开始 pushad/pushfd
2. 将第一个寄存器压入栈后就设对 ESP 寄存器设硬件断点
3. 运行程序, 触发断点，这是upx程序几乎结束

##### 例题讲解

ida反编译一个加upx壳的发现函数很少，但是可以发现，start函数开始有很多push操作

![image-20250115225432603](https://cdn.jsdelivr.net/gh/p0ach1l/Picture@master/test/image-20250115225432603.png)

打开x64dbg进行调试，指向到第二个push停止

![image-20250115225742755](https://cdn.jsdelivr.net/gh/p0ach1l/Picture@master/test/image-20250115225742755.png)

然后再右下栈窗口给esp下硬件访问断点，64位8字节，32位4字节

![image-20250115225858495](https://cdn.jsdelivr.net/gh/p0ach1l/Picture@master/test/image-20250115225858495.png)

接下来直接运行，就会断在upx壳快结束的位置

![image-20250115230107554](https://cdn.jsdelivr.net/gh/p0ach1l/Picture@master/test/image-20250115230107554.png)

接着一直往下运行，直到一个大的跳转，就是到达OEP

![image-20250115230300395](https://cdn.jsdelivr.net/gh/p0ach1l/Picture@master/test/image-20250115230300395.png)

到达OEP后就用x64dbg自带的插件进行dump和IAT的修复

![image-20250115230506018](https://cdn.jsdelivr.net/gh/p0ach1l/Picture@master/test/image-20250115230506018.png)

在第四步的时候，如果出现红色×就删除掉

![image-20250115230603184](https://cdn.jsdelivr.net/gh/p0ach1l/Picture@master/test/image-20250115230603184.png)

上边步骤已经完成了dump和IAT的修复，但是可能程序仍然不能运行

这时候我们就要去除重定位

我们需要修改其PE结构中的两个字段值：

1. File Header 的 Charateristics
2. Optional Header的 DllCharateristics

下面用小辣椒(CFF Explorer VII)进行修复

第三步勾选上

![image-20250115231709951](https://cdn.jsdelivr.net/gh/p0ach1l/Picture@master/test/image-20250115231709951.png)

第三步取消勾选

![image-20250115231854920](https://cdn.jsdelivr.net/gh/p0ach1l/Picture@master/test/image-20250115231854920.png)

这样一个完整的脱壳流程就完成了

 ## SMC

### 原理详解

自修改代码（Self-Modifying Code），指在一段代码执行前对它进行修改。把代码以加密的形式保存 在可执行文件中（或静态资源中），然后在程序执行的时候进行动态解密。这样我们在采用静态分析 时，看到的都是加密的内容，从而减缓甚至阻止静态分析。

先来看一段展示 SMC 思路的伪代码：

```c
if (运行条件满足) {
  DecryptProc(Address of Check) // 对 Check 代码解密
// ........  
  Check();                      // 调用 Check  
// ........  
  EncryptProc(Address of Check) // 再对代码进行加密，防止程序被 dump
}
```

下面来打造一个使用 SMC 进行静态分析对抗的示例，首先正常写一个程序：

```c
#include <stdio.h>
#include <windows.h>
int check(int in) {
    return in == 12345;
}
void decrypt() {
    DWORD old;
    VirtualProtect(check, 4096, PAGE_EXECUTE_READWRITE, &old);
    for (int i = 0; i < 36; i++)
    {
        *((char*)check + i) ^= 0x90;
    }
    VirtualProtect(check, 4096, old, NULL);
}
int main() {
    int input;
    scanf("%d", &input);     
    decrypt(); 
    if (check(input)) 
        printf("good!");
}
```

把check提取出来进行加密

![image-20250807140122305](https://cdn.jsdelivr.net/gh/p0ach1l/Picture@master/test/20250807140122407.png)

加密程序

```python
check = [0x55, 0x8B, 0xEC, 0x51, 0x81, 0x7D, 0x08, 0x39, 0x30, 0x00, 0x00, 0x75, 0x09, 0xC7, 0x45, 0xFC, 0x01, 0x00, 0x00, 0x00, 0xEB, 0x07, 0xC7, 0x45, 0xFC, 0x00, 0x00, 0x00, 0x00, 0x8B, 0x45, 0xFC, 0x8B, 0xE5, 0x5D, 0xC3]
print(len(check))
for i in range(len(check)) :
    print(hex(check[i] ^ 0x90) , end= " ")
```

查看几个字节，需改源程序解密位数，编译用010打开并替换check函数

![image-20250807134036722](https://cdn.jsdelivr.net/gh/p0ach1l/Picture@master/test/20250807134036818.png)

程序仍然可以正常运行，但是check函数已经进行加密

![image-20250807140412057](https://cdn.jsdelivr.net/gh/p0ach1l/Picture@master/test/20250807140412151.png)

### 对抗思路

能动态调试最好直接动态调试，因为在程序运行的某一时刻，它一定是解密完成的，这时也就暴露 了，使用动态分析运行到这一时刻即可过掉保护。
其次是根据静态分析获得解密算法，写出解密脚本提前解密这段代码。 解密得到的机器码可以通过 IDAPython 的 patch_byte 接口很方便地写回

还原SMC代码的idapython模板如下：

```python
import idc
import idaapi
import idautils

# ========= 用户配置 =========
start_ea = 0x00401000  # 起始地址
end_ea   = 0x00401024  # 结束地址
key = 0x90           # 解密的异或密钥
# ===========================

def decrypt_byte(byte, key):
    """解密函数"""
    return byte ^ key

def restore_smc(start, end, key):
    size = end - start
    encrypted_bytes = [idc.get_wide_byte(ea) for ea in range(start, end)]

    decrypted_bytes = [decrypt_byte(b, key) for b in encrypted_bytes]

    print(f"[+] 恢复字节范围: 0x{start:X} - 0x{end:X}")
    print(f"[+] 加密字节: {[hex(b) for b in encrypted_bytes]}")
    print(f"[+] 解密字节: {[hex(b) for b in decrypted_bytes]}")

    # 覆盖原字节
    for i, b in enumerate(decrypted_bytes):
        idc.patch_byte(start + i, b)

    # 删除旧反汇编
    idc.del_items(start, idc.DELIT_SIMPLE, size)

    # 重新创建指令
    ea = start
    while ea < end:
        insn_len = idc.create_insn(ea)
        if insn_len == 0:
            print(f"[!] 无法反汇编 0x{ea:X}")
            ea += 1
        else:
            ea += insn_len

    print("[+] SMC 区域还原完成 ✅")
    if not idaapi.add_func(start, end):
        print(f"[!] 添加函数失败: 0x{start:X}")
    else:
        print(f"[+] 函数已自动创建: 0x{start:X} - 0x{end:X}")

# 执行脚本
restore_smc(start_ea, end_ea, key)
```

## 反调试

### 1. 程序功能概览

- 程序逻辑：要求输入密码 → 如果正确，再进行一系列「反调试检测」
- 密码检查：`strncmp(Str1, "I have a pen.", 0xD)`

### 2. 常见反调试手法讲解（结合代码逐条拆解）

#### (1) **API 检测调试器**

```c
if (IsDebuggerPresent()) { ... }
CheckRemoteDebuggerPresent(GetCurrentProcess(), pbDebuggerPresent);
```

- **原理**：Windows 提供的 API，可以直接判断进程是否处于调试状态。
- **绕过思路**：修改返回值（hook API），或者打补丁跳过逻辑。

------

#### (2) **PEB 检测 NtGlobalFlag**

```c
if (sub_401120() == 112) { ... }
```

- **原理**：PEB（进程环境块）中 `NtGlobalFlag` 会在调试状态下被设置。
- **绕过思路**：修改 PEB 内存数据（调试器脚本里常用）。

------

#### (3) **时间检测 (GetTickCount)**

```c
TickCount = GetTickCount();
if (GetTickCount() - TickCount > 0x3E8) { ... }
```

- **原理**：调试过程中如果设置断点或单步，会消耗过多时间。
- **绕过思路**：Patch 时间判断，或者使用调试器插件隐藏延时。

------

#### (4) **检测工具存在 (Procmon)**

```c
CreateFileA("\\\\.\\Global\\ProcmonDebugLogger", ...)
```

- **原理**：某些工具（Procmon）会注册全局设备对象，可以直接探测。
- **绕过思路**：阻止 CreateFile 成功 / 改名工具。

------

#### (5) **检测调试器进程**

```c
switch (sub_401130()) {
  case 1: printf("But detected Ollydbg.\n"); break;
  case 2: printf("But detected ImmunityDebugger.\n"); break;
  case 3: printf("But detected IDA.\n"); break;
  case 4: printf("But detected WireShark.\n"); break;
}
```

- **原理**：通过进程枚举，检查调试器/抓包工具是否运行。
- **绕过思路**：修改检测函数返回值。

------

#### (6) **检测虚拟机环境**

```c
if (sub_401240() == 1) { printf("But detected VMware.\n"); exit(1); }
```

- **原理**：检测特定的硬件驱动、寄存器特征。
- **绕过思路**：虚拟机插件隐藏、直接 patch。

------

#### (7) **异常检测**

```c
pbDebuggerPresent[4] = 1 / 0; // 除零异常
```

- **原理**：调试器通常会捕获异常，如果程序自己处理 → 能检测到调试存在。
- **绕过思路**：Patch 掉异常语句。

## 堆

堆块格式

![image-20250822142512464](https://cdn.jsdelivr.net/gh/p0ach1l/Picture@master/test/20250822142519594.png)

下面解释各个字段的含义。
1. prev_size
  顾名思义，prev_size记录的是前一个堆块的大小。但实际上，这个字段 有如下两种情况：如果前一个物理地址相邻的chunk（即地址更小的 chunk）未使用，即已经被释放（free），prev_size会记录前一个chunk 的大小。否则，这个字段有可能被用来存储前一个chunk的数据，这种 情况称为空间复用。
2. .size
  首先，size字段是要求对齐的。在Linux系统中，size是2＊SIZE_SZ的整 数倍；在32位程序中，SIZE_SZ=4；在64位程序中，SIZE_SZ=8。所有用户程序申请的大小都会向上取整，比如在32位程序中，用户申请的 大小为0x14，那么向上取整后为0x18。
  这只是一个简单的例子，实际上，在分配的时候，因为有空间复用的 问题，所以分配0x14字节的chunk的时候，结果会和预想的不一样。这 个知识点会在后续讲解空间复用时详细说明，这里不深入探讨。 因此，在32位程序中，申请的所有chunk的大小都是2＊SIZE_SZ的倍 数，即8的倍数；在64位程序中，chunk的大小都是16的倍数。
  当把这些数字以二进制形式写出来的时候，会发现一个特点：
  ❑32位：0b1000,0b10000,0b11000...
  ❑64位：0b10000,0b100000,0b110000...
  这些数字的一个明显的特点是，最后三位都是0，所以ptmalloc的开发 者将最低的三位用于存储三个标志位，分别是A|M|P，从高位到低位， 其对应的解释如下：
  1）A：NON_MAIN_ARENA标志位，表示这个chunk是否属于主线 程，1表示属于，0表示不属于。
  2）M：IS_MAPPED标志位，表示这个chunk是否由mmap系统调用分 配，1表示由mmap系统调用分配，0表示不由mmap系统调用分配，即 由brk系统调用分配。
  3）P：PREV_INUSE标志位，表示这个chunk的物理地址相邻的前一个 chunk是否在使用中，1表示在使用中，即没有被释放，0表示不在使用 中，即已经被释放了。若前一个chunk已经被释放，可以通过prev_size 字段来获取前一个chunk的大小，ptmalloc也会通过prev_size来计算对应 的地址。

### fastbin

![image-20250822142730201](https://cdn.jsdelivr.net/gh/p0ach1l/Picture@master/test/20250822142730278.png)

 ### unsorted bin

![image-20250822142753080](https://cdn.jsdelivr.net/gh/p0ach1l/Picture@master/test/20250822142753131.png)
