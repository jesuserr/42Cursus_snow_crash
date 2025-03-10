# Level00

## How to login

```
ssh level00@192.168.56.104 -p 4242
password: level00
```

## How to get the flag

Once logged, there is nothing in the home folder to give us a clue. But then, we remember that in the intra video (https://elearning.intra.42.fr/notions/snow-crash/subnotions/snow-crash/videos/snow-crash) it is said "FIND this first file who can run only as flag00...", so we proceed to use command 'find' to search for files belonging to user flag00:
```
find / -user flag00
find / -user flag00 2>/dev/null  (even better)
```

We obtain a long list of files with denied access, except for these two files:
```
/usr/sbin/john
/rofs/usr/sbin/john
```

Checking the content of any of these files we see that contains the following string:
```
level00@SnowCrash:~$ cat /usr/sbin/john
cdiiddwpgswtgt
```

We try 'su flag00' with this text but it fails. It looks like a ciphered text using Caesar rotation. Using the external web (https://www.dcode.fr/caesar-cipher) the content is deciphered and found to be:
```
nottoohardhere
```

We proceed with 'su flag00' and introduce the previous text as the password, which give us access to the flag as soon as we write the 'getflag' command.
```
level00@SnowCrash:~$ su flag00
Password:
Don't forget to launch getflag !
flag00@SnowCrash:~$ getflag
Check flag.Here is your token : x24ti5gi3x0ol2eh4esiuxias
```
