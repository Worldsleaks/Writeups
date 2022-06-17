![image](https://user-images.githubusercontent.com/99112106/174278594-e55e360f-ba48-47c6-abc4-d1fe97a89f26.png)

# Brooklyn Nine Nine

## 1.Enumeration 

Let's start with a simple nmap scan:

```
nmap -sC -sV -Pn -oA nmap/BrooklynNineNine 10.10.219.243
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-04 15:24 CEST
Nmap scan report for 10.10.219.243
Host is up (0.072s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
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
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 16:7f:2f:fe:0f:ba:98:77:7d:6d:3e:b6:25:72:c6:a3 (RSA)
|   256 2e:3b:61:59:4b:c4:29:b5:e8:58:39:6f:6f:e9:9b:ee (ECDSA)
|_  256 ab:16:2e:79:20:3c:9b:0a:01:9c:8c:44:26:01:58:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.73 seconds

```

Okay, we have FTP, SSH and HTTP. 
* * *
## 2.  Exploring FTP

As you can see above from the nmap's output, anonymous access via FTP is allowed so we're going to have a look in there:

![image](https://user-images.githubusercontent.com/99112106/174278644-76921e3c-b8a8-4f8c-9d46-8d4f323cfdfd.png)

There is a note so we're going to get it. Once we're out from FTP, read it:

![image](https://user-images.githubusercontent.com/99112106/174278665-a645d1fc-072b-4523-aae8-4628f81b3eab.png)

Okay, we have 3 usernames (Amy, jake and holt) and Jake has a weak password. 

* * *
## 3. Bruteforcing with Hydra

We can try to guest Jake's password with hydra. In order to do that, we are going to use the wordlist "rockyou":

![image](https://user-images.githubusercontent.com/99112106/174278696-73513bcb-5048-4a6b-92f7-9a1a53f6462b.png)

That was fast!! It took just a few minutes to guest it.

* * *
## 4. Foothold

Now that we have SSH credentials for Jake, we're going to logged in:

![image](https://user-images.githubusercontent.com/99112106/174278733-6a3e9a95-1c1b-4646-b649-99cd09658f10.png)

Once we're in, we can see that the tree users listed aboved, already exists and have home directories:

![image](https://user-images.githubusercontent.com/99112106/174278751-3bf701ca-a538-4e7e-8378-098c56b20ce5.png)

Inside Holt's home directory, we found the user flag!

![image](https://user-images.githubusercontent.com/99112106/174278790-f1ce88ed-86d0-4022-b93e-c07f6c69b6be.png)

* * *
## 5. Privileges Escalation

Let's start out way to root!! First of all, we're going to check what files we can run with sudo privileges.

![image](https://user-images.githubusercontent.com/99112106/174278823-80b61595-0378-48b6-9df4-658217a6f4dc.png)

Okay, we can execute "less" comand with sudo privileges. Let's see what GTFOBins have to say...

![image](https://user-images.githubusercontent.com/99112106/174278857-566693d3-61e6-4952-b809-b92a0fa191a3.png)

Okay, let's try that...

![image](https://user-images.githubusercontent.com/99112106/174278890-856662de-85f2-4297-aea8-b4751c489a96.png)

That worked and we are root!!! As you can see above we can already retrieve root flag. Congratz.


* * *

## Bonus Track

When we visited the website we saw this:

![image](https://user-images.githubusercontent.com/99112106/174278920-f0f201b2-77f2-4141-8639-e87625f58c9c.png)

An inmense photo about the TV show, nothing rare...But if we have a look at the source code we can see it mentions something about steganography... 

![image](https://user-images.githubusercontent.com/99112106/174278947-c3a4afc7-37e2-4d3c-a3e5-f6e1f4352aec.png)

It seems that the background image from the web page has some hidden data... Let's try to extract it with **steghide**.

![image](https://user-images.githubusercontent.com/99112106/174278975-ec78f843-56a1-46c6-bedd-5d98dce71e84.png)

Mmmm we need a phrase in order to extract the data... Let's try to bruteforce our entry in with stegcracker:

![image](https://user-images.githubusercontent.com/99112106/174279012-ca24e886-ef55-400e-a692-ed6d6bb9d75a.png)

Yep! That worked! The phrase is "admin" and now we can extract the hidden data:

![image](https://user-images.githubusercontent.com/99112106/174279031-f62f963c-79e3-49b0-8434-db76593f0296.png)

As you can see above, we managed to extract a txt file with Holt's password!! We didn't need it a long the way but it was fun to discover too =)
