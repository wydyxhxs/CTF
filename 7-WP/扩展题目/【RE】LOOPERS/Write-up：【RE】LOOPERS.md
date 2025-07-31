
#wp #re

【[[RE]]】【RE】LOOPERS - [[扩展题目]]

创建时间：2025-07-18 11:29


参考WP：[2023年第三届陕西省大学生网络安全技能大赛--本科高校组 Reverse题解-CSDN博客](https://blog.csdn.net/OrientalGlass/article/details/131025651?utm_source=chatgpt.com)
### 解题过程

exp：
```python
#!/usr/bin/env python3

def XorPlus(enc: bytearray, a: int, b: int, c: int, d: int) -> bytearray:
    """
    在 bytearray `enc` 中，对下标 a, b, c, d 所指向的字节依次进行
    enc[a] ^= (enc[b] + enc[d]) & 0xFF
    enc[b] ^= (enc[c] + enc[d]) & 0xFF
    enc[c] ^= (enc[a] + enc[b]) & 0xFF
    enc[d] ^= (enc[a] + enc[c]) & 0xFF
    这样一轮混合变换后返回修改过的 enc。
    """
    enc[a] ^= (enc[b] + enc[d]) & 0xFF
    enc[b] ^= (enc[c] + enc[d]) & 0xFF
    enc[c] ^= (enc[a] + enc[b]) & 0xFF
    enc[d] ^= (enc[a] + enc[c]) & 0xFF
    return enc

def decryption(block: bytearray) -> bytearray:
    """
    对单个 8 字节块执行以下解密步骤：
      1) 连续 0x72 (114) 次四轮 XorPlus 混合变换：
         - 先用索引 (2,3,6,7)
         - 再用 (0,1,4,5)
         - 再用 (4,5,6,7)
         - 最后用 (0,1,2,3)
      2) 变换结束后，将结果与固定 key b'114!514!' 逐字节做 XOR
    返回解密后的 8 字节数据块。
    """
    # 重复多轮混合
    for _ in range(0x72):
        block = XorPlus(block, 2, 3, 6, 7)
        block = XorPlus(block, 0, 1, 4, 5)
        block = XorPlus(block, 4, 5, 6, 7)
        block = XorPlus(block, 0, 1, 2, 3)

    # 最后与密钥异或
    key = b'114!514!'
    for i in range(len(block)):
        block[i] ^= key[i]
    return block

def main():
    # 原始密文（十六进制字符串），共 64 字节
    hex_string = "91fba5ccfef6e0905eeeb47940d25543c286b10de778fbb268ab7580414c0758"
    data = bytes.fromhex(hex_string)

    # 用于累积解密后的所有字节
    flag_bytes = bytearray()

    # 将密文按 8 字节一块分割，逐块解密
    for i in range(0, len(data), 8):
        block = bytearray(data[i:i+8])      # 取出当前 8 字节
        decrypted_block = decryption(block) # 解密
        flag_bytes.extend(decrypted_block)  # 拼接到结果中

    # 尝试以 UTF-8 解码并打印 flag
    try:
        print(flag_bytes.decode('utf-8'))
    except UnicodeDecodeError:
        # 若包含非文本字符，则直接原始打印字节
        print(flag_bytes)

if __name__ == "__main__":
    main()

```
