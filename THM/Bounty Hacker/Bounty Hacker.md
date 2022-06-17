![image](https://user-images.githubusercontent.com/99112106/174274818-f7607b5c-a962-46ae-81b3-31c99b1d1263.png)

# BountyHacker

## 1.Enumeration 

Let's start with a simple nmap scan:

```
nmap -sC -sV -oA nmap/bountyhacker 10.10.81.78
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-04 09:00 CEST
Nmap scan report for 10.10.81.78
Host is up (0.036s latency).
Not shown: 967 filtered tcp ports (no-response), 30 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.0.152
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:f8:df:a7:a6:00:6d:18:b0:70:2b:a5:aa:a6:14:3e (RSA)
|   256 ec:c0:f2:d9:1e:6f:48:7d:38:9a:e3:bb:08:c4:0c:c9 (ECDSA)
|_  256 a4:1a:15:a5:d4:b1:cf:8f:16:50:3a:7d:d0:d8:13:c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.87 seconds
```

As we can see from the output, we have 3 open ports; ftp, ssh and HTTP. 
* * *
## 2. FTP Anonymous login
Since we have anonymous login allowed via FTP, we're going to have a look there:

![image](https://user-images.githubusercontent.com/99112106/174274875-018eb4e9-d0e4-4065-a52c-114c61bb303d.png)

We are going to retrieve the files we have just found. Let's take a look at these files:

![image](https://user-images.githubusercontent.com/99112106/174274909-f537c5a2-e649-43b4-8d10-38ecab92cea3.png)

It seems like "locks.txt" is actually a wordlist so i guess we'll have to do somekind of bruteforcing attack. We also can see the username "lin" at the bottom of "task.txt" file. That's interesting.
* * *
## 3. SSH Bruteforce

We have a username and a wordlist. It is worth trying to brute force the ssh with hydra.

![image](https://user-images.githubusercontent.com/99112106/174274960-da104f3a-c693-4ee1-aeba-61e29e1d2933.png)

Yes! We have ssh credentials for lin. 

* * *
## 4. First Foothold

Let's login via SSH to see if we can continue our way to root.

![image](https://user-images.githubusercontent.com/99112106/174275028-3a4c49f4-3399-44df-af5c-d9ec65b581f0.png)

Okay, we are in and we can retrieve our first flag!

![image](https://user-images.githubusercontent.com/99112106/174275077-de82de3b-f195-4189-ae38-d467d63e2ae9.png)

* * *
## 5. Privilege Escalation

First of all, we're going to check what files we can run with sudo privileges.

![image](https://user-images.githubusercontent.com/99112106/174275130-01c3a977-203b-48f6-978f-c0a7b0492beb.png)

It looks like we have sudo privs for tar's binary. Let's check it out in GTFObins.

![image](https://user-images.githubusercontent.com/99112106/174275181-7003bfa2-9fe0-496c-9093-e221a65a6fe5.png)

Let's see if it works...

![image](https://user-images.githubusercontent.com/99112106/174275222-d77e37ff-938d-43e0-8088-6c579b4c4609.png)

Yep, that worked fine. We're root and our CTF ends with this. Let's retrieve our final flag!

![image](https://user-images.githubusercontent.com/99112106/174275274-e5520a67-fe93-41f3-b3a2-66b1a4a44559.png)
