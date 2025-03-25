# 42 Cursus - Snow Crash

This repository documents our progress and solutions for the "Snow Crash" security project from the 42 Cursus. Snow Crash is a series of Capture the Flag (CTF) challenges that introduce various cybersecurity concepts. This README provides an overview of each level, focusing on the core vulnerability we exploited.

**Disclaimer:** This repository is for educational purposes. The solutions presented here should not be used to exploit systems without explicit permission.

## Project Overview

The Snow Crash project is based on a virtual machine (VM) environment. Each level represents a different user account (`levelXX`) with increasing privileges.  Our goal was to find a vulnerability that allows access to the `flagXX` account and retrieval of a token using the `getflag` command.

## General Instructions

1.  **Accessing the VM:** The VM is accessed via SSH on port 4242. The general format: `ssh levelXX@<IP_ADDRESS> -p 4242`. The initial login is `level00:level00`.

2.  **Finding the Flag:** Once we exploited a level and gained access to `flagXX`, we ran the `getflag` command to retrieve the token (usually the password for the *next* level).

3.  **Tools:** Many levels could be solved using only VM tools, but some required us to download files to our separate machines.

## Levels

### Level 00

*   **Vulnerability:** 🔐 A file accessible to `flag00` contains a Caesar-ciphered password. The vulnerability lies in the insecure storage of a weakly encrypted password.  We used the `find` command, combined with our knowledge of the expected user (`flag00`), to discover this file. The Caesar cipher itself is a very weak form of encryption, easily broken with online tools or manual analysis.

### Level 01

*   **Vulnerability:** 🗄️ The encrypted password hash for `flag01` is stored in the publicly readable `/etc/passwd` file. This file is world-readable, meaning any user on the system can view it. While the passwords are *hashed*, they are not *salted* and are vulnerable to dictionary attacks using tools like John the Ripper. The weakness of the `descrypt` algorithm used for hashing further facilitates cracking.

### Level 02

*   **Vulnerability:** 📡 Sensitive information (a password) is transmitted in cleartext over the network and captured in a `.pcap` file. Additionally, the password, although slightly obfuscated, contains easily identifiable control characters (DEL, 0x7f) that reveal the simple substitution method used. The vulnerability lies in the lack of encryption for network communication and the weak password obfuscation.

### Level 03

*   **Vulnerability:** 💻 The `level03` binary relies on the `echo` command without specifying its full path. This makes it vulnerable to PATH manipulation. By creating a malicious executable named `echo` in a directory earlier in the `PATH` than the standard `/bin` directory, we could hijack the execution flow and run arbitrary code (in this case, `/bin/bash`).

### Level 04

*   **Vulnerability:** ⌨️ The `level04.pl` script, a CGI script, is vulnerable to command injection. The script takes user input (`param("x")`) and directly embeds it within backticks in a shell command: `` `echo $y 2>&1` ``. Backticks in Perl (and many other scripting languages) execute the enclosed string as a shell command. Since there's no input sanitization or validation, an attacker can inject arbitrary commands.

### Level 05

*   **Vulnerability:** ⏰ A cron job runs a script (`/usr/sbin/openarenaserver`) that executes *any* file placed in the `/opt/openarenaserver/` directory and then deletes it. This allows us to place a malicious script in that directory, knowing it will be executed with the privileges of the user running the cron job (`flag05`). The vulnerability is the combination of an overly permissive cron job and a writable directory.

### Level 06

*   **Vulnerability:** 💉 The `level06.php` script (and its compiled binary) uses the deprecated `/e` modifier in the `preg_replace` function. The `/e` modifier treats the replacement string as PHP code, allowing for arbitrary code execution if an attacker can control the input to `preg_replace`. This is a classic code injection vulnerability, made particularly dangerous by the use of a deprecated and insecure feature.

### Level 07

*   **Vulnerability:** ⚙️ The `level07` executable uses the `LOGNAME` environment variable without any sanitization and passes it to the `/bin/echo` command. This creates a command injection vulnerability. By setting `LOGNAME` to a string containing a semicolon and a malicious command (e.g., `; /bin/bash`), we could execute arbitrary code with the privileges of the program (which is setuid to `flag07`).

### Level 08

*   **Vulnerability:** 🔗 The `level08` binary checks if the provided filename argument is "token", but it doesn't verify the *path* to the file. This allows us to create a symbolic link (symlink) named "token" that points to the actual token file, bypassing the intended access restriction. The program only checks the *name* of the file passed as an argument, not its contents or location.

### Level 09

*   **Vulnerability:** ➕➖ The `level09` binary implements a predictable character transformation. It adds the index of each character in the input string to its ASCII value. This is a very weak form of "encryption" and is easily reversible. The vulnerability lies in using a deterministic and easily guessable algorithm for protecting sensitive data.

### Level 10

*   **Vulnerability:** ⏳ The `level10` binary is vulnerable to a TOCTOU (Time-of-Check to Time-of-Use) race condition. The program first checks if the user has access to a file (using `access()`) and then opens the file (using `open()`). Between these two operations, we could quickly replace the file with a symbolic link, tricking the program into opening a different file than the one it checked. This is a classic race condition vulnerability.

### Level 11

*   **Vulnerability:** 🌐 The `level11.lua` script, a network service, contains a command injection vulnerability in the `hash` function. The function takes user input and directly concatenates it into a shell command string without any sanitization: `` io.popen("echo "..pass.." | sha1sum", "r") ``. This allows an attacker to inject arbitrary commands by including a semicolon (`;`) in the input.

### Level 12

*   **Vulnerability:** 🐚 While direct command injection is mitigated by input sanitization in `level12.pl`, the script's logic can be bypassed. The script uses `egrep "^$xx" /tmp/xd`, where `$xx` is a modified version of user input. By crafting an input that includes backticks and a wildcard (e.g., `` `/*/EXPLOIT` ``), we could execute an arbitrary script located in a predictable location, even though the intended search target (`/tmp/xd`) is not affected. The core vulnerability is the misuse of `egrep` with user-controlled input, even after sanitization.

### Level 13

*   **Vulnerability:** 🔨 The `level13` binary enforces a UID check, expecting to be run with UID 4242. This is not a vulnerability in the traditional sense, but rather a challenge that requires manipulating the program's execution environment. The vulnerability, in a broader sense, is that the program's security relies on an easily modifiable runtime condition (the UID). Using a debugger like `gdb` allows direct manipulation of the program's memory and registers, bypassing the intended check.

### Level 14

*   **Vulnerability:** 🛠️ The `/bin/getflag` binary implements two security measures: an anti-debugging check using `ptrace` and a UID check. Both of these are checks that can be bypassed using a debugger. The `ptrace` check detects if the program is being debugged and exits if it is. The UID check verifies that the program is being run by a specific user (`flag14`). The vulnerabilities, again, are the reliance on runtime conditions that can be altered by attackers with sufficient privileges and tools. *Further experimentation with the UID modification technique reveals additional possibilities.* 😉

## Contributing

Feel free to submit pull requests with improvements, corrections, or alternative solutions.
