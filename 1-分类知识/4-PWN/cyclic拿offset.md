### 32位
```python
pi from pwn import cyclic; open('/tmp/p','wb').write(cyclic(1000, n=4))
# 先运行这个
run < /tmp/p
# 崩溃后
cyclic -n 4 -l $eip
# ↳ 直接给出 offset
```

### 64位
```python
pi from pwn import cyclic; open('/tmp/p','wb').write(cyclic(2000, n=8))
#先运行这个
run < /tmp/p
# 崩溃后
x/gx $rsp                 # 取覆盖到栈顶的8字节片段
0x7fffffffe3a8: 0x6161616b6161616a
cyclic -n 8 -l 0x6161616b6161616a
# ↳ 得到 offset

```
