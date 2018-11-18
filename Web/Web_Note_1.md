<!-- First Attempt - SQLi -->
<!-- 初次尝试 SQLi -->
# First Attempt - SQLi | 初次尝试 SQLi

https://ctf.mukeran.cn/web/notes/1

前言
====
第一次开始研究CTF，并且也是第一次给CTF写WriteUp。我选择首先接触Web，因为我曾经有过Web的开发经验，懂一些Web的知识，有一定的基础。我写的这个严格来讲不是题目的WriteUp，而是一个知识的储存积累以及心理想法的文章。这篇是关于SQL注入的部分知识和4道入门题。

重点知识
====

MySQL相关
----
**information\_schema** - MySQL储存信息的Database  
**information\_schema.schemata** - 可以获取当前MySQL服务器的数据库列表  
**information\_schema.tables** - 所有MySQL表的信息，含有有用的schema\_name, table\_name两列  
**information\_schema.columns** - 所有MySQL列的信息，含有有用的schema\_name, table\_name, column\_name  
**select database()** - 显示当前数据库  

注入防护相关
----
有些网站进行了某些注入防护。  

1. **关键词过滤**
2. 类型限制
3. 字符转义
4. ...

本次主要是研究某些关键词过滤的网站如何注入  
事实上，如果一个网站的防护做的很到位的话，是不会被注入的，那么有些网站会做不到位的关键词过滤：  

1. 黑名单整体替换(删除) - 一次或多次：  
**如果是一次替换**，也就是遇到information\_schema将其删除的话，我们可以在关键词中再行插入关键词来绕过这个保护。比如：information\_information\_schemaschema。过滤部分如果只进行一次过滤的话，那么只有中间的那个information\_schema被过滤掉，剩下的拼在一起是我们说需要的关键词。(想各种方法即可)  
**如果是多次替换**，那么就没办法破了。  
2. 空格禁止(删除)：  
**如果是过滤空格的话**，我们有多种替代方案：用`/**/`, `+`, `%0a`代替空格 
3. ...

**小技巧**：  
MySQL语句中，我们可以使用`#`或者`-`来注释掉后面的代码，那么我们使用`'`来注入时，本来还要完成`'`的一个配对(语句后面往往还包含另一个`'`)，我们可以直接在语句后写`#`或者`-`(前提不被屏蔽或者后面没有重要语句)

注入测试相关
----
对于GET请求里面的Query，我们可以通过某些固定套路来测试是否支持注入。  
对于常见的id，id=1请求正常，id=1'报错，说明有可能可以注入  
我们尝试id=1' and '1'='1以及id=1' and '1'='2来看是否正常输出结果。如果前者能够出结果，而后者不能出结果，说明这个服务器支持注入，且能够使用and语句。如果不能正常执行，有可能网站会将mysql\_query报错的结果输出，我们可以看是不是因为关键词被屏蔽。

**对于能够返回结果的网站**，接下来的步骤就是：

1. `union select database()`(被屏蔽可以尝试`union select schema_name from information_schema.schemata`)(此步用于看当前的数据库，事实上也可以不用)
2. `union select table_name from information_schema.tables where table_schema='数据库名'`
3. `union select column_name from information_schema.columns where table_name='可能flag所在的表名'`
4. `union select 列名 from 表名 (where)` 获得flag


**对于不能返回结果的网站**：  
有些时候，有的网站不返回SQL查询结果，但是会反馈本次的SQL查询是否成功，是否是结果。根据这个性质(逻辑语句执行为false该值非法)，我们可以得到：

1. 判断table, column, row是否存在：  
   `and (select count(*) from ... > 0)`  
   使用该逻辑语句来判断是否存在某一条记录。有些时候我们需要用到
burp来进行字典爆破(未实践)。
2. 判断row值的长度：  
   `and (select length(列名) from ... > {length})`  
   `{length}`从1开始增加，直到某次报错，可知那次的`{length}`为条目长度。
3. 判断row值：  
   `and ascii(substr((select 列名 from 表名),{start},1))={ascii}`  
   `{start}`从1开始增加，表示substr的起点。substr长度为1，也就是获取当前测试`{start}`位的字符ascii码，`{ascii}`从可能的ASCII码范围枚举，当有返回值时，说明这个枚举的ASCII字符就是条目中的ASCII码。

目前我只做过很少的SQL注入题目，故目前没有其它经验

一个CTF上可能用不到的工具 - sqlmap.py
----
有些时候使用sqlmap.py的话，可以免去人工测试注入点。  
`sqlmap.py -u "{url}" --current-db` 获取当前db  
`sqlmap.py -u "{url}" -D {db_name} --tables` 获取`{db_name}`的所有表  
`sqlmap.py -u "{url}" -D {db_name} -T {table_name} --columns` 获取`{db_name}.{table_name}`所有列
`sqlmap.py -u "{url}" -D {db_name} -T {table_name} -C {column_name}` 获取改列所有值  
`sqlmap.py -u "{url}" -D {db_name} -T {table_name} --dump` 获取表的信息  
`sqlmap.py -u "{url}" --dump-all` 获取所有数据库信息

WriteUp
----

> Web\_WriteUp\_1.md  
> [CTF-WriteUp | mukeran.cn](https://ctf.mukeran.cn/web/writeups/1)