# CTF TELECOM Nancy 2024


![Static Badge](https://img.shields.io/badge/Télécom-Projet_scolaire-purple)    
![Static Badge](https://img.shields.io/badge/Networking-blue?logo=network)    

> [!CAUTION]
> This write-up deviates from the standard CTF workflow; instead, it follows a more intuitive, natural progression, and thus some flags might not be in the 'right' order

**Note**: This readme is also available in [french](nolink)   
**Note**: Don't like links? Me neither. They're all in the source code of this document. If, for some reason, you do not have it, simply check the end of the document

## Présentation

This document is my report for the Maxime Clementz's (PwC Luxembourg) CTF in TELECOM Nancy (2024)


### First flag
For the first flag, we're asked what are reflexes should be when connecting to a LAN. We therefore do a packet capture in order to check what's exchanged with connected machines. When opening the resulting `pcap file`, we can easily spot the flag in the data section: `flag1{Bien-joué}`
![screenshot first flag](https://github.com/cos-imo/TN_2024_CTF_report/blob/main/first_flag_wireshark.png)

### Second flag

### Third flag

Until now we have only run basic nmap scans. Those display very few informations and only scan the 1000 most common ports. Let's now try to run a scan on all ports, and ask it to display :

``` 
$ sudo nmap -A -p- 10.0.0.20
[sudo] Mot de passe de kali :
Starting Nmap 7.93 (https://nmap.org ) at 2024-11-25 18:47 CET
Nmap scan report for 10.0.0.20
Host is up (0.00071s latency).
Not shown: 65533 closed tep ports (reset)
PORT      STATE     SERVICE      VERSION
445/tcp   open      netbios-ssn  Samba smbd 4.6.2
2221/tcp  open      ssh          (protocol 2.0)
|  fingerprint-strings:
|    NULL:
|_     SSH-2.0-flag3{v3rsiOn}Ubuntu-4ubuntu0.5
|  ssh-hostkey:
|    3072 67f47fed20cb9373a3b21dfd7e644c29 (RSA)
|    256 f864ae029c8b3dbd1b3fcfa05c9fd29a (ECDSA)
|_   256 4533ba12e8ed1eeed2bfb22ee1649191 (ED25519)
...
(Useless output has been truncated)
```

We can then spot the third flag in the version string: `flag3{v3rsiOn}`

### Fourth flag
After discovering that samba runs on the Ubuntu machine, the next logical step is to take a look at all the availables shares:

```
$ smbclient -N -L 10.0.0.20
Sharename  Type  Comment
---------  ----  -------
print$     Disk  Printer Driver
samshare   Disk  Samba on ubuntu flag4{MoreInfoFromSMB!!}
```
We can now submit the flag: `flag4{MoreInfoFromSMB!!}`

### Fifth flag
Now that we have enough informations about this Samba server, let's try to connect:
```
$ smbclient //10.0.0.20/sambashare
Password for [WORKGROUP\kali]:
try "help" to get a list of possible commands.
smb: \> ls
.                  D          0  Mon   Jan  16  23:51:30  2023
..                 D          0  Mon   Nov  25  09:47:49  2024
tail-sshd-config   N        320  Mon   Jan  16  23:51:01  2023
netstat.out        N        594  Mon   Jan  16  23:51:00  2023
passwords.dic      N     332586  Mon   Jan  16  23:51:01  2023
tail-sshd-config   N        320  Mon   Jan  16  23:51:01  2023
          19947120 blocks of size 1024. 11102060 blocks available

smb: \> get .flag.txt
getting file \.flag.txt of size 27 as .flag.txt (3,3 KiloBytes/sec) (average 3,3 KiloBytes/sec)
smb: \> exit

$ cat .flag.txt
flag5{BienvenueEtranger:)}
```

### Seventh flag
Well, we've got a list of passwords... One of them must be a password for a challenge, right? We've noticed that flags are all in the form `flagN{VALUE}`. Let's check if we can find a string containing 'flag' in the file:

```
$ cat passwords.dic | grep flag
flag7{Ca_Vallait_Le_Coup_De_Regarder_Hein?}
```


### ???th flag

While we're connected to the sftp server, download the avilable files:
```
smb: \> ls
.                  D          0  Mon   Jan  16  23:51:30  2023
..                 D          0  Mon   Nov  25  09:47:49  2024
tail-sshd-config   N        320  Mon   Jan  16  23:51:01  2023
netstat.out        N        594  Mon   Jan  16  23:51:00  2023
passwords.dic      N     332586  Mon   Jan  16  23:51:01  2023
          19947120 blocks of size 1024. 11102060 blocks available

smb: \> get ./*
```

So we downloaded 3 files: an except of [ssh server config file](https://linux.die.net/man/1/ssh) (`tail-sshd-config`), the outcast of a [netstat](https://linux.die.net/man/8/netstat) command (`netstat.out`), and what appears to be a list of password (`passwords.dic`)

> [!NOTE]
> Here, since we were already connected, it was easier to use [sftp's `get` command](https://man7.org/linux/man-pages/man1/sftp.1.html), but we could simply have [mounted](https://man7.org/linux/man-pages/man8/mount.8.html) the samba share partition on our system:
> ```
> $ mkdir /mnt/samba
> $ sudo mount -t cifs -o username=kali //10.0.0.20/sambashare /tmp/samba
> ```
> Please note that doing so [is not recommended](https://en.wikipedia.org/wiki/Tunneling_protocol#Secure_Shell_tunneling)


```
-$ ssh -p 2221 10.0.0.20
** Hello moi-même **
cified as IP addresses or hostnames. You can also. rk/bits (e.g. 192.168.1.0/24) to specify all hosts and broadcast addresses included), or
-192.168.1.27) to specify all hosts in the
255.255.0) to
Note pour plus tard, si jamais j'oublie mon mot de passe ...
Il se trouve dans passwords.dic ;) Attention si je les essaie tous je risque de bloquer mon compte: s
Comme je suis pas mauvais avec grep, si je retire ceux
be used both on the file option.
qui ne terminent pas par 'S' et qui n'ont pas de 'R', il ne devrait en rester que 12 dans le dictionnaire fourni, able options. juste en dessous de la limite failban du serveur :D
Sinon je risque de me faire bannir mon IP ou même bloquer ills/arp-scan mon compte utilisateur sur le serveur!
Ensuite
"online password attack" avec l'outil hydra par exemple.
kali@10.0.0.20's password:
```

 - ssh documentation on linux.die.net (there is a part about config files): https://linux.die.net/man/1/ssh
 - netstat documentation on linux.die.net: https://linux.die.net/man/8/netstat
 - sftp documentation on man7: https://man7.org/linux/man-pages/man1/sftp.1.html
 - mount documentation on man7: https://man7.org/linux/man-pages/man8/mount.8.html
