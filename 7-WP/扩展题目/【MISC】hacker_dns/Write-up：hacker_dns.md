
#wp #misc

【[[MISC]]】hacker_dns - [[扩展题目]]

创建时间：2025-07-05 18:15
### 题目附件
[[hacker_dns.zip]]
### 解题过程
1、压缩包解密
题目给了一个加密的压缩包，经检查不是伪加密。使用John the ripper进行爆破。
启动kali，
```
zip2john hacker_dns.zip > 1.txt     # 提取 ZIP 密码哈希
john 1.txt                          # 用默认配置进行破解
john --show 1.txt                   # 显示已经破解出来的密码
```
![](https://cdn.jsdelivr.net/gh/wydyxhxs/images/pic/20250705201001064.png)

爆破成功，使用这个密码解压后得到两个文件
[[边伯贤key.jp
[[flag.pcap]]
经尝试，jpg文件使用了[[steghide]]进行加密，在kali中使用steghide进行解密：
![](https://cdn.jsdelivr.net/gh/wydyxhxs/images/pic/20250705201056631.png)
查看1.txt的内容：
```
$ cat 1.txt
这个key希望对你有帮助：ymhsec
```
得到一串字符`ymhsec` 

2、处理流量包
观察流量包，根据题目名字直接按dns进行筛选，得如下结果：
```
dns && not icmp          #wireshark过滤条件
```
![](https://cdn.jsdelivr.net/gh/wydyxhxs/images/pic/20250705201125531.png)
可见一直在访问**.text.com这个网址，前两位在不断变化。
使用tshark导出字段进行分析：
```
tshark -r flag.pcap -Y "dns.flags.response == 0 && not icmp" -T fields -e dns.qry.name > output.txt
```
用sublime打开这个txt文件进行快速操作：
![](https://cdn.jsdelivr.net/gh/wydyxhxs/images/pic/20250705201149449.png)
使用shift+鼠标右键拖选进行列选择选择前2列，粘贴到一个新窗口中按ctrl+shift+j转为单行，再替换所有空格和16进制换行符`0a` 为空得到字符串：
```
647868797b366a6530636d316c3678383937303333696730613172683430393135763635397d
```
在kali中将这个16进制字符串转为字符：
```
$ echo 647868797b366a6530636d316c3678383937303333696730613172683430393135763635397d | xxd -r -p
dxhy{6je0cm1l6x897033ig0a1rh40915v659}
```
dxhy{6je0cm1l6x897033ig0a1rh40915v659}
再结合上一步中获得的密文`ymhsec` ，进行维吉尼亚解密得到flag
```
flag{6fc0ea1e6f897033ee0c1fa40915d659}
```
![](https://cdn.jsdelivr.net/gh/wydyxhxs/images/pic/20250705201234665.png)


