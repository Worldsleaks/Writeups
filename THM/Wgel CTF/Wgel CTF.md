![image](https://user-images.githubusercontent.com/99112106/174284175-3c50b849-29d2-44f0-9805-4215d57b4e73.png)

# Wgel CTF

## 1.Enumeration 

Let's start with a simple nmap scan:

```
nmap -sC -sV -oA nmap/WgetCTF 10.10.98.19
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-05 08:40 CEST
Nmap scan report for 10.10.98.19
Host is up (0.083s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:96:1b:66:80:1b:76:48:68:2d:14:b5:9a:01:aa:aa (RSA)
|   256 18:f7:10:cc:5f:40:f6:cf:92:f8:69:16:e2:48:f4:38 (ECDSA)
|_  256 b9:0b:97:2e:45:9b:f3:2a:4b:11:c7:83:10:33:e0:ce (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.32 seconds
```

We have opened ports like port 22 (ssh) and port 80 (http).
* * *
## 2.  Exploring the website

If we have a look we can see the default apache web page.

![image](https://user-images.githubusercontent.com/99112106/174284187-6f80e6f7-14a0-46a6-a658-b32c64532e39.png)

But, despite running the default apache web page, we can find an interesting comment in the source code:

![image](https://user-images.githubusercontent.com/99112106/174284196-fc40e384-9fd0-42cd-b572-763a5d4b916a.png)

Okey, we have an username, "Jessie", and it says the website is outdated. Let's run Gobuster to find some hidden directories:

```
gobuster dir -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://10.10.98.19 -x php,html,txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.98.19
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
2022/04/05 09:01:32 Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 11374]
/sitemap              (Status: 301) [Size: 312] [--> http://10.10.98.19/sitemap/]
```

It didn't bring us much but we have an interesting directory "sitemap". Let's visit it:

![image](https://user-images.githubusercontent.com/99112106/174284218-9eddad1d-5e59-4453-a821-81b3779b3e7b.png)

We have a normal web page promoting some kind of CMS template or something. Let's run gobuster again:

```
gobuster dir -w /usr/share/dirb/wordlists/common.txt -u http://10.10.98.19/sitemap/ -x php,html,txt   
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.98.19/sitemap/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirb/wordlists/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
2022/04/05 09:13:58 Starting gobuster in directory enumeration mode
===============================================================
/.hta.html            (Status: 403) [Size: 276]
/.hta.txt             (Status: 403) [Size: 276]
/.hta                 (Status: 403) [Size: 276]
/.htaccess            (Status: 403) [Size: 276]
/.hta.php             (Status: 403) [Size: 276]
/.htpasswd            (Status: 403) [Size: 276]
/.htaccess.php        (Status: 403) [Size: 276]
/.htpasswd.php        (Status: 403) [Size: 276]
/.htaccess.html       (Status: 403) [Size: 276]
/.htaccess.txt        (Status: 403) [Size: 276]
/.htpasswd.html       (Status: 403) [Size: 276]
/.htpasswd.txt        (Status: 403) [Size: 276]
/.ssh                 (Status: 301) [Size: 317] [--> http://10.10.98.19/sitemap/.ssh/]
/about.html           (Status: 200) [Size: 12232]                                     
/blog.html            (Status: 200) [Size: 12745]                                     
/contact.html         (Status: 200) [Size: 10346]                                     
/css                  (Status: 301) [Size: 316] [--> http://10.10.98.19/sitemap/css/] 
/fonts                (Status: 301) [Size: 318] [--> http://10.10.98.19/sitemap/fonts/]
/images               (Status: 301) [Size: 319] [--> http://10.10.98.19/sitemap/images/]
/index.html           (Status: 200) [Size: 21080]                                       
/index.html           (Status: 200) [Size: 21080]                                       
/js                   (Status: 301) [Size: 315] [--> http://10.10.98.19/sitemap/js/]    
/services.html        (Status: 200) [Size: 10131]                                       
/shop.html            (Status: 200) [Size: 17257]                                       
/work.html            (Status: 200) [Size: 11428]                                       
                                                                                        
===============================================================
2022/04/05 09:15:09 Finished
===============================================================
```

We have managed to find a really interesting directory called ".ssh":

![image](https://user-images.githubusercontent.com/99112106/174284236-8f388814-4254-45b4-a8a7-ebca055ee585.png)

Inside it, we can retrieve a SSH private key:

![image](https://user-images.githubusercontent.com/99112106/174284251-991f09dc-02e5-486b-9923-097616dac64f.png)
* * *
## 3. Foothold

Okay, let's try to login with Jessie via SSH:

![image](https://user-images.githubusercontent.com/99112106/174284266-b521ee4c-b188-4fdd-b67f-073eb100b66a.png)

Yes! We're in Jessie's SSH account.

![image](https://user-images.githubusercontent.com/99112106/174284276-d88901e1-3d39-4782-9795-2fa281c0802b.png)

There we go. There is our first flag.
* * *
## 4. Privileges Escalation

Let's start out way to root!! First of all, we're going to check what files we can run with sudo privileges.

![image](https://user-images.githubusercontent.com/99112106/174284304-11f49095-4357-4b07-ba88-83393c62ca92.png)

It seems that we can run "wget" command with sudo privileges. Let's check out GTFOBins:

![image](https://user-images.githubusercontent.com/99112106/174284315-dd833156-a795-40df-a62f-8f91763255a4.png)

Mmmmm, it looks like we can send us files from the targeted machine to ours. So let's use wget command to send us the root flag, just like that:

![image](https://user-images.githubusercontent.com/99112106/174284328-02d1fac2-f21e-41c1-bc3c-dba58a65291c.png)

An there we go! We have just recieved the root flag!!

![image](https://user-images.githubusercontent.com/99112106/174284337-0ce75dad-c6dd-46f4-8c22-1d2a7aa838dd.png)
