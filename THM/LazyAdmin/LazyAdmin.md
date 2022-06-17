![image](https://user-images.githubusercontent.com/99112106/174275806-44c08176-3ec7-4644-a115-e0b11b192263.png)

# LazyAdmin

## 1.Enumeration 

Let's start with a simple nmap scan:

```
nmap -sC -sV -Pn -oA nmap/lazyadmin 10.10.248.175
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-04 09:35 CEST
Nmap scan report for 10.10.248.175
Host is up (0.10s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
|   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
|_  256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel 

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.70 seconds
``` 

Okay, we only have two open ports; SSH and HTTP. 

* * *
## 2.  Exploring the website

If we have a look we can see the default apache web page.

![image](https://user-images.githubusercontent.com/99112106/174275846-8ce51ae4-27e7-43a9-ba87-81b3e54754b5.png)

This brings us nothing usefull. Let's use gobuster to try and pull out the hidden directories.

```
gobuster dir -w /usr/share/dirb/wordlists/big.txt -u http://10.10.248.175 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.248.175
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirb/wordlists/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/04/04 09:39:24 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/content              (Status: 301) [Size: 316] [--> http://10.10.248.175/content/]
/server-status        (Status: 403) [Size: 278]                                    
                                                                                   
===============================================================
2022/04/04 09:40:48 Finished
===============================================================
```

We found an interesting directory called "content", let's have a look on that:

![image](https://user-images.githubusercontent.com/99112106/174275912-ecd42807-0062-444a-bfe5-82689390e09a.png)

Okay, it looks like the "lazyAdmin" is using SweetRice as his website management system. We can try to find the CMS's version to see if we can find out some already reported vulnerability. 

After some researching, i decided that we are going to run Gobuster again to see if we can find more hidden directories under "content". We're probably missing something.

```
gobuster dir -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://10.10.248.175/content/
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.248.175/content/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/04/04 09:56:46 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 323] [--> http://10.10.248.175/content/images/]
/js                   (Status: 301) [Size: 319] [--> http://10.10.248.175/content/js/]    
/inc                  (Status: 301) [Size: 320] [--> http://10.10.248.175/content/inc/]   
/as                   (Status: 301) [Size: 319] [--> http://10.10.248.175/content/as/]    
/_themes              (Status: 301) [Size: 324] [--> http://10.10.248.175/content/_themes/]
/attachment           (Status: 301) [Size: 327] [--> http://10.10.248.175/content/attachment/]
                                                                                              
===============================================================
2022/04/04 10:10:59 Finished
===============================================================
```

Okay!! We have already found some more interesting directories. Let's have a look at them to see what we can find. 

We found in a txt file the CMS's version, that is worth keeping.

![image](https://user-images.githubusercontent.com/99112106/174275946-5665a978-e484-422c-a25d-86776451457a.png)

After digging through all the content, i found a mysql backup, let's check it out:

![image](https://user-images.githubusercontent.com/99112106/174275971-0a870c5a-9d92-40ab-a117-cf15b5be3fa1.png)

It seems that the mysql backup file contains credentials!

![image](https://user-images.githubusercontent.com/99112106/174276026-c8c908f7-9c29-4a90-964a-0556e7fe956a.png)

Okay, we found a login page! We have an username called manager and a MD5 hashed password. Let's crack it:

![image](https://user-images.githubusercontent.com/99112106/174276051-75a566ac-e93b-423a-ac92-400b6512f7c4.png)

Yes! Now we have some credentials that we can use in the login page that we found in the second listing with gobuster.

![image](https://user-images.githubusercontent.com/99112106/174276087-f31d7c3c-1bad-4bb2-830b-f71b685b4b44.png)

* * *
## 3. Foothold

We are in the main dashboard!

![image](https://user-images.githubusercontent.com/99112106/174276124-c72a5f19-dae7-47c5-a85c-9a19b16c94cc.png)

We can see the administrator control panel to manage and configure the CMS. After a bit of fiddling with the website, I discovered that we can upload ad's. So, we are going to use this to upload a reverse shell:

![image](https://user-images.githubusercontent.com/99112106/174276155-fd83a218-290d-4b78-bf13-cb1dea62ebae.png)

We can find our reverse shell in the "/content/inc/ads" directory. Remember to open a listenning port on your attacking machine!

![image](https://user-images.githubusercontent.com/99112106/174276181-ffe328e5-d841-4bf5-87dd-b9c16d0b4bbd.png)

Okay, we're the webserver!!

![image](https://user-images.githubusercontent.com/99112106/174276222-1aa96df2-fe13-4a71-834c-b6e648c25a42.png)

First of all, let's upgrade our shell with Python:

![image](https://user-images.githubusercontent.com/99112106/174276259-e5716e9f-e907-4dcf-9885-748252ce6221.png)

Inside Itguy's home path, we can find the user flag:

![image](https://user-images.githubusercontent.com/99112106/174276284-b686532b-62da-41ee-83e2-3fbdd91b6aef.png)

* * *
## 4. Privileges Escalation

First of all, we're going to check what files we can run with sudo privileges.

![image](https://user-images.githubusercontent.com/99112106/174276330-5fc62037-aa9e-42f4-aba1-d13ac139fc5f.png)

That is, unexpected, to say the least. Turns out that "www-data" has been given permission to run a perl script with root privileges. Maybe we can leverage this to get a root shell? Lets take a look on that perl script:

![image](https://user-images.githubusercontent.com/99112106/174276359-d2995427-a2b8-446b-84cb-9e905a381098.png)

The script is actually running another script, "/etc/copy.sh". If we examine the privileges of that script, we find that we can read,write and execute it, nice.

If we have a look on that script, it already contains a reverse shell (????). The only thing you have to do is to change the IP address and the port (optional) where you want to recieve the root shell.

![image](https://user-images.githubusercontent.com/99112106/174276594-ace1a3ab-c05b-4bde-adef-49eeb4443a34.png)

Since "www-data" don't have any decent text editor, we'll use echo, as you see above. 

After setting up our listenning port, execute the perl backup with sudo command on the vulnerable machine:

![image](https://user-images.githubusercontent.com/99112106/174276722-720fd2a1-5221-436c-bfa2-a338d3407283.png)

Just like that, we would have recieved a root shell in our listenning port and we can pull out the root flag.

![image](https://user-images.githubusercontent.com/99112106/174276749-9c7b4f1b-41db-4cc4-8487-982047fa1d5e.png)

Congratz!
