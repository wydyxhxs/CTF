```python
# RC4 解密脚本的实现和验证
def rc4(key, data):
    # 初始化 S 算法的状态向量
    S = list(range(256))
    j = 0
    
    # 密钥调度算法 (KSA)
    for i in range(256):
        j = (j + S[i] + key[i % len(key)]) % 256
        S[i], S[j] = S[j], S[i]
    
    # 伪随机生成算法 (PRGA)
    i = j = 0
    output = []
    for byte in data:
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        K = S[(S[i] + S[j]) % 256]
        output.append(byte ^ K)
    
    return bytes(output)

# 测试数据
key = b'YourKey'  # 密钥
ciphertext = bytes([0x3a, 0x0f, 0x7d, 0x9b, 0x47, 0x23, 0x99])  # 加密后的数据

# 解密数据
plaintext = rc4(key, ciphertext)
plaintext

```