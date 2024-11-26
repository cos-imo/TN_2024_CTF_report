# CTF TELECOM Nancy 2024


![Static Badge](https://img.shields.io/badge/Télécom-Projet_scolaire-purple)    
![Static Badge](https://img.shields.io/badge/Networking-blue?logo=network)    

**Note**: This readme is also available in [french](nolink)

## Présentation


This document is my report for the Maxime Clementz's (PwC Luxembourg) CTF in TELECOM Nancy (2024)


### First flag
For the first flag, we're asked what are reflexes should be when connecting to a LAN. We therefore do a packet capture in order to check what's exchanged with connected machines. When opening the resulting `pcap file`, we can easily spot the flag in the data section:
![screenshot](https://github.com/cos-imo/TN_2024_CTF_report/blob/main/first_flag_wireshark.png)

### Second flag

### Third flag


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
