# CTF TELECOM Nancy 2024


![Static Badge](https://img.shields.io/badge/Télécom-Projet_scolaire-purple)    
![Static Badge](https://img.shields.io/badge/Networking-blue?logo=network)    

**Note**: This readme is also available in [french](nolink)

## Présentation


This document is my report for the Maxime Clementz's (PwC Luxembourg) CTF in TELECOM Nancy (2024)


### First flag
For the first flag, we're asked what are reflexes should be when connecting to a LAN. We therefore do a packet capture in order to check what's exchanged with connected machines. When opening the resulting `pcap file`, we can easily spot the flag:
[screenshot](https://github.com/cos-imo/TN_2024_CTF_report/blob/main/first_flag_wireshark.png)


#### Fourth flag
After discovering that samba runs on the Ubuntu machine, the next logical step is to try to connect to it:

```
smbclient -N -L 10.0.0.20
Sharename  Type  Comment
---------  ----  -------
print$     Disk  Printer Driver
samshare   Disk  Samba on ubuntu flag4{MoreInfoFromSMB!!}
```
We can now submit the flag: `flag4{MoreInfoFromSMB!!}`
