# CTF中MD5绕过与弱类型比较完整指南
---

## PHP弱类型比较基础

### 强类型vs弱类型比较

|比较方式|符号|特点|安全性|
|---|---|---|---|
|弱类型比较|`==`|会进行类型转换|⚠️ 存在绕过风险|
|强类型比较|`===`|不进行类型转换|✅ 相对安全|

### PHP类型转换规则

#### 字符串转数字规则

```php
// PHP类型转换示例
var_dump("123" == 123);        // true
var_dump("123abc" == 123);     // true  
var_dump("abc123" == 0);       // true
var_dump("0e123" == 0);        // true (科学计数法)
var_dump("0e123" == "0e456");  // true (都等于0)
```

#### 数组与字符串比较

```php
// 数组与字符串比较
var_dump(array() == "");       // true
var_dump(array() == 0);        // true
var_dump(array(0) == "0");     // true
```

### 常见弱类型绕过场景

#### 1. 科学计数法绕过

```php
// 0e开头的字符串会被当作科学计数法
"0e123" == "0e456"   // true (都等于0)
"0e123" == 0         // true
```

#### 2. 数组绕过

```php
// 数组比较绕过
array() == ""        // true
array() == 0         // true
array() == false     // true
```

#### 3. NULL绕过

```php
// NULL比较绕过
null == ""           // true
null == 0            // true
null == false        // true
```

---

## MD5弱类型绕过

### 基本原理

MD5函数可能返回`0e`开头的哈希值，在弱类型比较中被视为科学计数法（值为0）。

### 经典Payload

#### 1. 0e科学计数法Payload

```text
# MD5值以0e开头的字符串（弱类型比较时等于0）
240610708        # MD5: 0e462097431906509019562988736854
           # MD5: 0e830400451993494058024219903391
s878926199a      # MD5: 0e545993274517709034328855841020
s155964671a      # MD5: 0e342768416822451524974117254469
s214587387a      # MD5: 0e848240448830187420162954587645
s214587387a      # MD5: 0e848240448830187420162954587645
s1091221200a     # MD5: 0e940624217856561557816327384675
s1885207154a     # MD5: 0e509367213418206700842008763514
```

#### 2. 实战应用场景

```php
// 典型的vulnerable代码
if(md5($_GET['a']) == md5($_GET['b']) && $_GET['a'] != $_GET['b']){
    echo "flag{xxx}";
}

// 绕过方法
// ?a=240610708&b=QNKCDZO
// md5("240610708") = 0e462097431906509019562988736854
// md5("QNKCDZO") = 0e830400451993494058024219903391  
// 弱类型比较: 0 == 0 为 true
```

### 数组绕过MD5

```php
// 当输入为数组时，md5()函数返回NULL
if(md5($_GET['a']) == md5($_GET['b']) && $_GET['a'] != $_GET['b']){
    echo "flag{xxx}";
}

// 绕过方法
// ?a[]=1&b[]=2
// md5(array()) 返回 NULL
// NULL == NULL 为 true，但 array(1) != array(2)
```

### POST数据绕过

```http
POST /challenge.php HTTP/1.1
Content-Type: application/x-www-form-urlencoded

a[]=1&b[]=2
```

---

## MD5强类型绕过

### 基本原理

当代码使用强类型比较（`===`）时，需要找到MD5值完全相同的不同字符串。

### MD5碰撞Payload

#### 1. 经典MD5碰撞

```php
// 这两个字符串的MD5值完全相同
$str1 = "M%C9h%CC%0C`%C9%D6o%E1%05t%00r%BB%8Cl%10Dz%C2%A2%ECT%10%5C%0F%8F%EFj";
$str2 = "M%C9h%CC%0C`%C9%D6o%E1%05t%00r%BB%8Cl%10Dz%C2%A2%ECT%10%5C%0F%8F%EFj";

// 实际使用时需要URL解码
$collision1 = urldecode($str1);
$collision2 = urldecode($str2);
```

#### 2. 实用MD5碰撞对

```text
# 第一组碰撞
4dc968ff0ee35c209572d4777b721587d36fa7b21bdc56b74a3dc0783e7b9518afbfa200a8284bf36e8e4b55b35f427593d849676da0d1555d8360fb5f07fea2
4dc968ff0ee35c209572d4777b721587d36fa7b21bdc56b74a3dc0783e7b9518afbfa202a8284bf36e8e4b55b35f427593d849676da0d1d55d8360fb5f07fea2

# 第二组碰撞  
008ee33a9d58b51cfeb425b0959121c9-8dd5d92b032a65b1bb4e0df4a2b55a2b
008ee33a9d58b51cfeb425b0959121c9-3fd3e7b8c6a9b20e4a7d4b82a2b55a2b
```

### FastColl工具生成碰撞

```bash
# 使用FastColl工具生成MD5碰撞
./fastcoll -p prefix.txt -o collision1.bin collision2.bin

