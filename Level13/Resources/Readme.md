# Level13

## How to Login

To access the system, use the following SSH command:

```bash
ssh level13@<IP_ADDRESS> -p 4242
password: g1qKMiRpXf53AWhDaU7FEkczr
```

## How to Get the Flag

Upon logging in, you'll find an executable named `level13`. This challenge involves manipulating the program's execution using a debugger (`gdb`).

### Analyzing the Executable

Running `level13` directly produces the following output:

```
UID 2013 started us but we we expect 4242
```

This message indicates that the program checks the user ID (UID) and expects it to be 4242, but it's currently running with UID 2013 (the UID of `level13`). We download the executable using:

```bash
scp level13@<IP_ADDRESS>:/home/user/level13 ./
```

### Debugging with GDB

We'll use `gdb` to debug the executable, modify its behavior, and bypass the UID check.  Here's a step-by-step breakdown of the debugging process:

1.  **Start GDB:**

    ```bash
    gdb ./level13
    ```

2.  **Disassemble `main`:**  Examine the assembly code of the `main` function:

    ```gdb
    disassemble main
    ```

    This produces the following output (important parts highlighted):

    ```assembly
    Dump of assembler code for function main:
       0x0804858c <+0>:	push   %ebp
       0x0804858d <+1>:	mov    %esp,%ebp
       0x0804858f <+3>:	and    $0xfffffff0,%esp
       0x08048592 <+6>:	sub    $0x10,%esp
       0x08048595 <+9>:	call   0x8048380 <getuid@plt>      # getuid() is called
       0x0804859a <+14>:	cmp    $0x1092,%eax            # Compare eax (UID) with 0x1092 (4242)
       0x0804859f <+19>:	je     0x80485cb <main+63>     # Jump if equal
       0x080485a1 <+21>:	call   0x8048380 <getuid@plt>
       0x080485a6 <+26>:	mov    $0x80486c8,%edx
       0x080485ab <+31>:	movl   $0x1092,0x8(%esp)
       0x080485b3 <+39>:	mov    %eax,0x4(%esp)
       0x080485b7 <+43>:	mov    %edx,(%esp)
       0x080485ba <+46>:	call   0x8048360 <printf@plt>
       0x080485bf <+51>:	movl   $0x1,(%esp)
       0x080485c6 <+58>:	call   0x80483a0 <exit@plt>
       0x080485cb <+63>:	movl   $0x80486ef,(%esp)
       0x080485d2 <+70>:	call   0x8048474 <ft_des>      # Call ft_des()
       0x080485d7 <+75>:	mov    $0x8048709,%edx
       0x080485dc <+80>:	mov    %eax,0x4(%esp)
       0x080485e0 <+84>:	mov    %edx,(%esp)
       0x080485e3 <+87>:	call   0x8048360 <printf@plt>
       0x080485e8 <+92>:	leave
       0x080485e9 <+93>:	ret
    End of assembler dump.
    ```

    *   **`getuid()`:**  The `getuid()` function is called at `<+9>`.  The result (the UID) is stored in the `%eax` register.
    *   **Comparison:** At `<+14>`, the value in `%eax` is compared with `0x1092` (4242 in decimal).
    *   **Conditional Jump:** At `<+19>`, a `je` (jump if equal) instruction jumps to `<main+63>` if the UID is 4242.  This is the code path we want to take. If not equal, the program prints the error message and exits.
    *   **`ft_des()`:** If the jump is taken, the `ft_des()` function is called at `<+70>`. This function likely performs the decryption.

3.  **Set a Breakpoint:** Set a breakpoint *before* the comparison to modify the value in `%eax`:

    ```gdb
    break *0x0804859a
    ```

4.  **Run the Program:**

    ```gdb
    run
    ```

    The program will stop at the breakpoint.

5.  **Examine the UID:** Check the current value of `%eax` (it will be 2013):

    ```gdb
    x/i $pc       # Show current instruction
    display/d $eax  # Display eax in decimal
    ```

6.  **Modify `%eax`:**  Change the value of `%eax` to 4242 (0x1092):

    ```gdb
    set $eax=0x1092
    ```

7.  **Verify the Change:**

    ```gdb
    display/d $eax  # Check that eax is now 4242
    ```

8.  **Continue Execution:**

    ```gdb
    continue
    ```

    The program will now continue execution, taking the jump to `<main+63>`, calling `ft_des()`, and printing the decrypted token:

    ```
    your token is 2A31L79asukciNyi8uppkEuSx
    [Inferior 1 (process ...) exited with code ...]
    ```
    You have the token, so there is no need to execute `getflag`.

You have successfully completed Level13 by using `gdb` to modify the program's execution and bypass the UID check.
```


