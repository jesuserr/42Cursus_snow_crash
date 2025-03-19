# Level07

## How to Login

To access the system, use the following SSH command:

```bash
ssh level07@<IP_ADDRESS> -p 4242
password: wiok45aaoguiboiki2tuin6ub
```

## How to Get the Flag

Upon logging in, you'll see a simple prompt. The challenge involves exploiting a vulnerability related to environment variables.

### Analyzing the Executable

The `level07` executable, when run, simply prints "level07". This suggests it might be vulnerable to environment variable manipulation. We download the executable using `scp`:

```bash
scp level07@<IP_ADDRESS>:/home/user/level07 ./
```

Then use a tool like Binary Ninja (or `strings`, `gdb`, `objdump`, etc.) to examine the executable. The key finding is the use of the `LOGNAME` environment variable:

```
/bin/echo %s
```

This indicates that the program takes the value of the `LOGNAME` environment variable and passes it to `/bin/echo`. This is a classic command injection vulnerability.

### Exploiting the Vulnerability

The vulnerability can be exploited by setting the `LOGNAME` environment variable to a string that includes a command injection. We want to execute `/bin/bash` to get a shell as `flag07`.

1.  **Set the `LOGNAME` environment variable:**

    ```bash
    export LOGNAME="exploit ; /bin/bash"
    ```

    *   `export LOGNAME=...`: Sets the `LOGNAME` environment variable.
    *   `"exploit ; /bin/bash"`: This is the payload. The semicolon (`;`) allows us to execute a second command after the intended `echo` command.  `exploit` is just a placeholder; the important part is the `;/bin/bash`. The program will effectively execute `/bin/echo exploit ; /bin/bash`.

2.  **Run the `level07` executable:**

    ```bash
    ./level07
    ```

    Because of the modified `LOGNAME` variable, this will:
    *   Print "exploit".
    *   Execute `/bin/bash` *as the `flag07` user* (due to setuid).  You will now have a shell as `flag07`.

3.  **Get the flag:**

    ```bash
    getflag
    ```

    This will output:

    ```
    Check flag.Here is your token : fiumuikeil55xe9cu4dood66h
    ```

