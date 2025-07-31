一道简单的[[rust逆向]]
#rust逆向 #位运算 
## 题目信息
- 文件名：ezrust.exe
https://wwfj.lanzoul.com/iczMR30k5j4h  
密码:bueq

## 题目分析

### 1. 初步分析

这是一道Rust编写的逆向题目。通过IDA分析可以看到，这是一个典型的flag验证程序。

### 2. 程序流程分析

程序的主要逻辑在`sub_140002220`函数中，让我们逐步分析：

1. **用户输入**：程序首先输出提示信息"please input your flag:"，要求用户输入flag
2. **长度检查**：程序检查输入的长度是否为23字符
3. **字符处理**：对输入的每个字符进行特殊的位运算处理
4. **比较验证**：将处理后的结果与硬编码的目标数组进行比较

### 3. 核心算法分析

通过分析反编译代码，我们可以看到程序对输入进行了如下处理：

```c
// 伪代码逻辑
for each character in input:
    v13 = character >> 4         // 右移4位
    v9 = 16 * ((character & 15) ^ 8)  // 低4位异或8后乘以16
    result = v9 + (v13 ^ 9)      // 组合结果
```

具体分析：
- `sub_140002AE0(v29, 4)` 实现右移4位操作，提取高4位
- `sub_140002B30(v12, 15)` 实现与15的按位与操作，提取低4位
- 然后进行异或和移位运算的组合

### 4. 目标数组分析

程序中硬编码了一个23字节的目标数组：
```
[-82, -36, -66, -50, 124, -102, -66, 124, -115, 124, -66, -115, -17, -70, 124, -102, -97, 111, -1, -34, -97, -1, -33]
```

转换为无符号字节：
```
[174, 220, 190, 206, 124, 154, 190, 124, 141, 124, 190, 141, 239, 186, 124, 154, 159, 111, 255, 222, 159, 255, 223]
```

### 5. 逆向算法推导

从算法分析可以看出，对于输入字符c：
1. 取高4位：`high = c >> 4`
2. 取低4位：`low = c & 15`
3. 计算：`result = 16 * (low ^ 8) + (high ^ 9)`

要逆向这个过程，我们需要：
1. 对于目标值target，求解原始字符c
2. 设 `x = target // 16`，`y = target % 16`
3. 则 `low ^ 8 = x`，`high ^ 9 = y`
4. 所以 `low = x ^ 8`，`high = y ^ 9`
5. 原始字符 `c = high * 16 + low`

## 解题脚本

```python
def solve_ezrust():
    # 目标数组（程序中硬编码的期望结果）
    target = [174, 220, 190, 206, 124, 154, 190, 124, 141, 124, 190, 141, 239, 186, 124, 154, 159, 111, 255, 222, 159, 255, 223]
    
    flag = ""
    
    for target_byte in target:
        # 暴力搜索：对每个目标字节，尝试所有可能的ASCII字符
        for char_val in range(32, 127):  # 可打印ASCII范围
            # 模拟程序的加密算法
            high_4_bits = char_val >> 4      # 提取高4位
            low_4_bits = char_val & 0x0F     # 提取低4位
            
            # 核心算法：result = 16 * (low ^ 8) + (high ^ 9)
            encrypted = 16 * (low_4_bits ^ 8) + (high_4_bits ^ 9)
            
            if encrypted == target_byte:
                flag += chr(char_val)
                break
    
    return flag

# 验证函数
def verify_algorithm(char):
    """验证加密算法"""
    char_val = ord(char)
    high = char_val >> 4
    low = char_val & 15
    result = 16 * (low ^ 8) + (high ^ 9)
    return result

if __name__ == "__main__":
    flag = solve_ezrust()
    print(f"Flag: {flag}")
    
    # 验证结果
    target = [174, 220, 190, 206, 124, 154, 190, 124, 141, 124, 190, 141, 239, 186, 124, 154, 159, 111, 255, 222, 159, 255, 223]
    
    print("\n验证结果:")
    for i, char in enumerate(flag):
        calculated = verify_algorithm(char)
        expected = target[i]
        status = "✓" if calculated == expected else "✗"
        print(f"字符 '{char}' (ASCII {ord(char)}) -> {calculated} (期望 {expected}) {status}")
    
    # 检查是否所有字符都匹配
    all_match = all(verify_algorithm(char) == target[i] for i, char in enumerate(flag))
    print(f"\n整体验证结果: {'成功' if all_match else '失败'}")
```

## 运行结果

运行解题脚本后得到flag：

```
Flag: rUst_1s_@_s@f3_1anguage
```

