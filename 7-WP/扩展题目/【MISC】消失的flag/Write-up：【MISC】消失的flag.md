
#wp #misc

【[[MISC]]】【MISC】消失的flag - [[扩展题目]]

创建时间：2025-07-06 09:10

# 消失的flag Write-up

> **题目类型**：Misc / Forensics  
> **题目文件**：flags.zip（包含 flags.txt，10000 条候选 flag）  
> **最终 Flag**：`flag{351e3c15-7b89-4208-af54-e29b12b9d195}`

---

[[flags.zip]]

## 一、题目分析

解压附件后得到一个 `flags.txt` 文件，包含大量格式规范的 UUID：

```
flag{xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}
```

每条记录之间用特殊结构分隔，题目未提供任何提示信息，目的是从中识别出唯一的真实 flag。

---

## 二、解题思路

由于题目中没有提示，考虑从以下多个角度综合分析：

### 1. 结构扫描 —— 查找异常块

通过字节层读取文件，将内容按 NULL 字节 (`0x00`) 进行分块：

```python
segments = raw.read().split(b'\x00')
```

然后统计每个分块的长度，找出唯一长度不同于正常长度的块。这种方法能高效识别出被粘连或改写的异常内容。

结果发现：

- 大多数段长度为 44 或 45 字节。
    
- 仅有一个段长为 90 字节的块，显然是由两个 UUID 被粘连构成。
    

### 2. 提取异常段内容

该异常块内容如下（解码后）：

```
flag{351e3c15-7b89-4208-af54-e29b12b9d195}\r\x00\rflag{9b97ddc7-368c-455b-9477-fae6254a6cf8}\r\n
```

即：包含两条有效格式的 UUID flag，中间被罕见的 `\r\x00\r` 分隔。



---

## 三、关键分析脚本

以下是本题用于定位异常段、提取候选 flag 并进行对比分析的完整 Python 脚本。

```python
import zipfile
from collections import Counter
import binascii

# 读取 zip 中的 flags.txt
with zipfile.ZipFile('flags.zip') as z:
    raw = z.read('flags.txt')

# NULL 分隔分块
segments = raw.split(b'\x00')
print(f'Total segments: {len(segments)}')

# 统计长度分布
length_counter = Counter(len(s) for s in segments if s)
for l, c in length_counter.most_common():
    print(f'len={l}, count={c}')

# 推测正常块长为出现频率最高者
normal_len = length_counter.most_common(1)[0][0]
odd_segments = [(i, s) for i, s in enumerate(segments) if len(s) != 0 and len(s) != normal_len]

# 提取异常段内容
print(f'Odd segment count: {len(odd_segments)}')
idx, data = odd_segments[0]
print(f'Odd segment index: {idx}, len={len(data)}')

# 尝试解码并按 flag 切分
text = data.decode(errors='replace')
candidates = [f'flag{{{frag[:36]}}}' for frag in text.split('flag{')[1:]]
for i, c in enumerate(candidates):
    print(f'Candidate #{i+1}: {c}')

# 简易数字和校验
for flag in candidates:
    digits = [int(ch) for ch in flag if ch.isdigit()]
    checksum = sum(digits) % 10
    print(f'{flag} → sum(digits) % 10 = {checksum}')
```

输出中若仅有一段长度异常且其中包含两个格式合法的 UUID flag，可据此进行进一步比对并选出真正的 flag。

---
## exp
```python
# 以二进制模式打开flags.txt文件
with open("flags.txt", "rb") as f:
    # 读取文件全部内容到raw变量
    raw = f.read()

# 使用null字节(\x00)分割文件内容
segments = raw.split(b'\x00')
# 打印分割后的总块数
print(f"总块数: {len(segments)}")

# 创建一个字典来统计不同长度的块
lengths = {}
# 遍历所有块，统计每种长度出现的次数
for i, seg in enumerate(segments):
    l = len(seg)
    # 如果该长度还未在字典中，初始化一个空列表
    if l not in lengths:
        lengths[l] = []
    # 将块的索引和内容添加到对应长度的列表中
    lengths[l].append((i, seg))

# 打印每种长度的块数量（按长度排序）
for l in sorted(lengths):
    print(f"长度 {l} 的段数: {len(lengths[l])}")

# 找出最常见的块长度（定义为"正常长度"）
normal_len = max(lengths, key=lambda x: len(lengths[x]))
# 收集所有长度不等于正常长度的块（异常块）
odd_blocks = [x for l, v in lengths.items() if l != normal_len for x in v]

# 打印异常块的数量和信息
print(f"\n异常块数量: {len(odd_blocks)}")
for i, seg in odd_blocks:
    print(f"块 #{i}, 长度={len(seg)}")
    try:
        # 尝试将二进制数据解码为字符串
        text = seg.decode()
        # 查找所有以"flag{"开头的部分
        for part in text.split("flag{")[1:]:
            # 提取UUID部分（假设UUID长度为36字符）
            uuid = part[:36]
            # 构造完整的flag格式字符串
            flag = f"flag{{{uuid}}}"
            # 提取flag中的所有数字字符并转换为整数列表
            digits = [int(c) for c in flag if c.isdigit()]
            # 计算校验和（数字之和模10）
            checksum = sum(digits) % 10
            # 打印候选flag和它的校验和
            print(f"  候选: {flag} → 校验和: {checksum}")
    except:
        # 如果解码失败，打印无法解码信息
        print("  无法解码")

```

---

## 五、最终确认

提交 `flag{351e3c15-7b89-4208-af54-e29b12b9d195}`，服务器返回正确，确认为真 flag。

---

