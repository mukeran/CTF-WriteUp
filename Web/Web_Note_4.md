<!-- php:// Summary -->
<!-- php:// 总结 -->
# php:// Summary | php:// 总结

https://ctf.mukeran.cn/web/notes/4

前言
====
Web 里面经常遇到一些文件读取函数或表达式，如 `file_get_contents`, `readfile`, `include`。这些函数或表达式要求的 `$filename` 是可以为一个 url，那么这个时候我们就可以用到 PHP 定义的 `php://` 协议。

`php://` 在 Web 中常用方法
====
> `php://input`  
> `php://filter`
****
**php://input**:  
`file_get_contents('php://input')` 事实上就是读取 HTTP 请求里的 raw 内容。
****
**php://filter**:  
这是一个强大的工具。其格式为：  
`php://filter/read=<expression>/resource=<file>`(仅在这里写 CTF 中常见的用法)  
其中 `<file>` 为读取的文件。`<expression>` 是若干个过滤器，其语法类似于 bash 的管道，`filterA|filterB`。有这些常用的过滤器：  
1. `string.rot13` -> `str_rot13()` 对字符串进行 rot13 编码，即用一个字母后面第 13 个字母替换它
2. `string.toupper` -> `strtoupper()` 转换为大写
3. `string.tolower` -> `strtolower()` 转换为小写
4. `string.strip_tags` -> `strip_tags()` 从字符串中去除 HTML 和 PHP 标记
5. `convert.base64-encode` -> `base64_encode()` 将内容转化为 base64 编码，常用于获取源码或者 flag

读取源代码我们可以这样写：  
`php://read=convert.base64-encode/resource=./index.php`

例题
====
|题目|类型|
|---|---|
|[welcome to bugkuctf - Bugku](http://ctf.bugku.com/challenges "前往Bugku")|php:// 运用及序列化|
