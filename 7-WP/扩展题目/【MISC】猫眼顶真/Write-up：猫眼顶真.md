
#wp #misc #arnold #爆破 

【[[MISC]]】扩展题目 - [[扩展题目]]

创建时间：2025-07-14 15:15
题目附件：https://wwfj.lanzoul.com/iVNiD310x3ab  
密码:433s
### 解题过程
# 题目信息

- **类别**：Misc（杂项）
    
- **文件**：`flag.png`（大小 422×420 像素，一张看似被“条纹”打乱的图片）
![](https://cdn.jsdelivr.net/gh/wydyxhxs/images/pic/%E7%8C%AB%E7%9C%BC%E9%A1%B6%E7%9C%9F.png)   

---

# 一、背景知识

本题核心考点是 **Arnold 变换**（也叫“猫映射”）。

- 它会把方阵中每个像素位置 打乱到新的位置 。
    
- 变换是可逆的，且在一定次数后会周期性回到原始图像。
    
- 变换参数有三个：
    
    - ：控制映射的“斜度”
        
    - ：变换次数
        

题目显然对原始带 flag 的正方形图片做了 Arnold 混淆，于是只要写逆变换、暴力扫参数，就能把像素还原回去。

---

# 二、工具与准备

- **Python ≥3.6**
    
- 安装以下库（点击复制运行）：
    
    ```
    pip install numpy pillow opencv-python
    ```
    
- 把题目给的 `flag.png` 放到当前目录。
    

---

# 三、详细步骤

## 1. 裁剪成正方形

题目图是 **422×420**，而 Arnold 变换只能对 **方阵** 做混淆。  
我们把左右各裁掉 1 像素，得到 **420×420**：

```python
from PIL import Image

img = Image.open('flag.png')
# 左、右各去掉 1 列 (crop 左上 x,y 到 右下 x,y)
square = img.crop((1, 0, 1+420, 420))
square.save('flag_square.png')
print("已生成 420×420 的 flag_square.png")
```

执行后会多出一个 `flag_square.png`，后面都基于这张正方图来解。

---

## 2. 编写逆 Arnold 变换函数

博客里给出的正向 Arnold 变换是：

要还原，我们需要逆矩阵。推导后可得逆变换：

下面把它写成函数，每跑一次就反向“搬”回去一次。

```python
import numpy as np

def inverse_arnold(img_np, a, b):
    """对单次 Arnold 变换做一次逆操作，img_np 是 H×W×3 的 numpy 数组。"""
    N, _, _ = img_np.shape
    out = np.zeros_like(img_np)
    for x_prime in range(N):
        for y_prime in range(N):
            # 逆公式
            x = ((a*b+1)*x_prime - b*y_prime) % N
            y = (-a*x_prime + y_prime) % N
            out[x, y, :] = img_np[x_prime, y_prime, :]
    return out
```

---

## 3. 暴力搜索参数

我们对未知的 进行三重循环暴力穷举：

- （逆变换次数）从 1 到 8（如无结果再扩到 12、20……）
    
- 从 1 到 20（若必要可扩到 50+）
    

一旦某组参数能把条纹还原出可读的 `flag{…}`，就成功了。

```python
import cv2, os

# 读取裁剪后的正方图
img0 = cv2.imread('flag_square.png')  # BGR 顺序
H, W, _ = img0.shape

out_dir = 'decrypted'
os.makedirs(out_dir, exist_ok=True)

for t in range(1, 9):        # 先扫 1–8
    for a in range(1, 21):   # 先扫 1–20
        for b in range(1, 21):
            img = img0.copy()
            # 连续做 t 次逆变换
            for _ in range(t):
                img = inverse_arnold(img, a, b)
            fn = f't{t:02d}_a{a:02d}_b{b:02d}.png'
            cv2.imwrite(os.path.join(out_dir, fn), img)
    print(f'已完成 t={t}')
```

- 程序结束后，你会在 `decrypted/` 目录下看到大量 `t##_a##_b##.png` 文件。
    
- 用图片预览工具（如 Windows 下的“照片”、macOS 的“预览”等）逐张翻看，留意哪张出现了清晰的文字。
    

---

## 4. 观察与提取

- 找到那张出现 `flag{…}` 的图片，比如 `t03_a01_b01.png` 中就能看出：
    
    ```
    flag{YOUR_ACTUAL_FLAG}
    ```
    
- 把其中的 `YOUR_ACTUAL_FLAG` 抄下来，这就是本题的答案。
    

---

# 四、运行示例截图

> 下面是一张示例（仅示意，参数和实际结果可能不同）：

```
decrypted/t03_a01_b01.png：
┌───────────────────────┐
│  ███   ██  █ █ █      │
│  ██ █ ██   █  █  █     │
│  …                      │
│  flag{arnold_magic_42} │  ← 这行文字一目了然
└───────────────────────┘
```

---

# 五、完整脚本附录

你可以把下面代码保存为 `solve.py`，一键跑完——记得先裁剪生成 `flag_square.png`：

```python
#!/usr/bin/env python3
import os
import cv2
import numpy as np
from PIL import Image

# —— 1. 裁剪成 420×420 —— 
img = Image.open('flag.png')
square = img.crop((1, 0, 421, 420))
square.save('flag_square.png')

# —— 2. 定义逆 Arnold 变换 —— 
def inverse_arnold(img_np, a, b):
    N, _, _ = img_np.shape
    out = np.zeros_like(img_np)
    for x_p in range(N):
        for y_p in range(N):
            x = ((a*b+1)*x_p - b*y_p) % N
            y = (-a*x_p + y_p) % N
            out[x, y, :] = img_np[x_p, y_p, :]
    return out

# —— 3. 暴力破解 —— 
img0 = cv2.imread('flag_square.png')
out_dir = 'decrypted'
os.makedirs(out_dir, exist_ok=True)

for t in range(1, 9):
    for a in range(1, 21):
        for b in range(1, 21):
            img = img0.copy()
            for _ in range(t):
                img = inverse_arnold(img, a, b)
            fn = f't{t:02d}_a{a:02d}_b{b:02d}.png'
            cv2.imwrite(os.path.join(out_dir, fn), img)
    print(f'[+] t={t} done')

print("[*] 所有图片已输出到 decrypted/ ，请手动查看并找到带 flag 的那张。")
```

---

# 六、小结

1. **核心思路**：逆向 Arnold 变换 + 暴力枚举参数
    
2. **关键点**
    
    - 必须先把图裁为方阵
        
    - 逆变换公式要写对
        
    - 先用小范围快速扫（比如 ），不行再扩大范围
        
3. **输出**：在 `decrypted/` 找到含 `flag{…}` 的图
    ![](https://cdn.jsdelivr.net/gh/wydyxhxs/images/pic/20250714151631520.png)
    
4. **最终 flag**：
    
    ```
    flag{Arnold_1s_s8_e2sy}
    ```
    
