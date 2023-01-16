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
 cme smb <ip> -u 'a' -p '' # enumerate anonymous access
 ```
- OSINT: you can use open source intelligence to get a list of valid usernames depending on various [active directory naming conventions](https://activedirectorypro.com/active-directory-user-naming-convention/)

- LDAP: `ldapsearch -LLL -x -H ldap://test.local -b'' -s base '(objectclass=\*)'`
- LDAP: `nmap -n -sV --script "ldap* and not brute" -p 389 <dc-ip>` 
- Enum4Linux: `enum4linux -U <dc-ip> | grep 'user:'`
- SMB: Enumerate anonymous and null session: 
```bash
smbclient -U '%' -L //<dc-ip> && smbclient -U 'guest%' -L //<dc-ip>
```
- Once you craft a list of possible usernames, you can perform nmap kerberoas bruteforce to get valid ones:  
```bash
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
PS: Crackmapexec is prefered

- [Perform SMB Relay]


With credentials 

Use the credentials in you favor enumerate usefull information you might you use in further attacks.

- Get other users. `python3 GetADUsers.py -all test.local/john:password123 -dc-ip 10.10.10.1`
- Get almost all information about an AD envirement:
 ```bash
 crackmapexec smb 10.10.10.1 -u 'john' -p 'password123' --groups --local-groups --loggedon-users --rid-brute --sessions --users --shares --pass-pol
 ```
- Find computers you can login to. 

Enumerate following:

− Users
− Computers
− Domain Administrators
− Enterprise Administrators
− Shares


