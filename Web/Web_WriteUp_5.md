<!-- 第四次 WriteUp -->
<!-- 三道题 -->
# https://ctf.mukeran.cn/web/writeups/5

三道题，三道都像密码学

[PHP_encrypt_1(ISCCCTF) - Bugku](https://ctf.bugku.com/challenges#PHP_encrypt_1(ISCCCTF) "前往Bugku")
----
很简单的代码理解(凯撒)

1. 题目给了一串 base64，以及一个 php 源文件。下载下来，发现这个源文件里面就是这个 base64 串的来源。
2. 文件的内容是：
    ```php
    function encrypt($data,$key)
    {
        $key = md5('ISCC');
        $x = 0;
        $len = strlen($data);
        $klen = strlen($key);
        for ($i=0; $i < $len; $i++) { 
            if ($x == $klen)
            {
                $x = 0;
            }
            $char .= $key[$x];
            $x+=1;
        }
        for ($i=0; $i < $len; $i++) {
            $str .= chr((ord($data[$i]) + ord($char[$i])) % 128);
        }
        return base64_encode($str);
    }
    ```
   发现就是一个简单的移位变换而已，第一个 for 循环是使用 $key 生成一个与 $data 串等长的密钥，然后第二个 for 循环用这个密钥对 data 进行变换，然后构造成密文 $str。
3. 所以我们直接写出解密算法，逆向变换即可：
    ```php
    function decrypt($data) {
        $data = base64_decode($data);
        $key = md5('ISCC');
        $x = 0;
        $len = strlen($data);
        $klen = strlen($key);
        for ($i=0; $i < $len; $i++) { 
            if ($x == $klen)
            {
                $x = 0;
            }
            $char .= $key[$x];
            $x+=1;
        }
        for ($i = 0; $i < $len; $i++) {
            $res .= chr((ord($data[$i]) - ord($char[$i]) + 128) % 128);
        }
        return $res;
    }
    ```
   最后运行 `echo decrypt('{base64 串}');` 即可得到 flag。

[login4 - Bugku](https://ctf.bugku.com/challenges#login4 "前往Bugku")
----
CBC 字节翻转攻击  
1. WriteUp 暂缺

[让我进去 - 实验吧](http://www.shiyanbar.com/ctf/1848 "前往实验吧")
----
hash 长度扩展攻击
1. WriteUp 暂缺