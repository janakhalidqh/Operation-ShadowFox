# ShadowFox — Phase 2: Active Directory Setup \& Validation

> \*\*Lab Notice:\*\* All credentials used in this lab are disposable test credentials and must never be reused in production environments.

## 1\. Lab Environment

|Machine|Role|IP|DNS|
|-|-|-|-|
|DC01|Domain Controller / DNS|`10.10.10.10`|`10.10.10.10` after promotion|
|SRV01|Member Server|`10.10.10.20`|`10.10.10.10`|
|WS01|Windows 11 Workstation|`10.10.10.30`|`10.10.10.10`|
|Domain|Active Directory|`shadowfox.local`|—|
|NetBIOS|Domain|`SHADOWFOX`|—|

All VMs should use the same **Internal Network**.

## 

### **ShadowFox — Phase 2 Commands**



#### **1. DC01 Network**



New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.10.10.10 -PrefixLength 24



Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 127.0.0.1



After AD/DNS promotion:



Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.10.10.10



#### **2. Install AD DS + DNS**



Install-WindowsFeature AD-Domain-Services -IncludeManagementTools



Install-ADDSForest `

&#x20; -DomainName "shadowfox.local" `

&#x20; -DomainNetbiosName "SHADOWFOX" `

&#x20; -InstallDns `

&#x20; -Force



#### **3. Organizational Units**



New-ADOrganizationalUnit -Name "ShadowFox" -Path "DC=shadowfox,DC=local"



New-ADOrganizationalUnit -Name "Workstations" `

&#x20; -Path "OU=ShadowFox,DC=shadowfox,DC=local"



New-ADOrganizationalUnit -Name "Servers" `

&#x20; -Path "OU=ShadowFox,DC=shadowfox,DC=local"



New-ADOrganizationalUnit -Name "Users" `

&#x20; -Path "OU=ShadowFox,DC=shadowfox,DC=local"



New-ADOrganizationalUnit -Name "ServiceAccts" `

&#x20; -Path "OU=ShadowFox,DC=shadowfox,DC=local"



#### **4. Domain Users**



$ouUsers = "OU=Users,OU=ShadowFox,DC=shadowfox,DC=local"



$pw = ConvertTo-SecureString "P@ssw0rd-Lab-2025!" -AsPlainText -Force



New-ADUser -Name "Sara Al-Otaibi" `

&#x20; -SamAccountName "s.alotaibi" `

&#x20; -UserPrincipalName "s.alotaibi@shadowfox.local" `

&#x20; -Path $ouUsers `

&#x20; -AccountPassword $pw `

&#x20; -Enabled $true



New-ADUser -Name "Omar Al-Harbi" `

&#x20; -SamAccountName "o.alharbi" `

&#x20; -UserPrincipalName "o.alharbi@shadowfox.local" `

&#x20; -Path $ouUsers `

&#x20; -AccountPassword $pw `

&#x20; -Enabled $true



New-ADUser -Name "Helpdesk User" `

&#x20; -SamAccountName "helpdesk" `

&#x20; -UserPrincipalName "helpdesk@shadowfox.local" `

&#x20; -Path $ouUsers `

&#x20; -AccountPassword $pw `

&#x20; -Enabled $true



#### **5. Domain Admin**



New-ADUser -Name "IT Admin" `

&#x20; -SamAccountName "it.admin" `

&#x20; -UserPrincipalName "it.admin@shadowfox.local" `

&#x20; -Path "OU=Users,OU=ShadowFox,DC=shadowfox,DC=local" `

&#x20; -AccountPassword $pw `

&#x20; -Enabled $true



Add-ADGroupMember -Identity "Domain Admins" -Members "it.admin"



#### **6. SQL Service Account + SPN**



New-ADUser -Name "SQL Service" `

&#x20; -SamAccountName "svc\_sql" `

&#x20; -UserPrincipalName "svc\_sql@shadowfox.local" `

&#x20; -Path "OU=ServiceAccts,OU=ShadowFox,DC=shadowfox,DC=local" `

&#x20; -AccountPassword $pw `

&#x20; -Enabled $true `

&#x20; -PasswordNeverExpires $true



setspn -S MSSQLSvc/srv01.shadowfox.local:1433 svc\_sql



#### **7. Member DNS**



Run on WS01 and SRV01:



Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.10.10.10



#### **8. Join SRV01 to Domain**



Run on SRV01:



Add-Computer -DomainName "shadowfox.local" -Credential (Get-Credential) -Restart



#### **9. Join WS01 to Domain**



Run on WS01:



Add-Computer -DomainName "shadowfox.local" -Credential (Get-Credential) -Restart



#### **10. Move Computer Objects**



Run on DC01:



Get-ADComputer WS01 |

&#x20; Move-ADObject -TargetPath "OU=Workstations,OU=ShadowFox,DC=shadowfox,DC=local"



Get-ADComputer SRV01 |

&#x20; Move-ADObject -TargetPath "OU=Servers,OU=ShadowFox,DC=shadowfox,DC=local"

