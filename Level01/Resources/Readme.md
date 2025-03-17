# Level01

## How to Login

To access the system, use the following SSH command:

```bash
ssh level01@192.168.56.104 -p 4242
password: level01
```

## How to Get the Flag

Upon logging in, start by searching for files containing the string "flag01" to find potential leads:

```bash
find / -type f -exec grep -l "flag01" {} + 2>/dev/null
```

This command yields the following files:
- `/etc/group`
- `/etc/passwd`
- `/proc/2641/task/2641/cmdline`
- `/proc/2641/cmdline`
- `/rofs/etc/group`
- `/rofs/etc/passwd`

Upon inspecting these files, focus on `/etc/passwd`, where you'll find the entry:

```
flag01:42hDRfypTqqnw:3001:3001::/home/flag/flag01:/bin/bash
```

The password `42hDRfypTqqnw` does not work directly. It is encrypted and needs to be cracked.

### Cracking the Password

1. **Install John the Ripper**: Ensure you have John the Ripper installed on your system.
2. **Download a Dictionary**: Obtain a dictionary file, such as `rockyou.txt`.
3. **Prepare the Password File**: Save the encrypted password `42hDRfypTqqnw` into a file named `password.txt`.
4. **Crack the Password**: Use John the Ripper with the dictionary:

   ```bash
   john --wordlist=rockyou.txt password.txt
   ```

   The cracking process should return:

   ```
   password.txt
   Loaded 1 password hash (descrypt, traditional crypt(3) [DES 128/128 SSE2-16])
   Will run 4 OpenMP threads
   Press 'q' or Ctrl-C to abort, almost any other key for status
   abcdefg          (?)
   1g 0:00:00:00 100% 50.00g/s 819200p/s 819200c/s 819200C/s 123456..bibiana
   Use the "--show" option to display all of the cracked passwords reliably
   Session completed
   ```

   The cracked password is `abcdefg`.

### Accessing the Flag

With the password `abcdefg`, switch to the `flag01` user:

```bash
su flag01
```

Enter the password `abcdefg` when prompted. Once logged in as `flag01`, execute the `getflag` command to retrieve the flag:

```bash
getflag
```

The command will return the flag:

```
f2av5il02puano7naaf6adaaf
```

You have successfully completed Level01.