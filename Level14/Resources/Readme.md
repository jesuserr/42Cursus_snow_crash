# Level14

## How to login

```bash
ssh level14@192.168.1.184 -p 4242
password: 2A31L79asukciNyi8uppkEuSx
```

## How to get the flag

Once logged, we find nothing in the home folder. So we decide to search directly for our friend 'getflag' and try to execute it, obtaining the expected result.
```bash
level14@SnowCrash:~$ ls
level14@SnowCrash:~$ whereis getflag
getflag: /bin/getflag
level14@SnowCrash:~$ /bin/getflag
Check flag.Here is your token : 
Nope there is no token here for you sorry. Try again :)
```

In order to obtain more info, we decide to run command 'ltrace' with the binary file to intercept and record dynamic library calls which are called by the executed process.
```bash
level14@SnowCrash:~$ ltrace /bin/getflag
__libc_start_main(0x8048946, 1, 0xbffff7e4, 0x8048ed0, 0x8048f40 <unfinished ...>
ptrace(0, 0, 1, 0, 0)                          = -1
puts("You should not reverse this"You should not reverse this
)            = 28
+++ exited (status 1) +++
```

It seems that 'getflag' runs 'ptrace' and for some reason it returns '-1' and makes the program to exit. After some research, we find that 'ptrace' system call is often used by programs to prevent debugging or reverse engineering. When 'ptrace' returns -1, it typically indicates that the process is already being traced (e.g., by ltrace or gdb), which is why the program is exiting prematurely. We proceed to disassemble the binary with the help of 'objdump' and search for 'ptrace' occurrences.
```bash
level14@SnowCrash:~$ objdump -d /bin/getflag | grep ptrace -A 3
08048540 <ptrace@plt>:
 8048540:       ff 25 2c b0 04 08       jmp    *0x804b02c
 8048546:       68 58 00 00 00          push   $0x58
 804854b:       e9 30 ff ff ff          jmp    8048480 <_init+0x3c>
--
 8048989:       e8 b2 fb ff ff          call   8048540 <ptrace@plt>   # Call to ptrace   
 804898e:       85 c0                   test   %eax,%eax              # has ptrace returned -1 ?
 8048990:       79 16                   jns    80489a8 <main+0x62>    # if so, exit
 8048992:       c7 04 24 a8 8f 04 08    movl   $0x8048fa8,(%esp)
 ```

The second occurrence shows the call made to 'ptrace' and how the program verifies if the value returned is negative or not. Therefore by need to bypass this protection, and for that we will use 'gdb'. So, we start running 'getflag' on 'gdb' and we obtain a message that confirms our previous investigation, the program detects that is being debugged and exits.
```bash
level14@SnowCrash:~$ gdb /bin/getflag
GNU gdb (Ubuntu/Linaro 7.4-2012.04-0ubuntu2.1) 7.4-2012.04
...
(gdb) run
Starting program: /bin/getflag 
You should not reverse this
[Inferior 1 (process 25746) exited with code 01]
(gdb) 
 ```

So, it's time to set a breakpoint where 'test %eax,%eax' is and then modify the value of the register 'eax' to 0 to bypass the protection.
```bash
(gdb) disas main
Dump of assembler code for function main:
   0x08048946 <+0>:     push   %ebp
   0x08048947 <+1>:     mov    %esp,%ebp
   ...
   0x08048989 <+67>:    call   0x8048540 <ptrace@plt>
   0x0804898e <+72>:    test   %eax,%eax                # breakpoint
   0x08048990 <+74>:    jns    0x80489a8 <main+98>      # jump if not negative
(gdb) break *main+72
Breakpoint 1 at 0x804898e
(gdb) run
Starting program: /bin/getflag 

Breakpoint 1, 0x0804898e in main ()
(gdb) print $eax
$1 = -1                                   # ptrace detected gdb
(gdb) set $eax=0                          # set value to 0 to trick 'jns'
(gdb) continue
Continuing.
Check flag.Here is your token : 
Nope there is no token here for you sorry. Try again :)
[Inferior 1 (process 25900) exited normally]
```

