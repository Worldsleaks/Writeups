![image](https://user-images.githubusercontent.com/99112106/174272089-5e4f147f-0616-4041-8247-ffb3ba66da78.png)

# Pickle Rick

## 1.Enumeration 

Let's start with a simple scanning ports tool such as Nmap.

```
nmap -sV -A 10.10.205.214
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-31 09:30 CEST
Nmap scan report for 10.10.205.214
Host is up (0.036s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d8:ab:68:d8:58:0d:4c:25:7c:7c:1f:3a:58:1c:6a:ed (RSA)
|   256 a1:6c:fe:8f:fe:a9:77:a7:d0:60:b2:49:b0:d4:23:19 (ECDSA)
|_  256 ab:61:f0:1e:f9:3f:93:51:21:78:b6:72:b9:24:44:c6 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Rick is sup4r cool
|_http-server-header: Apache/2.4.18 (Ubuntu)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=3/31%OT=22%CT=1%CU=35243%PV=Y%DS=2%DC=T%G=Y%TM=624558A
OS:D%P=x86_64-pc-linux-gnu)SEQ(SP=100%GCD=1%ISR=10E%TI=Z%II=I%TS=8)SEQ(SP=1
OS:00%GCD=1%ISR=10E%TI=Z%CI=I%II=I%TS=8)OPS(O1=M506ST11NW7%O2=M506ST11NW7%O
OS:3=M506NNT11NW7%O4=M506ST11NW7%O5=M506ST11NW7%O6=M506ST11)WIN(W1=68DF%W2=
OS:68DF%W3=68DF%W4=68DF%W5=68DF%W6=68DF)ECN(R=Y%DF=Y%T=40%W=6903%O=M506NNSN
OS:W7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%D
OS:F=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O
OS:=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W
OS:=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%R 
OS:IPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 443/tcp)
HOP RTT      ADDRESS
1   34.65 ms 10.9.0.1
2   34.96 ms 10.10.205.214

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.32 seconds

```

As always in an entry-level CTF, we have ssh and http running on our target host. We can assume that we have to retrieve ssh's credentials from the web. 
* * *
## 2.Exploring the website
Let's visit the website:

![image](https://user-images.githubusercontent.com/99112106/174272142-415d55a1-d91d-4730-bef1-69d3642539b3.png)

If we check the source code from the index page we can retrieve an username!! 

![image](https://user-images.githubusercontent.com/99112106/174272174-413930b8-5efe-47d9-92cf-8a5fd8035ab8.png)

Okay, we have an username, now we need a password. Let's run gobuster to see if it finds some interesting files or directories.

```
gobuster dir -w /usr/share/dirb/wordlists/big.txt -u http://10.10.205.214
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.205.214
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirb/wordlists/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/03/31 10:06:57 Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 297]
/.htaccess            (Status: 403) [Size: 297]
/assets               (Status: 301) [Size: 315] [--> http://10.10.205.214/assets/]
/robots.txt           (Status: 200) [Size: 17]                                    
/server-status        (Status: 403) [Size: 301]                                   
                                                                                  
===============================================================
2022/03/31 10:08:14 Finished
===============================================================

```

The robots.txt file is enable so let's check it out.

![image](https://user-images.githubusercontent.com/99112106/174272225-e73eb664-0368-42e8-810c-a2d8fcdb8aa7.png)

Is this a password or just a rabbit hole? We'll find out later, i guess. We also found the "assets" directory. This only contains the images of the website that hopefully do not have any hidden content. After testing other dictionaries with gobuster, I discovered a login which looks like we can login with the credentials we found before.

![image](https://user-images.githubusercontent.com/99112106/174272265-6f7a0a49-f3a7-4106-a0ca-e59d28eb8b0a.png)

Once inside we find a web page with different categories but the one we are interested in is this first one. It seems that we can insert code in it.

![image](https://user-images.githubusercontent.com/99112106/174272294-6f2e5180-3b5e-40aa-87e6-b1252a0ffe81.png)

Interesting. Okay, we have the first ingredient for the CTF:

![image](https://user-images.githubusercontent.com/99112106/174272353-d34d0dfd-979b-43eb-990d-d3fcbf6643da.png)

We also have a clue. Let's read it:

![image](https://user-images.githubusercontent.com/99112106/174272409-62d33e82-9676-4aa1-9e21-fdf5c07d9d46.png)

Actually, this is funny. We can't use 'cat', let's try other alternatives in order to read it:

![image](https://user-images.githubusercontent.com/99112106/174272444-1cf3eb7d-fb08-4e6f-9f44-0bf6d584e11e.png)

'Less' command worked! I also tried with more, tail and head without luck. Okay, it tells us to search through the system to get the second ingredient.

![image](https://user-images.githubusercontent.com/99112106/174272473-09408c07-3f67-48d6-b4f7-5bf73fdcd238.png)

After a while, we found the second ingredient in Rick's home path. 
* * *
## 3. Getting a shell

At this point we are going to try to get a reverse shell by injecting code into the command line that gives us. Set up a random listenning port:

![image](https://user-images.githubusercontent.com/99112106/174272520-41bd7164-803f-46c8-87c3-c5a61b60f85a.png)

And then, inject the reverse shell:

![image](https://user-images.githubusercontent.com/99112106/174272573-26406aca-eea4-44dd-8e82-56323e45ede2.png)

It worked!

![image](https://user-images.githubusercontent.com/99112106/174272608-10293d3d-b713-40c3-8bf1-9abb7738ab9a.png)

* * *
## 4. Privilege Escalation
Okay, that was a very quick privilege escalation! We look for the binaries that we can run with SUID privileges and furthermore, we realize that we can run everything with sudo!!!!! 

![image](https://user-images.githubusercontent.com/99112106/174272630-e914314a-a80e-4c07-9d09-c01bbee9ba9e.png)

Once again we rely on GTFObins to facilitate privilege escalation.

![image](https://user-images.githubusercontent.com/99112106/174272691-1b29e798-c480-4a42-b483-b28ee93db397.png)

Finally, we can get the final ingredient:

![image](https://user-images.githubusercontent.com/99112106/174272728-a2b82a3c-6af1-4605-9d7f-7bf72507ceba.png)
