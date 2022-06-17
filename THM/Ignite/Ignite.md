![image](https://user-images.githubusercontent.com/99112106/174284861-da26d37a-1f0a-498d-821a-a12e8e0271e3.png)

# Ignite

## 1.Enumeration 

Let's start with a simple nmap scan:

```
nmap -sC -sV -oA nmap/Ignite 10.10.75.217 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-05 15:42 CEST
Nmap scan report for 10.10.75.217
Host is up (0.038s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/fuel/
|_http-title: Welcome to FUEL CMS
|_http-server-header: Apache/2.4.18 (Ubuntu)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.12 seconds
```

It seems like we only have 1 way to go xD

* * *
## 2.  Exploring the website

If we visit the website we can see the following:

![image](https://user-images.githubusercontent.com/99112106/174284889-5e35abad-bd1a-4c53-bfcd-6e6ba70e54d7.png)

The web page is running Fuel CMS, a content managment system (CMS). Something to keep in mind is the Fuel version, 1.4.

If we scroll down the web page we find some interesting instructions:

![image](https://user-images.githubusercontent.com/99112106/174284907-99e6f44d-ecbd-480a-a1b8-de6721d16a61.png)

We find a login page where we can use the given credentials:

![image](https://user-images.githubusercontent.com/99112106/174284932-5d8e93ba-95cd-41d8-ab2d-e3d8ebb8f57e.png)

Once we are logged in, we can see the admin panel:

![image](https://user-images.githubusercontent.com/99112106/174284983-0fe7f9a3-0762-4bcf-b624-c68b9b12cb8c.png)

* * *
## 3. Remote Code Execution

After a while, digging through the admin panel... I decided to search for some well known vulnerabilities for Fuel CMS:

![image](https://user-images.githubusercontent.com/99112106/174285005-813632ec-8b45-49d0-9093-570304add2ad.png)

Remote Code Execution (RCE) sounds pretty well... =) We have 2 python scripts and 1 ruby script. Use the one that fits better for you.

Copy the exploit you chose:

![image](https://user-images.githubusercontent.com/99112106/174285019-173f6eb1-e6e0-453c-b518-17d4c4b6f923.png)

And execute it!!

![image](https://user-images.githubusercontent.com/99112106/174285035-65ab5a27-e9f0-4901-9a90-23daefd60258.png)

Okay!! That worked fine!

* * *
## 4. Foothold

Our RCE did work!! Now we're going to execute a reverse shell back to our attacker machine:

![image](https://user-images.githubusercontent.com/99112106/174285054-5c663b29-34c0-45b2-8de7-ab49baaed4b4.png)

Our first try didn't work as expected but second did work!! We have our reverse shell working:

![image](https://user-images.githubusercontent.com/99112106/174285069-246302b3-3dfb-4e06-bde9-53ec5a27ae66.png)

As always, upgrade your TTY:

![image](https://user-images.githubusercontent.com/99112106/174285082-a6e7a488-7ffa-4f6b-aaab-568902707987.png)

Now we can pull out the first flag:

![image](https://user-images.githubusercontent.com/99112106/174285096-08360934-89c0-4dca-90b8-94c5a212b8ee.png)

* * *
## 5. Privileges Escalation

We don't have any file with enable sudo privileges sooo let's dig in the fuel configuration files. After a while we discover an interesting file called "database.php":

![image](https://user-images.githubusercontent.com/99112106/174285114-6a5f7994-8b06-44da-b561-9b0005b51083.png)

If you scroll down until the very end, you'll find root's credentials ;)

![image](https://user-images.githubusercontent.com/99112106/174285127-81697b09-1e31-4506-8bdd-ee343606ead4.png)

Change user to root!

![image](https://user-images.githubusercontent.com/99112106/174285146-ff28796d-977e-4591-8158-4a294bd9a488.png)

And that's it, we're root. Pull out final flag and we're done! Congratz =D