We were success in bypassing the 'ptrace' protection but it seems that there is another check, since we are not given the token yet. After some thought, we conclude that we are logged as 'level14' and the token will be given only if we are 'flag14', so must be some place in the binary where a user check is made. Before digging in the binary we collect some info about the user's ids.
```bash
getent passwd | grep 14
level14:x:2014:2014::/home/user/level14:/bin/bash
flag14:x:3014:3014::/home/flag/flag14:/bin/bash
```

We go back to gdb, disassemble the binary and look for the call to 'getuid'. We need to set a breakpoint where the UID is checked (cmp $0xbbe,%eax) and then replace 0xbbe (3016) by 0xbc6 (3014) which is the UID for 'flag14' as we found above.
```bash
(gdb) disas main
Dump of assembler code for function main:
   0x08048946 <+0>:     push   %ebp
   0x08048947 <+1>:     mov    %esp,%ebp
   ...
   0x08048afd <+439>:   call   0x80484b0 <getuid@plt>   # call to getuid, returns UID in eax register (2014 - level14)
   0x08048b02 <+444>:   mov    %eax,0x18(%esp)          # pushes the value of eax to the stack pointer
   0x08048b06 <+448>:   mov    0x18(%esp),%eax          # assigns to eax the content of the stack pointer
   0x08048b0a <+452>:   cmp    $0xbbe,%eax              # check if UID is 0xbbe (3006) - breakpoint
(gdb) break *main+452
Breakpoint 2 at 0x8048b0a
(gdb) continue
Continuing.

Breakpoint 2, 0x08048b0a in main ()
(gdb) print $eax
$1 = 2014
(gdb) set $eax=0xbc6              # fake UID to 3014
(gdb) continue
Continuing.
Check flag.Here is your token : 7QiHafiNa3HVozsaXkawuYrTstxbpABHD8CPnHJ
[Inferior 1 (process 26057) exited normally]
```

The token is obtained and also the flag.
```bash
level14@SnowCrash:~$ su flag14
Password: 
Congratulation. Type getflag to get the key and send it to me the owner of this livecd :)
flag14@SnowCrash:~$ getflag
Check flag.Here is your token : 7QiHafiNa3HVozsaXkawuYrTstxbpABHD8CPnHJ
flag14@SnowCrash:~$
```

Summarized gdb commands (for reference)
```bash
level14@SnowCrash:~$ gdb /bin/getflag
(gdb) break *main+72
Breakpoint 1 at 0x804898e
(gdb) break *main+452
Breakpoint 2 at 0x8048b0a
(gdb) run
Starting program: /bin/getflag 

Breakpoint 1, 0x0804898e in main ()
(gdb) set $eax=0 
(gdb) continue
Continuing.

Breakpoint 2, 0x08048b0a in main ()
(gdb) set $eax=0xbc6
(gdb) continue
Continuing.
Check flag.Here is your token : 7QiHafiNa3HVozsaXkawuYrTstxbpABHD8CPnHJ
[Inferior 1 (process 26057) exited normally]
```

## Easter Egg

With this exploit we can obtain any flag, only it's needed to set 'eax' with the UID value corresponding to 'flagXX'. For instance for 'flag14' the UID value has been 3014 as seen previously. Then, we can obtain any flag just modifying the last two numbers of the UID value. For instance, to obtain 'flag13' we just need to set eax = 0xbc5 (3013) and we obtain the flag (which, as can be seen, matches with the password at top).
```bash
(gdb) set $eax = 0xbc5    # fake UID to 3013
(gdb) continue
Continuing.
Check flag.Here is your token : 2A31L79asukciNyi8uppkEuSx
[Inferior 1 (process 26332) exited normally]
(gdb) 
```
