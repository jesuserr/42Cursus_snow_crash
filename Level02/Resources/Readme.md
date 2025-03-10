# Level02

## How to login

```
ssh level02@192.168.56.104 -p 4242
password: f2av5il02puano7naaf6adaaf
```

## How to get the flag

Once logged, we find a .pcap file called 'level02.pcap' in the home folder. PCAP files contain packet data of a network and are used to analyze the network communications. In order to open this file we will use Wireshark which is the world's most popular network protocol analyzer.

Since Wireshark is installed in our KALI virtual machine, we need to copy the file from the SnowCrash VM to our KALI VM using secure file copy through ssh, with the utility 'scp'. When asked for password we need to introduce the login password shown above.
```
┌──(kali㉿kali)-[~]
└─$ scp -P 4242 level02@192.168.56.104:level02.pcap .
           _____                      _____               _     
          / ____|                    / ____|             | |    
         | (___  _ __   _____      _| |     _ __ __ _ ___| |__  
          \___ \| '_ \ / _ \ \ /\ / / |    | '__/ _` / __| '_ \ 
          ____) | | | | (_) \ V  V /| |____| | | (_| \__ \ | | |
         |_____/|_| |_|\___/ \_/\_/  \_____|_|  \__,_|___/_| |_|
                                                        
  Good luck & Have fun

          192.168.56.104 
level02@192.168.56.104's password: 
level02.pcap                               100% 8302     2.6MB/s   00:00    
                                                                             
┌──(kali㉿kali)-[~]
└─$ 
```

If the copy was successful, we will have the .pcap file in our KALI home folder and then we will proceed to open it with Wireshark. Unfortunately we cannot open it since we have no permissions to read the file, therefore we will modify the rights using the command 'chmod'.
```
┌──(kali㉿kali)-[~]
└─$ chmod u+r level02.pcap
```

Once the file is loaded we will see all the communications held between the client and server (a total of 95 transactions of information using TCP protocol). Wireshark, provides tools to analyze the entire conversation in a more readable format. We go to Analyze -> Follow -> TCP Stream, and then we are provided with a summary of the conversation in ASCII format, where the following catches up our attention.
```
Password: 
ft_wandr...NDRel.L0L
```

We proceed to introduce the password using 'su flag02' but it fails to authenticate. Paying more attention to the password we decide to make an Hexdump to see exactly which is the content, and then, we notice that 0x7f is the code for DEL, so it means that for each 0x7f code we need to delete a letter.
```
    000000D6  00 0d 0a 50 61 73 73 77  6f 72 64 3a 20            ...Passw ord: 
000000B9  66                                                 f
000000BA  74                                                 t
000000BB  5f                                                 _
000000BC  77                                                 w
000000BD  61                                                 a
000000BE  6e                                                 n
000000BF  64                                                 d
000000C0  72                                                 r
000000C1  7f                                                 .
000000C2  7f                                                 .
000000C3  7f                                                 .
000000C4  4e                                                 N
000000C5  44                                                 D
000000C6  52                                                 R
000000C7  65                                                 e
000000C8  6c                                                 l
000000C9  7f                                                 .
000000CA  4c                                                 L
000000CB  30                                                 0
000000CC  4c                                                 L
000000CD  0d                                                 .
```

So, we guess that the password should be 'ft_waNDReL0L' and this time we have success
```
level02@SnowCrash:~$ su flag02
Password: 
Don't forget to launch getflag !
flag02@SnowCrash:~$ getflag
Check flag.Here is your token : kooda2puivaav1idi4f57q8iq
flag02@SnowCrash:~$ 
```
