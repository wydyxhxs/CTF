题目：

table = string.ascii_lowercase

密文：xhgq{djckckgqmmlgxxcte}

格式：flag{}

已知密文和部分原文，使用以下脚本解密：
```python
import string

def modInverse(a, m):
    """
    计算 a 在模 m 下的乘法逆元
    使用扩展欧几里得算法，但对于m=26的小数，可以直接遍历查找
    """
    for x in range(1, m):
        if (a * x) % m == 1:
            return x
    return None # 如果没有逆元则返回None

def solve_affine_cipher():
    """
    解决仿射密码问题的主函数
    """
    # 字母表和密文
    alphabet = string.ascii_lowercase
    m = len(alphabet)
    full_ciphertext = "xhgq{djckckgqmmlgxxcte}"
    
    # --- 步骤 1: 使用已知明文/密文对求解密钥 (a, b) ---
    
    # 我们知道 'flag' -> 'xhgq'
    # 让我们使用 'a' -> 'g' 和 'f' -> 'x'
    # 明文 'a' (0) -> 密文 'g' (6)
    # 明文 'f' (5) -> 密文 'x' (23)

    # 从 (a * 0 + b) % 26 = 6, 我们可以直接得到 b
    p1_num = alphabet.find('a')  # 0
    c1_num = alphabet.find('g')  # 6
    # b = c1_num
    b = (c1_num - p1_num * 0) % m  # 更通用的写法
    print(f"[*] 从 'a' -> 'g' 推断出密钥 b = {b}")

    # 现在用 (a * 5 + 6) % 26 = 23 求解 a
    p2_num = alphabet.find('f')  # 5
    c2_num = alphabet.find('x')  # 23
    
    # 5a ≡ (23 - 6) mod 26
    # 5a ≡ 17 mod 26
    # a = 17 * (5的逆元) mod 26
    inv_p2 = modInverse(p2_num, m)
    if inv_p2 is None:
        print(f"[!] 错误: {p2_num} 在模 {m} 下没有逆元。")
        return
    
    a = ( (c2_num - b + m) * inv_p2 ) % m
    print(f"[*] 从 'f' -> 'x' 推断出密钥 a = {a}")

    # --- 步骤 2: 准备解密 ---
    
    # 计算 a 的模逆元 a_inv
    a_inv = modInverse(a, m)
    if a_inv is None:
        print(f"[!] 错误: 密钥 a={a} 在模 {m} 下没有逆元，无法解密。")
        return
        
    print(f"[*] 计算出 a 的逆元 a_inv = {a_inv}")
    print(f"[*] 解密公式为: D(y) = {a_inv} * (y - {b}) mod {m}")

    # --- 步骤 3: 解密整个密文 ---
    
    plaintext = ""
    for char in full_ciphertext:
        if char in alphabet:
            y = alphabet.find(char)
            # 应用解密公式
            x = (a_inv * (y - b + m)) % m
            plaintext += alphabet[x]
        else:
            # 对于非字母字符（如 '{', '}'），直接保留
            plaintext += char
            
    print("\n" + "="*30)
    print(f"密文: {full_ciphertext}")
    print(f"解密后的明文: {plaintext}")
    print("="*30)


if __name__ == "__main__":
    solve_affine_cipher()
```
