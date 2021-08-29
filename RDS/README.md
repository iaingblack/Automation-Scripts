Basic script to do the following.

# DC

## Rename Machine

```powershell
$machineName = 'RDS-DC'
Rename-Computer -NewName $machineName -Restart
```

## Change to a Static IP - Usually Ethernet0 in VMWare Workstation VMs

```powershell
$nicName = 'Ethernet0'
$address = '192.168.29'
$gw = "$($address).2"
New-NetIPAddress –InterfaceAlias $nicName -AddressFamily IPv4 –IPAddress $ip –PrefixLength 24 -DefaultGateway $gw
$nic = Get-NetIPAddress -AddressFamily IPv4 -InterfaceAlias 'Ethernet0'
Set-DnsClientServerAddress -InterfaceIndex $nic.InterfaceIndex -ServerAddresses ($gw, "8.8.8.8")
```

## Create a Domain Controller

```powershell
$netbiosName = 'rds'
$domainName = "$($netbiosName).local"
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
Import-Module ADDSDeployment
Install-ADDSForest `
-CreateDnsDelegation:$false `
-DatabasePath "C:\Windows\NTDS" `
-DomainMode "WinThreshold" `
-DomainName "$($netbiosName).local" `
-DomainNetbiosName $netbiosName `
-ForestMode "WinThreshold" `
-InstallDns:$true `
-LogPath "C:\Windows\NTDS" `
-NoRebootOnCompletion:$false `
-SysvolPath "C:\Windows\SYSVOL" `
-Force:$true
```

## Create Group and Add Users

```
New-ADGroup -Name RDS-Users -GroupScope DomainLocal -GroupCategory Security
$password = 'Password1' | ConvertTo-SecureString -AsPlainText -Force
$username = 'rds-user'
$users = 1..4
foreach ($user in $users) {
    Write-Host "Creating User: '$($username)$($user)'"
    New-ADUser -Name "$($username)$($user)" -GivenName "$($username)$($user)" -Surname "$($username)$($user)" -SamAccountName "$($username)$($user)" -UserPrincipalName "$($username)$($user)" -AccountPassword $password -Enabled $true
    Add-ADGroupMember -Identity RDS-Users -Members "$($username)$($user)"
}
```

# Member Servers

## Enable RDP

```powershell
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -value 0
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

## Change DNS of a Network Adaptor - Usually Ethernet0 in VMWare Workstation VMs

```powershell
$nicName = 'Ethernet0'
$address = '192.168.29'
$dcIP = "$($address).10"
$nic = Get-NetIPAddress -AddressFamily IPv4 -InterfaceAlias 'Ethernet0'
Set-DnsClientServerAddress -InterfaceIndex $nic.InterfaceIndex -ServerAddresses ($dcIP, "8.8.8.8")
```

## Join Machine to Domain and Rename (doesnt quite work to join domain, to investigate)

```powershell
$machineName = 'RDS-NewMachine'
Rename-Computer -NewName $machineName -DomainCredential rds.local\administrator -Restart
```