# 验证碰撞
md5sum collision1.bin collision2.bin
```

---

## MD5碰撞攻击

### 理论基础

MD5存在已知的碰撞漏洞，可以构造出具有相同MD5值的不同文件。

### 实际碰撞文件

#### 1. 二进制碰撞文件

```python
# Python生成MD5碰撞示例
collision1 = bytes.fromhex('4dc968ff0ee35c209572d4777b721587d36fa7b21bdc56b74a3dc0783e7b9518afbfa200a8284bf36e8e4b55b35f427593d849676da0d1555d8360fb5f07fea2')
collision2 = bytes.fromhex('4dc968ff0ee35c209572d4777b721587d36fa7b21bdc56b74a3dc0783e7b9518afbfa202a8284bf36e8e4b55b35f427593d849676da0d1d55d8360fb5f07fea2')

import hashlib
print(hashlib.md5(collision1).hexdigest())  # 输出相同MD5
print(hashlib.md5(collision2).hexdigest())  # 输出相同MD5
```

#### 2. 文件上传碰撞绕过

```php
// 文件上传MD5校验绕过
if(md5_file($_FILES['file']['tmp_name']) === $expected_md5){
    // 上传成功
}

// 使用碰撞文件绕过MD5校验
```

### 在线MD5碰撞工具

- **HashClash**：生成MD5碰撞的工具
- **FastColl**：快速MD5碰撞生成器
- **在线工具**：部分网站提供MD5碰撞生成服务

---

## SHA1绕过技巧

### SHA1弱类型绕过

#### 1. 0e科学计数法SHA1

```text
# SHA1值以0e开头的字符串
aaroZmOk        # SHA1: 0e66507019969427134894567494305185566735
aaK1STfY        # SHA1: 0e76658526655756207688271159624026011393
aaO8zKZF        # SHA1: 0e89257456677279068558073954252716165668
aa3OFF9m        # SHA1: 0e36977786278517984959260394024281014729
```

#### 2. 数组绕过SHA1

```php
// SHA1数组绕过
if(sha1($_GET['a']) == sha1($_GET['b']) && $_GET['a'] != $_GET['b']){
    echo "flag{xxx}";
}

// 绕过：?a[]=1&b[]=2
```

### SHA1碰撞攻击

```text
# 著名的SHA1碰撞样本（SHAttered攻击）
PDF1: 38762cf7f55934b34d179ae6a4c80cadccbb7f0a
PDF2: 38762cf7f55934b34d179ae6a4c80cadccbb7f0a

# 这两个不同的PDF文件具有相同的SHA1值
```

---

## 其他哈希函数绕过

### CRC32绕过

```php
// CRC32弱类型绕过
crc32("PLOE") == crc32("aAE0")  // true (数值相等)
```

### MD4绕过

```text
# MD4 0e科学计数法
8E8RKF6Q        # MD4: 0e967641681823153728876084350301
```

### SHA256/SHA512绕过

虽然SHA256和SHA512更安全，但仍然存在弱类型绕过：

```php
// 数组绕过
hash('sha256', array()) == hash('sha256', array())  // true (都返回NULL)
```

---

## JSON弱类型绕过

### JSON解析绕过

#### 1. 数字与字符串混淆

```json
{
    "password": 123456,
    "admin": true
}

// PHP解析后可能被当作字符串"123456"
```

#### 2. 布尔值绕过

```json
{
    "admin": true,
    "level": 1
}

// 在弱类型比较中：true == 1 为 true
```

### JSON数组绕过

```json
{
    "a": [],
    "b": []
}

// 空数组在PHP中可能被转换为相同值
```

---

## 实战Payload合集

### 1. MD5弱类型绕过合集

```text
# 最常用的MD5弱类型Payload
240610708        → 0e462097431906509019562988736854
QNKCDZO          → 0e830400451993494058024219903391  
s878926199a      → 0e545993274517709034328855841020
s155964671a      → 0e342768416822451524974117254469
s214587387a      → 0e848240448830187420162954587645
s1091221200a     → 0e940624217856561557816327384675
s1885207154a     → 0e509367213418206700842008763514
s1502113478a     → 0e861580163291561247404381396064
s1885207154a     → 0e509367213418206700842008763514
s1836677006a     → 0e481036490867661113260034900752
s155964671a      → 0e342768416822451524974117254469
s1184209335a     → 0e072485820392773389523109082030
s1665632922a     → 0e731198061491163073197128363787
s1502113478a     → 0e861580163291561247404381396064
s1836677006a     → 0e481036490867661113260034900752
s1885207154a     → 0e509367213418206700842008763514
```

### 2. SHA1弱类型绕过合集

```text
aaroZmOk        → 0e66507019969427134894567494305185566735
aaK1STfY        → 0e76658526655756207688271159624026011393
aaO8zKZF        → 0e89257456677279068558073954252716165668
aa3OFF9m        → 0e36977786278517984959260394024281014729
```

### 3. 数组绕过通用Payload

```text
# GET参数
?a[]=1&b[]=2
?param1[]=123&param2[]=456

# POST数据
a[]=1&b[]=2
user[]=admin&pass[]=123456

