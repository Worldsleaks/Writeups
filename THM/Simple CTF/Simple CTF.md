![image](https://user-images.githubusercontent.com/99112106/174273034-a18474cf-8864-4144-9529-8217af6f45d8.png)

# Simple CTF

## 1.Enumeration 

Let's start with a simple scanning ports tool such as Nmap.
```
nmap -sV -A -Pn 10.10.123.79                                                                                1 ⚙
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-31 11:47 CEST
Nmap scan report for 10.10.123.79
Host is up (0.034s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.0.218
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
|_http-server-header: Apache/2.4.18 (Ubuntu)
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
|_  256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (ED25519)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.10 - 3.13 (92%), Crestron XPanel control system (90%), ASUS RT-N56U WAP (Linux 3.4) (87%), Linux 3.1 (87%), Linux 3.16 (87%), Linux 3.2 (87%), HP P2000 G3 NAS device (87%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (87%), Linux 5.4 (86%), Linux 2.6.32 (86%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   33.43 ms 10.9.0.1
2   33.55 ms 10.10.123.79

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 50.65 seconds

```

Okay, we have ftp with anonymous login enable, http and ssh running on 2222 port, nice try.
* * *
## 2. Searching in FTP

Let's log in as anonymous in ftp and see what we can find out.

![image](https://user-images.githubusercontent.com/99112106/174273088-3951ee11-aa3b-49ec-ab59-e739cfdd760a.png)

We found out there is a txt file for Mitch. By the way, FTP is configured as extended passive mode, that means it'll took a while to connect. Lets see what is inside that txt file:

![image](https://user-images.githubusercontent.com/99112106/174273124-10c6905e-dd28-4b59-9b65-ad2bb4597a48.png)

Okay, it seems there is a weak password out there that we can crack easily. We'll find out later on, i guess.
* * *
## 3. Cracking with Hydra

Okay, we found that there's a weak password for the user called "mitch". That give us enough clues to try to brute force the ssh password for mitch's account. 

![image](https://user-images.githubusercontent.com/99112106/174273159-ec265dfe-8f26-4eae-bece-2e8e492bf364.png)

After a while, we found out the password!!
* * *
## 4. Getting into SSH

Lets login with the ssh's credentials we already have:

![image](https://user-images.githubusercontent.com/99112106/174273197-e092bd21-c88f-4884-a2b8-14765d4969a9.png)

We are logged in and we can get the user flag. Just like that.
* * *
## 5. Privileges Escalation

We can run vim with sudo privileges. That give us an enourmos advantage.

![image](https://user-images.githubusercontent.com/99112106/174273746-4794d0b5-5de7-493a-a09a-196e9ed4069a.png)

Let's find out what GTFObins have to say about vim:

![image](https://user-images.githubusercontent.com/99112106/174273761-1ca0a4da-5834-4942-9e04-a04c1b51fd7c.png)

Okay, let's give that a try! 

![image](https://user-images.githubusercontent.com/99112106/174273795-f2a08f6b-3341-4f0e-aeb1-b1fb198bc4d7.png)

That worked!! So now we're root and we can pull out the last flag.
