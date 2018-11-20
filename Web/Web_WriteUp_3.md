<!-- 第三次 WriteUp Part 1 -->
<!-- 两道水题 -->
# https://ctf.mukeran.cn/web/writeups/3

两道水题

[login1(SKCTF) - Bugku](http://ctf.bugku.com/challenges#login1(SKCTF) "前往Bugku")
----
这个题就是用到[https://ctf.mukeran.cn/web/notes/6](https://ctf.mukeran.cn/web/notes/6)的 SQL 约束攻击。  
1. 发现题目提示一个 SQL 约束攻击
2. 点开网页，发现是一个登录框，登录框也有注册按钮
3. 点开注册按钮，随便注册一个账号，尝试登录，提示不是管理员
4. 不是提示约束攻击吗，我们创建一个账户，用户名 admin 后面加几个空格，密码随便设一个，登录，得到 flag

[文件上传2(湖湘杯) - Bugku](http://ctf.bugku.com/challenges#%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A02(%E6%B9%96%E6%B9%98%E6%9D%AF) "前往Bugku")
----
这个题用到[https://ctf.mukeran.cn/web/notes/4](https://ctf.mukeran.cn/web/notes/4)的 php://。
1. 点开网站，发现有一个超链接 "uploading a picture"，点开发现里面是一个文件上传
2. 我想这可能是一个文件上传的利用吧，尝试上传，发现文件名都被强行更改了
3. 于是这道题可能并不是文件上传。看到 URL 里面的 GET 参数 `op`，可能有点文章。随便尝试 `op=test`，发现报错 `no such file`。然后就尝试 `op=index.php`，发现也报错，可能是自己加了后缀名。于是尝试 `op=index`，发现没有报错也没有显示。
4. 这时我们用 php:// 大法，构造 payload: `php://filter/read=convert.base64-encode/resource=./index.php`，发现又报错，再尝试 `php://filter/read=convert.base64-encode/resource=./index`，发现输出了 base64 编码，解码后得到 index 的源码，发现没有提示
5. 直接尝试 `php://filter/read=convert.base64-encode/resource=./flag` 得到 flag base64