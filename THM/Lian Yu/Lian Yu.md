![image](https://user-images.githubusercontent.com/99112106/174281485-42c10ba6-1ec3-4a3b-83ed-a91c07c30ee9.png)

# Lian Yu

## 1.Enumeration 

Let's start with a simple nmap scan:

```
nmap -sC -sV -Pn -oA nmap/Lian_Yu 10.10.93.4 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-04 16:08 CEST
Nmap scan report for 10.10.93.4
Host is up (0.037s latency).
Not shown: 996 closed tcp ports (reset)
PORT    STATE SERVICE VERSION
21/tcp  open  ftp     vsftpd 3.0.2
22/tcp  open  ssh     OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)
| ssh-hostkey: 
|   1024 56:50:bd:11:ef:d4:ac:56:32:c3:ee:73:3e:de:87:f4 (DSA)
|   2048 39:6f:3a:9c:b6:2d:ad:0c:d8:6d:be:77:13:07:25:d6 (RSA)
|   256 a6:69:96:d7:6d:61:27:96:7e:bb:9f:83:60:1b:52:12 (ECDSA)
|_  256 3f:43:76:75:a8:5a:a6:cd:33:b0:66:42:04:91:fe:a0 (ED25519)
80/tcp  open  http    Apache httpd
|_http-title: Purgatory
|_http-server-header: Apache
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          41818/udp6  status
|   100024  1          50356/tcp6  status
|   100024  1          58551/tcp   status
|_  100024  1          60545/udp   status
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.63 seconds
```

We have ports 21, 22, 80 and 111.
* * *
## 2. Listing directories
Let's search some hidden directories with gobuster:

```
gobuster dir -w /usr/share/dirb/wordlists/big.txt -u http://10.10.93.4
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.93.4
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirb/wordlists/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/04/04 16:14:07 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 199]
/.htpasswd            (Status: 403) [Size: 199]
/island               (Status: 301) [Size: 233] [--> http://10.10.93.4/island/]
/server-status        (Status: 403) [Size: 199]                                
                                                                               
===============================================================
2022/04/04 16:15:25 Finished
===============================================================
```

Okay, we have just discovered "island" directory. Let's have a lookt!

![image](https://user-images.githubusercontent.com/99112106/174281535-b08819f4-e566-4b42-bc04-dab776a299cd.png)

We have an username named "Lian_Yu" and a code word "vigilante"... I'm going to keep it and now i'll run gobuster again to see if there are more hidden directories under "/island":

```
gobuster dir -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://10.10.93.4/island/
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.93.4/island/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/04/04 16:34:16 Starting gobuster in directory enumeration mode
===============================================================
/2100                 (Status: 301) [Size: 238] [--> http://10.10.93.4/island/2100/]
```

Yes! We found another hidden directory called "/2100"... Rare af. The website looks like this:

![image](https://user-images.githubusercontent.com/99112106/174281589-fef0eac4-4f7a-41c1-a18d-c39215aa098c.png)

If we look at the source code from that web page we find this:

![image](https://user-images.githubusercontent.com/99112106/174281634-c3ce6fcd-2cde-4c7b-a3ae-7ed3f371a0b7.png)

Hmm it looks like some file is being used as the extension .ticket. How about another gobuster scan inside this directory but this time we will add the extension ‘.ticket’ to specifically search for. Perhaps we get what we are looking for.

```
gobuster dir -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://10.10.93.4/island/2100/ -x php,html,txt,ticket
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.93.4/island/2100/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt,ticket,php,html
[+] Timeout:                 10s
===============================================================
2022/04/04 16:34:09 Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 292]
/green_arrow.ticket   (Status: 200) [Size: 71] 
```

Okay! We have discovered "green_arrow.ticket" (??) Let's have a look at it:

![image](https://user-images.githubusercontent.com/99112106/174281698-816bfe42-e8e2-4269-9ca1-76493a1b1195.png)

After a while, i discovered that the token was encoded in Base58:

![image](https://user-images.githubusercontent.com/99112106/174281750-1410d11e-bb10-4c5f-a78e-a3687ba61b2b.png)

This password is actually FTP's password!! 
* * *
## 3. Searching via FTP
Using the username "vigilante" we found above, we managed to logged in via FTP, as you can see.

![image](https://user-images.githubusercontent.com/99112106/174281827-5b13053a-6ab2-4426-b27c-fd622606fe78.png)

Get all the files you can get. There is a hidden file called ".other_user" that can be interesting to have a look...

![image](https://user-images.githubusercontent.com/99112106/174281858-39679cdf-eddc-4858-8985-7fa198cbd09b.png)

Okay, it's not as interesting as I thought it was going to be... But we can list another user, "slade". We'll keep it in mind. Let's have a look at the other files:

![image](https://user-images.githubusercontent.com/99112106/174281926-6c871df4-ec44-498a-969c-1e227a0ad101.png)
* * *
## 4. Steganography
As you can see, we have 3 images and one of them says it's a data file (???)... Interesting. If we try to visualize it:

![image](https://user-images.githubusercontent.com/99112106/174281968-144feabe-a47e-4063-8802-360089792360.png)

It displays an error message. Maybe because the magic bytes are set wrong. Let's change that so we can display it right. These are the magic bytes for a PNG file:

![image](https://user-images.githubusercontent.com/99112106/174282008-bfc3d16d-6ec4-49b2-afd9-75627650be10.png)

We're going to use this command:

```
hexeditor Leave_me_alone.png
```

Replace this:

![image](https://user-images.githubusercontent.com/99112106/174282043-4ac462aa-cc9f-49ee-978e-88d3f9c8b358.png)

For this:

![image](https://user-images.githubusercontent.com/99112106/174282069-e91ae799-d09d-4bfa-9cf7-cf06d510e7cd.png)

We can now display the image:

![image](https://user-images.githubusercontent.com/99112106/174282111-2b4acee2-98d5-4ea8-8769-0862ba18a717.png)

The other photos are:
- Queen's Gambit.png

![image](https://user-images.githubusercontent.com/99112106/174282143-851d0102-ef24-4a83-8582-2be939dff7c1.png)

- aa.jpg

![image](https://user-images.githubusercontent.com/99112106/174282232-66e5cf34-2a39-4bbd-bebe-c52823595e33.png)

We're going to try to extract hidden data from the images using the password we found before restoring the image:

![image](https://user-images.githubusercontent.com/99112106/174282260-b55bb8fa-7e16-408f-be6d-b554e8f8d051.png)

As we saw above, we managed to extract a zip file. Let's see what is inside that zip file:

![image](https://user-images.githubusercontent.com/99112106/174282281-68ea5eea-3c36-4b68-8665-d07619fbf548.png)

It seems that the file called "shado" contains a ssh's password for the user we listed before!! 
* * *

## 5. Foothold
Let's try to logged in via SSH:

![image](https://user-images.githubusercontent.com/99112106/174282304-f29c5f10-495a-488c-ad0e-fea42f5e1595.png)

We are in!! We can retrieve the user flag:

![image](https://user-images.githubusercontent.com/99112106/174282320-5902d018-ef5d-49f2-9d3d-bc6476d2a424.png)
* * *

## 6. Privileges Escalation

First of all, we're going to check what files we can run with sudo privileges.

![image](https://user-images.githubusercontent.com/99112106/174282333-2ebae15d-60c1-4097-8163-d07e9e8c8acc.png)

Okay... We can run pkexec with sudo privileges... Let's see what GTFOBins have to say about that...

![image](https://user-images.githubusercontent.com/99112106/174282351-05001b1d-b5bc-4ea6-92e5-b4f8f5a3f740.png)

Let's try that!

![image](https://user-images.githubusercontent.com/99112106/174282374-3156fee5-102b-4d6c-bc86-c44f3dd29d63.png)

Okay!! That worked perfectly! We're root and we can retrieve the root flag.

Congratz!! =)
