# [[GDB]]常用指令


| 指令               | 描述                                                          |
|--------------------|---------------------------------------------------------------|
| `gdb <program>`     | 启动调试指定的程序                                           |
| `start`             | 启动程序并在 `main` 函数入口处暂停                            |
| `run`               | 启动程序并运行，直到遇到第一个断点或程序结束                |
| `run <args>`        | 启动程序并传递命令行参数 `<args>`                            |
| `break <location>`  | 在指定的位置设置断点，`<location>` 可以是函数名、行号等    |
| `break <function>`  | 在函数入口处设置断点                                         |
| `break *<address>`  | 在指定内存地址设置断点                                       |
| `delete <breakpoint>` | 删除指定的断点（`<breakpoint>` 为断点编号）                  |
| `info breakpoints`  | 查看所有已设置的断点                                         |
| `watch <variable>`  | 设置观察点，监视某个变量的变化                               |
| `info watchpoints`  | 查看所有已设置的观察点                                       |
| `c` / `continue`    | 继续程序执行，直到遇到下一个断点                             |
| `step`              | 执行当前行，若该行是函数调用，则进入该函数内部调试         |
| `next`              | 执行当前行，若该行是函数调用，则跳过该函数                   |
| `finish`            | 执行当前函数，直到返回调用该函数的地方                       |
| `until`             | 继续执行直到程序位置大于当前行或满足某个条件                |
| `stepi`             | 单步执行指令                                                  |
| `nexti`             | 单步跳过指令（不进入函数）                                   |
| `print <expression>`| 打印指定表达式的值（包括变量、寄存器等）                     |
| `x/<format> <address>` | 以指定格式查看内存内容，`<format>` 可以是 `x`（十六进制）等  |
| `display <expression>` | 每次程序停下时自动显示指定表达式的值                        |
| `info locals`       | 查看当前函数的局部变量                                       |
| `info args`         | 查看当前函数的参数                                           |
| `backtrace` / `bt`  | 打印当前调用栈                                               |
| `info frame`        | 查看当前堆栈帧的详细信息                                     |
| `info registers`    | 查看所有寄存器的值                                           |
| `info source`       | 查看当前源代码文件的信息                                     |
| `list`              | 显示当前源代码                                               |
| `set $<register> = <value>` | 设置寄存器的值                                        |
| `set variable <variable> = <value>` | 修改变量的值                                     |
| `quit` / `q`        | 退出 GDB 调试会话                                             |
| `help`              | 获取 GDB 帮助                                                 |
| `help <command>`    | 获取指定命令的帮助                                           |
| `info <topic>`      | 查看某个主题的调试信息（如 `info breakpoints`）               |
| `source <file>`     | 执行 GDB 脚本文件                                             |
| `set pagination off`| 关闭分页，输出结果不分页                                     |
| `set confirm off`   | 关闭确认提示（如删除断点时）                                 |
| `up`                | 移动到上一个堆栈帧                                           |
| `down`              | 移动到下一个堆栈帧                                           |
| `set disassembly-flavor intel` | 设置反汇编语法为 Intel 风格（默认是 AT&T 风格） |
| `set logging on`    | 开启 GDB 日志记录                                             |
| `set logging off`   | 关闭 GDB 日志记录                                             |
| `set verbose on`    | 打开详细模式，显示更多调试信息                               |
| `info threads`      | 查看当前程序的所有线程                                       |
| `thread <id>`       | 切换到指定线程                                               |
| `kill`              | 结束程序的执行                                               |
