##### 1、验证winget是否存在
```
winget --version
```
![](https://cdn.jsdelivr.net/gh/wydyxhxs/images/pic/20250706135240188.png)
这里显示版本号就可以进行下一步
##### 2、搜索winget仓库
```
winget search strings
```
![](https://cdn.jsdelivr.net/gh/wydyxhxs/images/pic/20250706135213429.png)
##### 3、用winget安装strings
```
winget install --id Microsoft.Sysinternals.Strings -e
```
![](https://cdn.jsdelivr.net/gh/wydyxhxs/images/pic/20250706135429301.png)

至此，strings就安装成功了，并配置到系统变量中，可以在cmd窗口中使用了。
![](https://cdn.jsdelivr.net/gh/wydyxhxs/images/pic/20250706135608392.png)

e.g.
![](https://cdn.jsdelivr.net/gh/wydyxhxs/images/pic/20250706140831002.png)
