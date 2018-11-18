<!-- WriteUp 2 -->
<!-- 三道简单题 -->
# https://ctf.mukeran.cn/web/writeups/2

三道题都挺水的

[程序员本地网站 - Bugku](http://ctf.bugku.com/challenges#%E7%A8%8B%E5%BA%8F%E5%91%98%E6%9C%AC%E5%9C%B0%E7%BD%91%E7%AB%99 "前往Bugku")
----
这道题真的挺水的。  
1. 打开网址，发现提示“请从本地访问”，这时就想可能是用 `X-Forwarded-For` 伪造本地请求。
2. 使用 BurpSuite 抓包，在 header 中加入 `X-Forwarded-For: 127.0.0.1`，Go，得到 flag

[这是一个神奇的登录框 - Bugku](http://ctf.bugku.com/challenges#%E8%BF%99%E6%98%AF%E4%B8%80%E4%B8%AA%E7%A5%9E%E5%A5%87%E7%9A%84%E7%99%BB%E9%99%86%E6%A1%86 "前往Bugku")
----
这道题是运用 sqlmap
1. 打开网址，发现有一个 Form 。二话不说，直接查看源码，分析 Form 结构。
2. 发现 Form 没啥特殊的，于是随便填一个内容，然后 BurpSuite 抓包。
3. 尝试 `'`，发现没有没有报错。尝试 `"`，发现报错。这个题竟然偷偷用双引号！说明可以注入。
4. 二话不说，保存 header，终端使用 sqlmap。`sqlmap.py -r "header" -p "admin_name" --dbs`，成功！
5. 一顿熟悉的 sqlmap 操作，flag GET，注意数据库中 flag 不是最终答案，最后要用 flag{} 包起来。

[flag.php - Bugku](http://ctf.bugku.com/challenges#flag.php "前往Bugku")
----
注意看懂提示  
1. 打开网址，发现一个 Form，查看源代码，发现 Form 的目标是 `#`，有可能是一个注入的题，于是用 BurpSuite 测试这个 POST。发现无论输入什么内容都没有反馈。
2. 看到题目中的 hint。脑洞大开(怎么可能)，在 URL 后面加上 query 参数 `hint=1`，然后发现页面输出了源代码。结果这个题是代码审计。
3. 发现上面的代码判断 Cookie 中的 ISecer 在 PHP 的 `unserialize` 之后是否与  `$KEY` 完全相等。然后发现代码尾部有一个 `$KEY` 的赋值。这里就要注意代码的执行顺序，事实上在判断时，`$KEY`并没有被赋值，那么应该是初值 `null`，而代码是 `unserialize($cookie) === "$KEY"`，也就是比较的右侧是一个空串，那么我们用 BurpSuite，把 Cookie 中的 ISecer 设置成空串的 `serialize` : `s:0:""`，得到 flag。