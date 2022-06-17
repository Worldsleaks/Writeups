![image](https://user-images.githubusercontent.com/99112106/174287331-09b2cd5a-0170-49e3-933e-cc9535923d4b.png)

# Boiler CTF

## 1.Enumeration 

Let's start with a simple nmap scan:

```
nmap -sC -sV -oA nmap/BoilerCTF 10.10.209.83
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-05 10:12 CEST
Nmap scan report for 10.10.209.83
Host is up (0.078s latency).
Not shown: 997 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
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
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
80/tcp    open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.18 (Ubuntu)
10000/tcp open  http    MiniServ 1.930 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.46 seconds
```

We have opened ports like port 21 (ftp), port 80 (http) and port 10000 (http). Let's repeat our scan but we're going to spead up things a little bit and this time we're going through all the ports:

```
nmap -p- --min-rate=10000 10.10.209.83 -v
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-05 10:16 CEST
Initiating Ping Scan at 10:16
Scanning 10.10.209.83 [4 ports]
Completed Ping Scan at 10:16, 0.21s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 10:16
Completed Parallel DNS resolution of 1 host. at 10:16, 0.00s elapsed
Initiating SYN Stealth Scan at 10:16
Scanning 10.10.209.83 [65535 ports]
Discovered open port 80/tcp on 10.10.209.83
Discovered open port 21/tcp on 10.10.209.83
Discovered open port 55007/tcp on 10.10.209.83
Discovered open port 10000/tcp on 10.10.209.83
Increasing send delay for 10.10.209.83 from 0 to 5 due to max_successful_tryno increase to 4
Completed SYN Stealth Scan at 10:16, 7.96s elapsed (65535 total ports)
Nmap scan report for 10.10.209.83
Host is up (0.056s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE
21/tcp    open  ftp
80/tcp    open  http
10000/tcp open  snet-sensor-mgmt
55007/tcp open  unknown

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 8.30 seconds
           Raw packets sent: 79005 (3.476MB) | Rcvd: 67803 (2.712MB)

```

Okay, It seems like we missed the port 55007. Let's run another scan on that port to see what's running on:

```
nmap -p 55007 -sC -sV 10.10.209.83       
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-05 10:18 CEST
Nmap scan report for 10.10.209.83
Host is up (0.036s latency).

PORT      STATE SERVICE VERSION
55007/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e3:ab:e1:39:2d:95:eb:13:55:16:d6:ce:8d:f9:11:e5 (RSA)
|   256 ae:de:f2:bb:b7:8a:00:70:20:74:56:76:25:c0:df:38 (ECDSA)
|_  256 25:25:83:f2:a7:75:8a:a0:46:b2:12:70:04:68:5c:cb (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 3.23 seconds
```

So we have SSH running on port 55007, that's cool!

* * *
## 2.  Exploring FTP
As you can see from the above scan, we have enable anonymous login, so let's have a look on that:

![image](https://user-images.githubusercontent.com/99112106/174287357-954368ea-d814-49ab-8fdd-61b989c096f3.png)

We managed to logged in and to retrieved a hidden text file, let's have a look at it:

![image](https://user-images.githubusercontent.com/99112106/174287379-821bc11c-98e7-46a5-9ccd-f89c0db5b6ee.png)

That is weird =D It seems like it's encoded with caeser chiper, let's find out:

![image](https://user-images.githubusercontent.com/99112106/174287394-5fcd7cce-47e0-4d39-bb1e-17a2189b77c8.png)

Yes! We decoded the message but it looks like a rabbit hole T.T


* * *
## 3.  Exploring the website

If we have a look we can see the default apache web page:

![image](https://user-images.githubusercontent.com/99112106/174287407-d5e67410-10ba-48f9-9201-e34b1ad2367d.png)

Let's run Gobuster:

```
gobuster dir -w /usr/share/dirb/wordlists/big.txt -u http://10.10.209.83 -q                
/.htaccess            (Status: 403) [Size: 296]
/.htpasswd            (Status: 403) [Size: 296]
/joomla               (Status: 301) [Size: 313] [--> http://10.10.209.83/joomla/]
/manual               (Status: 301) [Size: 313] [--> http://10.10.209.83/manual/]
/robots.txt           (Status: 200) [Size: 257]                                  
/server-status        (Status: 403) [Size: 300] 
```

Let's check "robots.txt":

![image](https://user-images.githubusercontent.com/99112106/174287425-55677124-a659-4bd3-b79d-58d426cbaeee.png)

Jumm... We have a bunch of disallowed directories and a ASCII text.. Let's decoded it.

![image](https://user-images.githubusercontent.com/99112106/174287438-b58a07f6-f269-4a10-8aac-e926df46c9f3.png)

Okay, and now let's decoded from Base64:

![image](https://user-images.githubusercontent.com/99112106/174287456-14d7e8e9-cd40-4a3c-880b-e2ab05da84b8.png)

And finally, looks like a MD5 hash. Hope Crackstation can help us with it:

![image](https://user-images.githubusercontent.com/99112106/174287539-9b5a87c7-de53-47d7-87c2-ca94342ac971.png)

We are trolled...Damn it...

After checking "robots.txt" and "manual" directory, we found an interesting directory called "joomla":

![image](https://user-images.githubusercontent.com/99112106/174287559-44be1799-7002-46d8-85a0-3544f6f273b9.png)

It looks like some kind of CMS that allows the admin to create a blog. We're going to list again some possible hidden directories with Gobuster:

```
gobuster dir -w /usr/share/dirb/wordlists/big.txt -u http://10.10.209.83/joomla/ -q
/.htpasswd            (Status: 403) [Size: 303]
/.htaccess            (Status: 403) [Size: 303]
/_archive             (Status: 301) [Size: 322] [--> http://10.10.209.83/joomla/_archive/]
/_database            (Status: 301) [Size: 323] [--> http://10.10.209.83/joomla/_database/]
/_files               (Status: 301) [Size: 320] [--> http://10.10.209.83/joomla/_files/]   
/_test                (Status: 301) [Size: 319] [--> http://10.10.209.83/joomla/_test/]    
/administrator        (Status: 301) [Size: 327] [--> http://10.10.209.83/joomla/administrator/]
/bin                  (Status: 301) [Size: 317] [--> http://10.10.209.83/joomla/bin/]          
/build                (Status: 301) [Size: 319] [--> http://10.10.209.83/joomla/build/]        
/cache                (Status: 301) [Size: 319] [--> http://10.10.209.83/joomla/cache/]        
/cli                  (Status: 301) [Size: 317] [--> http://10.10.209.83/joomla/cli/]          
/components           (Status: 301) [Size: 324] [--> http://10.10.209.83/joomla/components/]   
/images               (Status: 301) [Size: 320] [--> http://10.10.209.83/joomla/images/]       
/includes             (Status: 301) [Size: 322] [--> http://10.10.209.83/joomla/includes/]     
/installation         (Status: 301) [Size: 326] [--> http://10.10.209.83/joomla/installation/] 
/language             (Status: 301) [Size: 322] [--> http://10.10.209.83/joomla/language/]     
/layouts              (Status: 301) [Size: 321] [--> http://10.10.209.83/joomla/layouts/]      
/libraries            (Status: 301) [Size: 323] [--> http://10.10.209.83/joomla/libraries/]    
/media                (Status: 301) [Size: 319] [--> http://10.10.209.83/joomla/media/]        
/modules              (Status: 301) [Size: 321] [--> http://10.10.209.83/joomla/modules/]      
/plugins              (Status: 301) [Size: 321] [--> http://10.10.209.83/joomla/plugins/]      
/templates            (Status: 301) [Size: 323] [--> http://10.10.209.83/joomla/templates/]    
/tests                (Status: 301) [Size: 319] [--> http://10.10.209.83/joomla/tests/]        
/tmp                  (Status: 301) [Size: 317] [--> http://10.10.209.83/joomla/tmp/]          
/~www                 (Status: 301) [Size: 318] [--> http://10.10.209.83/joomla/~www/]
```

Okayyy ~~ We have some directories to dig in. 

It looks like there are more rabbit holes along the way...

![image](https://user-images.githubusercontent.com/99112106/174287578-e2c9ce51-4a90-4cbd-9a09-96cb02bfd6b6.png)

After a while digging throug directories, I managed to discover that "test" directory was an interesting one:

![image](https://user-images.githubusercontent.com/99112106/174287596-99503642-3ea0-400b-ae40-4cf3c8bc39fb.png)

A few researches after, we discovered a critical vulnerability on ExploitDB for sar2html:

![image](https://user-images.githubusercontent.com/99112106/174287603-3554f9ff-fd59-4c29-acab-08ef4a247e52.png)

So the exploit is basically a RCE (Remote Code Execution). Following the given steps, we procede:

![image](https://user-images.githubusercontent.com/99112106/174287627-a382002e-403c-4383-a000-3b2e6df73296.png)

We can see a "log.txt" in the "select host" sidebar. Use cat to see the content:

![image](https://user-images.githubusercontent.com/99112106/174287649-3f2f82f9-eb90-46f8-bd92-0d026c43b789.png)

Yes!! We managed to read a ssh log file wich contains credentials! (Basterd:superduperp@$$)
* * *
## 4. Foothold

Let's try to login via SSH with those credentials:

![image](https://user-images.githubusercontent.com/99112106/174287669-81084419-dc9c-417c-bb0e-ac4cfd947ce9.png)

We're in! First of alll, let's upgrade our shell:

![image](https://user-images.githubusercontent.com/99112106/174287681-1bf2543d-1bdf-4128-adac-ea6a8130d5d7.png)

Let's have a look to home directories:

![image](https://user-images.githubusercontent.com/99112106/174287707-7c77a882-e484-4795-aa24-f8e3f78cf017.png)

Okay, it seems like we can't access user flag and the only thing we can do now is try to vuln "backup.sh" from basterd's home directory. Let's have a look on that:

![image](https://user-images.githubusercontent.com/99112106/174287723-014efac6-a117-451f-bf0c-3d8d14670ff9.png)

In the script we can find a password for "stoner"!! Let's SSH with that user:

![image](https://user-images.githubusercontent.com/99112106/174287740-a698b342-6ca1-40ea-86cd-fe9cd0517c9a.png)

Nice!! Inside Stoner home directory we found a congratulations txt file (first flag):

![image](https://user-images.githubusercontent.com/99112106/174287751-60f2d786-64a1-4aef-9a27-9784570f6afe.png)

* * *
## 5. Privileges Escalation
Now, if we check the sudo privileges for stoner user, we are trolled again...

![image](https://user-images.githubusercontent.com/99112106/174287762-963917f6-5515-428c-bee1-70cb5575045c.png)

Let's find for files with enable SUID (Set owner User ID up on execution) bits:

![image](https://user-images.githubusercontent.com/99112106/174287776-e9080a49-2d95-4a45-bb3e-22c1b5e63ae6.png)

We managed to discover that "find" have enable SUID bits, so let's checkout GTFOBins...

![image](https://user-images.githubusercontent.com/99112106/174287793-1db927f0-c79f-4421-8c01-2cf6c8530782.png)

Let's try that:

![image](https://user-images.githubusercontent.com/99112106/174287805-6414ab11-98f6-44c0-80e6-5500fa277367.png)

It worked! And we can retrieve the root flag. Congratz =D
