<!-- 第三次 WriteUp Part 2 -->
<!-- 这里针对一道分析 CMS 博客 Typecho 漏洞的题 -->
# https://ctf.mukeran.cn/web/writeups/4

第一次接触比较硬核的代码审计，而且我发现我几年前可能还是这个漏洞的受害者。这道题可以带来很多反思。

Note
----
[https://ctf.mukeran.cn/web/notes/5](https://ctf.mukeran.cn/web/notes/5)

[小明的博客 - Bugku](http://ctf.bugku.com/challenges#%E5%B0%8F%E6%98%8E%E7%9A%84%E5%8D%9A%E5%AE%A2 "前往Bugku")
----
Typecho 1.0 版本的 BUG，详见上面 Note。这里只说怎么得到 flag

1. 执行 `var_dump(scandir('./'))`，看到可疑文件 `*******.txt`
2. 访问 `*******.txt`，得到 flag