# JSON格式
{"a":[],"b":[]}
{"user":[],"pass":[]}
```

### 4. 组合绕过Payload

```php
// 多重验证绕过示例
if(md5($_POST['user']) == md5($_POST['pass']) && $_POST['user'] != $_POST['pass']){
    if(sha1($_POST['key1']) == sha1($_POST['key2']) && $_POST['key1'] != $_POST['key2']){
        echo "flag{xxx}";
    }
}

// 绕过方法：
// user[]=1&pass[]=2&key1[]=3&key2[]=4
```

---

## CTF实战案例

### 案例1：双重MD5验证绕过

```php
// 挑战代码
if(isset($_GET['a']) && isset($_GET['b'])){
    if($_GET['a'] != $_GET['b'] && md5($_GET['a']) == md5($_GET['b'])){
        if(md5(md5($_GET['a'])) == md5(md5($_GET['b']))){
            echo $flag;
        }
    }
}

// 解决方案
// ?a[]=1&b[]=2
// md5(array()) 返回 NULL
// md5(NULL) 也返回 NULL
// NULL == NULL 为 true
```

### 案例2：JSON+MD5组合绕过

```php
// 挑战代码
$data = json_decode($_POST['data'], true);
if(md5($data['password']) == "0"){
    echo $flag;
}

// 解决方案
// POST: data={"password":[]}
// md5(array()) 返回 NULL
// NULL == "0" 为 true（弱类型比较）
```

### 案例3：文件上传MD5绕过

```php
// 挑战代码
$upload_md5 = md5_file($_FILES['file']['tmp_name']);
if($upload_md5 == $target_md5){
    echo $flag;
}

// 解决方案：使用MD5碰撞文件
```

---

## 高级绕过技巧

### 1. 长度扩展攻击

```python
# 针对某些自定义哈希算法
def length_extension_attack(original_hash, original_length, append_data):
    # 实现长度扩展攻击
    pass
```

### 2. 时间盲注绕过

```php
// 利用MD5计算时间差异
if(md5($_GET['input']) == $secret_hash){
    // 可能存在时间差异
}
```

### 3. 内存占用攻击

```php
// 大数组可能导致内存耗尽
if(md5($_POST['data']) == "target"){
    // $_POST['data'] 为超大数组时可能导致DoS
}
```

---

## 防御与检测

### 安全编码实践

#### 1. 使用强类型比较

```php
// ❌ 危险的弱类型比较
if(md5($input) == $expected_hash)

// ✅ 安全的强类型比较  
if(md5($input) === $expected_hash)
```

#### 2. 输入验证

```php
// ✅ 验证输入类型
if(!is_string($_GET['input'])){
    die("Invalid input type");
}

if(md5($_GET['input']) === $expected_hash){
    echo "Valid";
}
```

#### 3. 使用更强的哈希算法

```php
// ✅ 使用更安全的哈希算法
hash('sha256', $input)
password_hash($password, PASSWORD_DEFAULT)  // 用于密码哈希
```

### 代码审计要点

#### 检查清单

- [ ] 是否使用了弱类型比较（`==`）
- [ ] 是否对输入类型进行验证
- [ ] 是否使用了安全的哈希算法
- [ ] 是否存在数组绕过可能性
- [ ] 是否有JSON解析相关的类型混淆

#### 常见vulnerable模式

```php
// 🚨 危险模式1：直接弱类型比较
if(md5($_GET['input']) == $target)

// 🚨 危险模式2：未验证输入类型
$data = json_decode($_POST['json'], true);
if(md5($data['key']) == $hash)

// 🚨 危险模式3：数组未过滤
if(md5($_POST['password']) == md5($_POST['confirm']))
```

---

## 工具与资源

### 在线工具

- **MD5解密网站**：cmd5.com, hashkiller.co.uk
- **哈希碰撞生成**：GitHub上的各种工具
- **类型转换测试**：在线PHP沙盒

### 本地工具

```bash
# HashClash - MD5碰撞工具
git clone https://github.com/cr-marcstevens/hashclash.git

# FastColl - 快速MD5碰撞
wget https://www.win.tue.nl/hashclash/fastcoll_v1.0.0.5.exe

# 自定义脚本生成Payload
python3 generate_md5_collision.py
```

### 学习资源

- **CTF Wiki**：ctf-wiki.org
- **OWASP指南**：类型混淆攻击
- **学术论文**：MD5和SHA1碰撞研究

---

## 📚 总结

通过本指南，你应该掌握：

✅ **基础理论**：

- PHP弱类型比较机制
- 各种哈希函数的特点和漏洞

✅ **实战技巧**：

- MD5/SHA1弱类型绕过方法
- 数组绕过和JSON绕过技巧
- 碰撞攻击的应用

✅ **安全意识**：

- 如何编写安全的哈希验证代码
- 常见漏洞模式的识别
- 防御措施的实施

✅ **CTF应用**：

- 快速识别哈希绕过题目
- 常用Payload的使用
- 高级绕过技巧的应用

**记住核心原则**：弱类型比较是万恶之源，数组是绕过神器，强类型比较+输入验证是王道！

---

_💡 提示：将常用的Payload保存到你的CTF工具包中，遇到哈希验证题目时可以快速尝试！_