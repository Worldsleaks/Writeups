![image](https://user-images.githubusercontent.com/99112106/174270790-ff1ff69e-c44c-438d-b52d-015637b561b5.png)

# RootMe

## 1.Enumeration 

Let's start with a simple scanning ports tool such as Nmap.

```
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-30 15:33 CEST
Nmap scan report for 10.10.97.10
Host is up (0.073s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
|   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
|_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: HackIT - Home
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.29 (Ubuntu)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=3/30%OT=22%CT=1%CU=42976%PV=Y%DS=2%DC=T%G=Y%TM=62445C3
OS:2%P=x86_64-pc-linux-gnu)SEQ(SP=105%GCD=1%ISR=10B%TI=Z%CI=Z%II=I%TS=A)OPS
OS:(O1=M506ST11NW6%O2=M506ST11NW6%O3=M506NNT11NW6%O4=M506ST11NW6%O5=M506ST1
OS:1NW6%O6=M506ST11)WIN(W1=F4B3%W2=F4B3%W3=F4B3%W4=F4B3%W5=F4B3%W6=F4B3)ECN
OS:(R=Y%DF=Y%T=40%W=F507%O=M506NNSNW6%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R 
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 3389/tcp)
HOP RTT      ADDRESS
1   35.16 ms 10.9.0.1
2   57.84 ms 10.10.97.10

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.88 seconds

```

We can see ports 22 (ssh) and 80 (http) are opened. Typical for an easy level CTF.
* * *
## 2.Exploring HTTP port
If we visit the web site hosted by apache on port 80 we see the following:

![image](https://user-images.githubusercontent.com/99112106/174270848-6e0aeb85-0dd6-4e42-8cfa-7a9ebd0a63b0.png)

Let's try to pull out the hidden directories with gobuster.

```
gobuster dir -w /usr/share/dirb/wordlists/big.txt -u http://10.10.97.10 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.97.10
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirb/wordlists/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/03/30 15:39:41 Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 276]
/.htaccess            (Status: 403) [Size: 276]
/css                  (Status: 301) [Size: 308] [--> http://10.10.97.10/css/]
/js                   (Status: 301) [Size: 307] [--> http://10.10.97.10/js/] 
/panel                (Status: 301) [Size: 310] [--> http://10.10.97.10/panel/]
/server-status        (Status: 403) [Size: 276]                                
/uploads              (Status: 301) [Size: 312] [--> http://10.10.97.10/uploads/]
                                                                                 
===============================================================
2022/03/30 15:41:01 Finished
===============================================================
```

We found 2 interesting directories; "panel" and "uploads". Let's visit panel directory:

![image](https://user-images.githubusercontent.com/99112106/174270896-60204b3c-a07a-4360-ba53-a5db5e27a329.png)

It looks like we have a page where we can upload files. I suppose that later we can see what we have uploaded from the "uploads" directory. Let's try to upload a php [reverse shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)!

![image](https://user-images.githubusercontent.com/99112106/174270942-86dfe7d6-84aa-4cf8-bf1f-4f5a656794f7.png)

Okay, It looks like we can't upload files with php extension. Let's try other file extensions such as php3, phtml etc.

![image](https://user-images.githubusercontent.com/99112106/174270980-78ba9e44-f37c-4bae-8055-204631eb60b1.png)

Finally, it worked with ".**phtml**" extension. We will use netcat to set the port that we indicate in the reverse shell to listening mode.

![image](https://user-images.githubusercontent.com/99112106/174271031-54cdf88b-17c7-4090-bed0-f140183595e7.png)

And now we run the reverse shell from the "uploads" directory we discovered earlier. 

![image](https://user-images.githubusercontent.com/99112106/174271072-42d57739-21b9-4456-9c04-f3f58afed41c.png)

* * *
## 3.Getting a shell
Once we execute our reverse shell, we will receive a shell on the port we were listening to:

![image](https://user-images.githubusercontent.com/99112106/174271112-f23e3113-5822-4587-b058-8c2161d11749.png)

First of all, let's stabilize de shell with python:

![image](https://user-images.githubusercontent.com/99112106/174271150-2abdba27-4f38-4ada-8fe4-d4c75138eeca.png)

Let's pull out the first of the flags, the user flag. To do this we search for it with the *find* command:

![image](https://user-images.githubusercontent.com/99112106/174271194-0816d1ce-65b6-447b-968a-0270ddfd947f.png)

* * *
## 4. Privilege Escalation
Once we have the first flag, search for binary files with enabled SUID:

![image](https://user-images.githubusercontent.com/99112106/174271240-35c950f9-be64-4927-8a08-2b7b00b18ca7.png)

We see that python has SUID privileges which will allow us to elevate privileges to root. For this we use the [GTFOBins](https://gtfobins.github.io/gtfobins/python/#suid) web page:

![image](https://user-images.githubusercontent.com/99112106/174271287-59db7eb3-f0f8-452d-b631-90849e2da8c3.png)

And finally, we are **root** and we managed to retrieved the final flag. Congratulations.
