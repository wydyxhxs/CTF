#爆破
#压缩包 
[[john the ripper]]   -kali中的工具
```
zip2john hacker_dns.zip > 1.txt     # 提取 ZIP 密码哈希
john 1.txt                          # 用默认配置（自动启用 incremental + rules）进行破解
john --show 1.txt                   # 显示已经破解出来的密码
```
如果是rar文件，则使用`rar2john` 
