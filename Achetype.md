# Enumeration
Enumeration is defined as the process of extracting user names, machine names, network resources, shares and services from a system. Performing a network scan to detect what ports are open is as an essential part of the enumeration process. This network scan offers us the opportunity to better understand the attacking surface and design targeted attacks.
```
nmap -sC -sV {TARGET_IP}

-sC : Default script scan to extract more information about target system
-sV : Probe open ports to determine service/version info (useful for targeted exploits on older versions
```
<img src="https://user-images.githubusercontent.com/106356256/199875374-2c73361e-16bd-4049-9f3f-2c8b98e470a2.png" style="height: 800px; width:700px;"/>

As we can see SMB ports are open and a Microsoft SQL server is running. We can enumerate further using ```smbclient```

```
smbclient -N -L \\\\{TARGET_IP}\\
```
```
-N : No password
-L : This option allows you to look at what services are available on a server
```

<img src="https://user-images.githubusercontent.com/106356256/199875440-db34e865-7ec7-444d-aea5-b9fd6fa371b4.png" style="height: 250px; width:300px;"/>

After connecting to the smb server we see some interesting shares. Attempting to access the ADMIN$ and C$ shares is ineffective, as credentials are required. However, attempting to access the backups share is a success. We find a single file:

```prod.dtsConfig``` 

<img src="https://user-images.githubusercontent.com/106356256/199875478-c431409b-fa91-439b-96dd-0df2d84c3c45.png" style="height: 150px; width:550px;"/>

Upon researching the dtsConfig file extension, we find the following: 

*"A DTSCONFIG file is an XML configuration file used to apply property values to SQL Server Integration Services (SSIS) packages. The file contains one or more package configurations that consist of metadata such as the server name, database names, and other connection properties to configure SSIS packages."*

This file could potentially provide some useful information about the MSSQL server we saw on ```port 1433``` so we grab it using get for further examination.

<img src="https://user-images.githubusercontent.com/106356256/199875513-4b42d7f0-d872-48e2-a01a-94fd3bb08678.png" style="height: 100px; width:800px;"/>

Upon examining the file we find a password and its corresponding id in plain text. We now have potentially valid credentials for accessing the MSSQL server. We just need some way to connect and authenticate with the server and we will be able to view what is stored in the database.

Luckily there exists a repository of python classes that may have what we are looking for.
```
Impacket is a collection of Python classes for working with network protocols. Impacket is focused on providing low-level programmatic access to the packets and for some protocols (e.g. SMB1-3 and MSRPC) the protocol implementation itself. Packets can be constructed from scratch, as well as parsed from raw data, and the object oriented API makes it simple to work with deep hierarchies of protocols. The library provides a set of tools as examples of what can be done within the context of this library.
```
Using the impacket script ```mssqlclient.py``` we are able to connect and authenticate with the server after providing the found credentials.

<img src="https://user-images.githubusercontent.com/106356256/199875544-300ef7ab-00b3-48ee-a9c1-c56c51c49219.png" style="height: 200px; width:550px;"/>

# Foothold
Now that we are succesfully in the SQL server we should check our privilages and see what we are able to do. It is important to note that it is impossible to know all the possible exploits and commands for all possible systems off the cuff. It is wise to consult and use external help and resources when possible. Personally, this was the first time i've ever encountered and interacted with MSSQL, so some research was required. The following links were extremely helpful in getting me up to speed on what was possible when pentesting on MSSQL.

https://book.hacktricks.xyz/network-services-pentesting/pentesting-mssql-microsoft-sql-server

https://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet

<img src="https://user-images.githubusercontent.com/106356256/199875561-a2f2b640-9f78-45b5-8f79-45a7a48f9674.png" style="height: 100px; width:550px;"/>

