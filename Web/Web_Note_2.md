<!-- Code examination and more about SQL -->
<!-- 代码审计及 SQL 的补充 -->
# Code examination and more about SQL | 代码审计及 SQL 的补充

https://ctf.mukeran.cn/web/notes/2

知识点
====
**补充SQLi的注入测试**：  
有些可以注入的点，往往加入了一些关键词保护。这里有一种方便的方法来检验关键词是否被屏蔽。  
由于逻辑语句 `and`, `or` 有可能直接被屏蔽，我们考虑被屏蔽可能性较小的符号 `^` ，以及函数 `length` 。  
`^` 在众多语言中为异或的运算符，在SQL中也不例外。那么我们考虑 `id=1` 形式的注入，在这种情况下，我们可以将 1 与 `(length("")) > 0` 进行异或。那么 `length` 里面，我们可以填写关键词进行测试。比如： `id=1'^(length("select")>0)%23` 。如果 `select` 被屏蔽，那么就变成了 `length("")` 。此时函数返回值肯定不为0，那么就是 `1^0` 为 `True` ，此时网页正常显示或者有正确反馈，反之则无。  
  
**从CTF的题目描述中获取线索**：  
有些CTF题目中会有提示。比如 [flag.php - Bugku](http://ctf.bugku.com/challenges#flag.php "前往Bugku") 中，提示 `hint` ，其实第一次看到这种提示摸不着头脑，然后查了一下 WriteUp，发现就是让你 GET 传一个 `hint=1` 过去，然后源码就显示出来了。  
  
**伪造访问 IP**：  
由于有些 Web 程序的设计缺陷，在获取访问 IP 时使用了 header 中的 `X-Forwarded-For` 作为参考，我们就可以利用这一点，在 header 中发送伪造的 IP 地址来通过验证。  
  
**BurpSuite 配合 sqlmap**：  
遇到SQLi的题目都可以先拿 sqlmap 进行这次做题的时候遇到 POST 注入的。  
sqlmap 支持导入 HTTP request 文本来进行注入分析。我们就可以使用 BurpSuite 拦截包后，将 request 保存为文件，然后在 sqlmap 中使用 `-r filename` 来运行。我们还可以使用 sqlmap 的 `-p` 参数来指定一个 field 来尝试注入。  
  
**有些时候 SQL 语句中的转义是双引号**

WriteUp
====
> Web\_WriteUp\_2.md  
> [CTF-WriteUp | mukeran.cn](https://ctf.mukeran.cn/web/writeups/2)