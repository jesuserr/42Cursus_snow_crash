# Level09

## How to Login

To access the system, use the following SSH command:

```bash
ssh level09@<IP_ADDRESS> -p 4242
password: 25749xKZ8L7DkSCwJkT9dyv6f
```

## How to Get the Flag

Upon logging in, you'll find several files, including a setuid executable named `level09` and a file named `token`.

### Analyzing the Files

Listing the directory contents with `ls -la` reveals:

```
r-x------ 1 level09 level09  140 Mar  5  2016 .
d--x--x--x 1 root    users    340 Aug 30  2015 ..
-r-x------ 1 level09 level09  220 Apr  3  2012 .bash_logout
-r-x------ 1 level09 level09 3518 Aug 30  2015 .bashrc
-rwsr-sr-x 1 flag09  level09 7640 Mar  5  2016 level09
-r-x------ 1 level09 level09  675 Apr  3  2012 .profile
----r--r-- 1 flag09  level09   26 Mar  5  2016 token
```

*   `level09`:  This is the executable we need to analyze. It's setuid to `flag09`, meaning it runs with `flag09`'s privileges.
*   `token`: This file, owned by `flag09`, likely contains a transformed version of the password.

### Analyzing the `level09` Executable

Running `level09` without arguments shows:

```
You need to provied only one arg.
```

Running it with an argument (e.g., `token`) produces a short, seemingly random output (`tpmhr` in your example).  `cat token` shows non-ASCII characters, and `xxd token` shows the hexadecimal representation:

```
0000000: 6634 6b6d 6d36 707c 3d82 7f70 826e 8382  f4kmm6p|=..p.n..
0000010: 4442 8344 757b 7f8c 890a                 DB.Du{....
```

The key insight is that `level09` performs a character-by-character transformation on its input.  Testing with a simple input like `./level09 0000000000` gives `0123456789`. This strongly suggests that the transformation is an addition based on the character's position (index) in the input string.  Specifically, `output[i] = input[i] + i`.

### Reversing the Transformation

Since the `token` file contains the *output* of this transformation, we need to *reverse* it to find the original password.  The reverse operation is subtraction: `input[i] = output[i] - i`.

1.  **Hexadecimal Representation:**  We have the hexadecimal representation of the `token` file's contents from `xxd`:

    ```
    0x66 0x34 0x6b 0x6d 0x6d 0x36 0x70 0x7c 0x3d 0x82 0x7f 0x70 0x82 0x6e 0x83 0x82 0x44 0x42 0x83 0x44 0x75 0x7b 0x7f 0x8c 0x89 0x0a
    ```

2.  **C Program for Reversal:** You correctly wrote a C program to perform the subtraction:

    ```c
    #include <stdio.h>
    #include <string.h>

    int main(int ac, char **av)
    {
        unsigned char temp[26] = {0x66, 0x34, 0x6b, 0x6d, 0x6d, 0x36, 0x70, 0x7c, 0x3d, 0x82, 0x7f, 0x70, 0x82, 0x6e, 0x83, 0x82, 0x44, 0x42, 0x83, 0x44, 0x75, 0x7b, 0x7f, 0x8c, 0x89, 0x0a};
        int i;
       
        i = -1;
        while (++i < 26) 
            temp[i] = temp[i] - i;
        printf("after decryption : %s\n", temp);
        return 0; 
    }
    ```
3.  **Compile and Run:** Compile the C code (e.g., `gcc -o decrypt decrypt.c`) and run it (`./decrypt`). This produces the output `f3iji1ju5yuevaus41q1afiuq`, which is the password for `flag09`.

4.  **Switch User and Get Flag:**

    ```bash
    su flag09
    password: f3iji1ju5yuevaus41q1afiuq
    getflag
    ```

    This will output:

    ```
    Check flag.Here is your token : s5cAJpM8ev6XHw998pRWG728z
    ```

```


