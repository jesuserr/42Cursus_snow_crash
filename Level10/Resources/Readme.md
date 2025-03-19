# Level10

## How to login

```bash
ssh level10@192.168.1.184 -p 4242
password: s5cAJpM8ev6XHw998pRWG728z
```

## How to get the flag

Once logged, we have access to two files called 'level10' and 'token' (both belonging to user 'flag10'). We try to see the content of 'token' but we are not allowed since the file belongs to user 'flag10'. On the other hand, we have the binary file 'level10' that needs two arguments (being the first one the token) that presumably will give us access to the password.
```bash
level10@SnowCrash:~$ ls -l
total 16
-rwsr-sr-x+ 1 flag10 level10 10817 Mar  5  2016 level10
-rw-------  1 flag10 flag10     26 Mar  5  2016 token
level10@SnowCrash:~$ cat token 
cat: token: Permission denied
level10@SnowCrash:~$ ./level10
./level10 file host
        sends file to host if you have access to it
level10@SnowCrash:~$ ./level10 token localhost
You don't have access to token
```

It is noticed that the binary has the Setuid bit set, meaning that it will be executed with the privileges of the file owner, this can be a security risk if the file is vulnerable to exploitation since an attacker could potentially exploit the file to gain the privileges of the file owner (flag10). Thanks to this we can execute 'level10'.

In order to obtain more info, we decide to run command 'ltrace' with the binary file to intercept and record dynamic library calls which are called by the executed process. 
```bash
level10@SnowCrash:~$ ltrace ./level10 token localhost
__libc_start_main(0x80486d4, 3, 0xbffff7c4, 0x8048970, 0x80489e0 <unfinished ...>
access("token", 4)                             = -1
printf("You don't have access to %s\n", "token"You don't have access to token
) = 31
+++ exited (status 31) +++
level10@SnowCrash:~$ 
```

It seems that 'access' is called in order to verify if the user can access to 'token'. Taking a look to 'access' man pages we find the following behavior that can be used for our exploiting purposes.
```
NOTES
       Warning:  Using  access()  to  check if a user is authorized to, for
       example, open a file before actually doing so using open(2)  creates
       a  security  hole,  because  the  user  might exploit the short time
       interval between checking and opening the  file  to  manipulate  it.
       For this reason, the use of this system call should be avoided.  (In
       the example just described, a safer alternative would be  to  tempo‐
       rarily  switch  the  process's  effective user ID to the real ID and
       then call open(2).)
```

This issue creates what is called a Time-of-Check to Time-of-Use (TOCTOU) race condition. So, if we are able to swap 'token' with our exploit file in the lapsus of time between the successful call to 'access' and the call to 'open' we could have a chance to inject our exploit. This can be done replacing the 'token' file with a symbolic link to our exploit, tricking 'level10' into opening the wrong file.

