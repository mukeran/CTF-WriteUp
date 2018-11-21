<!-- SQL 约束攻击 -->
<!-- SQL 约束攻击 -->
# SQL 约束攻击

https://ctf.mukeran.cn/web/notes/6

前言
====
SQL 约束攻击不属于 SQLi，因为其中运用的原理并没有设计注入。而 SQL 约束攻击应该是利用 MySQL 的设计缺陷(需要低于一个版本，具体哪个版本未知)。

利用方法
====
假设我们的表 `user` 现在是这样：  

|username varchar(10)|password varchart(10)|
|--------|--------|
|admin|123456|

现在我执行下面这一句 SQL：
```SQL
insert into `user` (`username`, `password`) values ('admin     ', 123456789);
```
上面的 admin 后面跟着 5 个空格。那么这个时候，`user` 表就表成了：  

|username varchar(10)|password varchart(10)|
|--------|--------|
|admin|123456|
|admin`_____`|123456789|

第二行 admin 用 `_` 代表空格。  
在某个 MySQL 版本前，我们执行下面这一句 SQL：
```SQL
select * from `user` where `username`='admin     ';
```
我们会惊奇的发现，查询结果返回了两行。两条数据都被查询到了。那么如果此时按照一般逻辑获取第一行的话，我们就变成了之前的 admin。  
为什么呢？这是因为 MySQL 神一般和 PHP 一样，在查询时补充了那么多的空格使长度相等。这个时候本来不相等的就被匹配上了。

影响
====
这个 BUG 只存在于某个版本之前。如果一个网站获取用户信息是以用户名作为基准，且用户创建的时候没有过滤空格，用户名为定长 varchar，我们就可以尝试利用这一个漏洞。

例题
====
[login1(SKCTF) - Bugku](http://ctf.bugku.com/challenges#login1(SKCTF) "前往Bugku")