Using some commands from the cheatsheet we can see that our aquired credentials were that of the role 'sysadmin' which has the highest privilages, so no privilage escalation is required. Our next step is to aquire the ability to execute commands on our target system. We can use the procedure ```xp_cmdshell``` to do so but it is disabled. Luckily we have the role of 'sysadmin' so we can simply enable it using some commands found in the cheatsheet. After enabling ```xp_cmdshell``` we check if it is working by running a simple whoami command. The command is successful and so we can move on to the next stage.


<img src="https://user-images.githubusercontent.com/106356256/199875593-8c9ea83b-cb39-45bf-a0c7-14377e0186fe.png" style="height: 150px; width:200px;"/>

Now that we have command execution on the target system the next logical step is to establish a reverse shell so that we have full access to the system and can find and download our flags.

Firstly, we download a netcat binary with the goal of getting our target to download it and execute it. We start a simple HTTP server on our system which will receive and complete the download request from our target. We also open a netcat listen which will complete our reverse shell.

Before we can request the netcat binary on our target system we must find a suitable folder to download it in, as not all of them will have the correct access. ```C:\Users\sql_svc\Downloads``` seems like a good candidate. 

<img src="https://user-images.githubusercontent.com/106356256/199875645-4cff6a01-ea3d-43a5-bd51-2923bb43e7c8.png" style="height: 200px; width:700px;"/>

We can verify if our request was successful by looking at our http server, and we can see the GET request successfully came through. 

<img src="https://user-images.githubusercontent.com/106356256/199875664-d9a409a4-080c-44ea-b212-8b7140214815.png" style="height: 100px; width:400px;"/>

We then execute the netcat binary onto the same port we opened our netcat listener on. We have now succesfully established a reverse shell on our target.

<img src="https://user-images.githubusercontent.com/106356256/199875688-a2ebaef6-2fbb-483d-9fc3-5df3af11f9ff.png" style="height: 150px; width:600px;"/>

After rummaging around sql_svc 's directories we find that the user flag is located on the desktop (as is usually the case for HTB)

<img src="https://user-images.githubusercontent.com/106356256/199875722-d79aed25-d8d4-4837-a66a-d10f8bbcb44f.png" style="height: 250px; width:300px;"/>


# Privilege Escalation
Now that we have found our user flag it is time to find the administrator flag. This will most likely be on the targets system, in ```C:\Users\Administrator``` which we cannot access with our current privilages. We must therefore escalate our privilages.

An excellent tool for windows privilege escalation is WinPEAS. Running WinPEAS on our target system will provide us with alot of information regarding possible exploits and avenues of attack to escalate our privilages with. We can use our simple HTTP server to once again make the target download our binary like we did for netcat.

<img src="https://user-images.githubusercontent.com/106356256/199875736-5fd821d6-95e8-4175-afed-9aa8041f8252.png" style="height: 250px; width:500px;"/>

Running WinPEAS on the target system provides us with a large output. For example, it shows that we have SeImpersonatePrivilege. Which is vulnerable to the juicy potato exploit. This can be used to escalate our privilages. But before we do that we should have a look at all the avenues that will be faster and simpler to investigate before we commit to this exploit.

<img src="https://user-images.githubusercontent.com/106356256/199875761-25289d64-8653-4a60-91ce-28dcddc67499.png" style="height: 250px; width:500px;"/>

We can see that WinPEAS has noticed the file ```ConsoleHost_history.txt``` has been accessed frequently. This file is a powershell command history file and could contain some useful information.

<img src="https://user-images.githubusercontent.com/106356256/199875788-afc1a003-de59-43fd-b3f7-d76156500e61.png" style="height: 100px; width:700px;"/>

As we can see the file did contain some interesting information. That being the credentials to the Administrator account. We can now use a tool such as ```Evil-Winrm``` (if port 5985 is open) to use these credentials to gain an admin shell and find the final admin flag. Using nmap again we can see that port 5985 is indeed open.

<img src="https://user-images.githubusercontent.com/106356256/199875807-b7d3e487-3394-4064-a150-d971ddb2fa2b.png" style="height: 700px; width:500px;"/>

We have successfully gained an admin shell. Navigating to /Desktop/ provides us with the final flag. 

