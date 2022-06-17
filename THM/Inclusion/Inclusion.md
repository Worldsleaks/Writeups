![image](https://user-images.githubusercontent.com/99112106/174283259-d55e11d2-a223-440e-98f7-a588065102f3.png)

# Inclusion

## 1.Enumeration 

Let's start with a simple nmap scan:

```
nmap -sC -sV -oA nmap/inclusion 10.10.48.18 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-04 11:39 CEST
Nmap scan report for 10.10.48.18
Host is up (0.058s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e6:3a:2e:37:2b:35:fb:47:ca:90:30:d2:14:1c:6c:50 (RSA)
|   256 73:1d:17:93:80:31:4f:8a:d5:71:cb:ba:70:63:38:04 (ECDSA)
|_  256 d3:52:31:e8:78:1b:a6:84:db:9b:23:86:f0:1f:31:2a (ED25519)
80/tcp open  http    Werkzeug httpd 0.16.0 (Python 3.6.9)
|_http-title: My blog
|_http-server-header: Werkzeug/0.16.0 Python/3.6.9
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.71 seconds

```

Okay, we only have two open ports; SSH and HTTP. 

* * *
## 2.  Exploring the website

If we have a look we can see the following page:

![image](https://user-images.githubusercontent.com/99112106/174283287-51582b6b-e01e-4ed8-9557-502e49c4d784.png)

Okay, it seems that we have 3 URL's. In one of them we can read something about LFI and it gives us an simple example:

![image](https://user-images.githubusercontent.com/99112106/174283302-11d287b8-0ef8-4317-8139-7e177d263bf3.png)

This give us an idea about we can do in our situation. Let's try to do that simple LFI trick as we saw above:

![image](https://user-images.githubusercontent.com/99112106/174283318-6d22e520-c4e2-4147-bfe4-3c090c744c92.png)

It worked! We managed to see the "/etc/passwd" file where we can enumerate some users such as "falconfeast". Right after listing the user "falconfeast", we can see a commented line with what looks like credentials.

Now that we have an username, we can pull out the user flag from his home path!

```
http:://<IP address>/article?name=../../../../../../home/falconfeast/user.txt
```

![image](https://user-images.githubusercontent.com/99112106/174283331-bcd896a7-5647-4130-a24b-8d1922abd67c.png)

* * *
## 3. Foothold

Using the credentials we saw above in the "/etc/passwd" file, we're trying to login via SSH:

![image](https://user-images.githubusercontent.com/99112106/174283356-b319b3cc-e3c0-4878-8e6e-7f0fe463b2fc.png)

We're in!

* * *
## 4. Privilege Escalation

First of all, we're going to check what files we can run with sudo privileges.

![image](https://user-images.githubusercontent.com/99112106/174283377-f158238f-fea2-4295-9c09-02b8bdc0bb40.png)

Damn... We can run socat with sudo privileges... This is done!!!

What do you have to say for us, GTFObins?...

![image](https://user-images.githubusercontent.com/99112106/174283400-834fce08-8a85-47ad-bc05-b1161ef9e05c.png)

Okay, you the boss~~

![image](https://user-images.githubusercontent.com/99112106/174283425-8168c78a-5294-4385-b95b-39dc9cd02408.png)

Yep. That was a really simple PrivEsc. Retrieve root flag and we're done. Congratz.
