# Task 2 Physical Active Directory

What database does the AD DS contain?
Answer: `ntds.dit`

Where is the NTDS.dit stored?
Answer: `%SystemRoot%\NTDS`

What type of machine can be a domain controller?
Answer: `windows server`

# Task 3 The Forest

What is the term for a hierarchy of domains in a network?
Answer: `tree`

What is the term for the rules for object creation?
Answer: `domain schema`

What is the term for containers for groups, computers, users, printers, and other OUs?
Answer: `Organizational Unit`

# Task 4 Users + Groups

Which type of groups specify user permissions?
Answer: `Security groups`

Which group contains all workstations and servers joined to the domain?
Answer: `domain computers`

Which group can publish certificates to the directory?
Answer: `cert publishers`

Which user can make changes to a local machine but not to a domain controller?
Answer: `Local administrator`

Which group has their passwords replicated to read-only domain controllers?
Answer: `Allowed RODC Password Replication Group`

# Task 5 Trusts + Policies

What type of trust flows from a trusting domain to a trusted domain?
Answer: `directional`

What type of trusts expands to include other trusted domains?
Answer: `transitive`

# Task 6 Active Directory Domain Services + Authentication

What type of authentication uses tickets?
Answer: `kerberos`

What domain service can create, validate, and revoke public key certificates?
Answer: `certificate services`

# Task 7 AD in the Cloud

What is the Azure AD equivalent of LDAP?
Answer: `rest apis`

What is the Azure AD equivalent of Domains and Forests?
Answer: `Tenants`

What is the Windows Server AD equivalent of Guests?
Answer: `trusts`

# Task 8 Hands-On Lab

Deploy the Machine

What is the name of the Windows 10 operating system?
Using https://github.com/PowerShellMafia/PowerSploit/

`get-netcomputer -fulldata | select name,operatingsystem`

Answer: `Windows 10 Enterprise Evaluation`

When was the password last set for the  SQLService user?


What is the second "Admin" name?
`get-aduser -filter * | select name`

Answer: `Admin2`

Which group has a capital "V" in the group name?
`get-adgroup -filter 'name -like "*V*"' | select name`

Answer: `Hyper-V Administrators`


When was the password last set for the SQLService user?
`get-aduser -identity sqlservice -properties * |select passwordlastset`

Answer: `5/13/2020 8:26:58 PM`