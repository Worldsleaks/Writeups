![image](https://user-images.githubusercontent.com/99112106/174277168-499587fe-dcd9-4b94-b05b-aa4df57cce34.png)

# Mr Robot CTF

## 1.Enumeration 

Let's start with a simple nmap scan:

``` 
nmap -sC -sV -oA nmap/MrRobotCTF 10.10.29.207
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-04 12:20 CEST
Nmap scan report for 10.10.29.207
Host is up (0.038s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache
443/tcp open   ssl/http Apache httpd
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.38 seconds

```

Okay, we only have three ports; 22, 80 and 443 but ssh is actually closed...

* * *
## 2.  Exploring the website

Let's check out the website on port 80:

![image](https://user-images.githubusercontent.com/99112106/174277217-340a45cc-f691-452b-94b2-d7ac7440c7a3.png)

Weird page what so ever... It's like a CLI.

After bruteforcing hidden directories, Gobuster discovers several locations, including:

- /login (Status: 302)
- /wp-content (Status: 301)
- /admin (Status: 301)
- /wp-login (Status: 200)
- /license (Status: 200)
- /wp-includes (Status: 301)
- /robots.txt (Status: 200)

Let's see what is inside "robots.txt":

![image](https://user-images.githubusercontent.com/99112106/174277260-113f9a29-ace3-4183-8448-438b516d1961.png)

We found  a txt file called "key-1-of-3.txt". It seems like it's the first of our flags. Nice!

![image](https://user-images.githubusercontent.com/99112106/174277327-9f048095-7992-447d-91e2-ef5edcd69b24.png)

We also found a wordlist called "fsocity.dic".... Keep it!

After a while I discovered that http returned an error page with information instead of the 404 error page that always appears.

![image](https://user-images.githubusercontent.com/99112106/174277370-b060fdf2-28d6-4202-8d9f-67d892d82e49.png)

As you can see above, It's a Wordpress. We also can see a login page, let's see...

![image](https://user-images.githubusercontent.com/99112106/174277437-3274bf67-2dfc-4f30-9ba1-724df2947b64.png)

It's worth trying to bruteforce our way in with the wordlist we found before but, we need an username.... What we're going to do is open BurpSuite and send the request to the Intruder. There, we are going to try to guest the username using the wordlist we found before.

We're looking for a http response with a different lenght. Let's see.. 

![image](https://user-images.githubusercontent.com/99112106/174277489-072d2794-17e8-4e0e-8bc0-0e46c02f1ac9.png)

Okay! The username "Elliot" has a different length response and he's actually the main character of the TV show so everything seems to fit.

We are going to use Hydra to bruteforce the wordpress login for Elliot user.

```
hydra -l Elliot -P fsocity.dic 10.10.29.207 http-post-form "/wp-login/:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2Fmrrobot.thm%2Fwp-admin%2F&testcookie=1:S=302" -vV -f
```

![image](https://user-images.githubusercontent.com/99112106/174277538-76b64ff4-31ee-40cb-a0b9-54b5ebccd817.png)

After a long wait... We finally have valid credentials!! 
* * *
### Other method to retrieve credentials!!

Our gobuster discovered a directory called "license", if we visit it, we can't see nothing usefull:

![image](https://user-images.githubusercontent.com/99112106/174277568-bc8aca51-43f5-4995-996f-77b490272f60.png)

In fact, it tells us if we are script kiddies and we're going to show them we're not. Curl it:

![image](https://user-images.githubusercontent.com/99112106/174277606-7dbc2a49-edbf-4270-9ced-859521da5aae.png)

As we can see, we have a base64 encoded text. Let's decode it:

![image](https://user-images.githubusercontent.com/99112106/174277635-67fb8e42-f50e-492a-a2c0-8e6579e48fcf.png)

Yep, that was an easier and faster way to retrieve the credentials. That's why you always have to think outside the box.


* * *
## 3. Foothold

Using the credentials we bruteforced aboved, we managed to logged in and we're in the admin panel:

![image](https://user-images.githubusercontent.com/99112106/174277673-47d5e55e-95d6-41dd-9709-67d219fb4106.png)

As an administrator we can edit the appearance from the 404 error page.

![image](https://user-images.githubusercontent.com/99112106/174277721-aa72f859-311d-484d-8bac-6a3b7832db8a.png)

We're going to edit that template, adding a php reverse shell so that when we run the 404 error page, it returns a reverse shell on our attacking machine.

![image](https://user-images.githubusercontent.com/99112106/174277755-22b241ba-b74c-4353-97ba-d3e8ee170493.png)

So now, if we request the server a page that doesn't exist...

![image](https://user-images.githubusercontent.com/99112106/174277784-a909651f-1b25-4db6-8864-92891fbdcd19.png)

We recieve the reverse shell:

![image](https://user-images.githubusercontent.com/99112106/174277838-4cc7f5d1-b8b6-4ab0-9f78-9c45a6ca8d0d.png)

First of all, upgrade TTY shell:

![image](https://user-images.githubusercontent.com/99112106/174277900-be882dc2-91aa-44f4-9968-bd4f22e449aa.png)

Mmmm Okay, we can't read the second flag but we have a new username with his MD5 hashed password.

![image](https://user-images.githubusercontent.com/99112106/174277941-7c481383-0e2d-4a5b-90f7-030744f7db5f.png)

Use crackstation to crack that MD5:

![image](https://user-images.githubusercontent.com/99112106/174277997-73a5f981-8c64-41ca-897a-c6c2c8b697a9.png)

Just like that. Let's change user:

![image](https://user-images.githubusercontent.com/99112106/174278027-41a9e599-9724-4d60-b507-4bae7697e374.png)

Again, upgrade the shell and pull out the second flag:

![image](https://user-images.githubusercontent.com/99112106/174278091-73c63ca0-f0f5-4854-b598-75c1293dd72d.png)

* * *
## 4. Privileges Escalation

Let's start our way to root!

![image](https://user-images.githubusercontent.com/99112106/174278118-11e8d0dc-43a2-4dea-98b9-566f23e40e72.png)

Okay, we don't have any sudo privileges... good. Let's find out if we have some binary with enable SUID privs:

```
find / -user root -perm -4000 -print 2>/dev/null
```

![image](https://user-images.githubusercontent.com/99112106/174278173-3416b91a-05ff-4213-a91e-95a2786ee56e.png)

Nmap with SUID privileges?? That's really weird... Let's see what GTFOBins have to say about that...

![image](https://user-images.githubusercontent.com/99112106/174278224-3c40ecc2-e445-471a-8001-cc483b1051e8.png)

It seems like there are two ways to escalate privileges. Let's try the second one but without sudo command because we're not in the sudoers file.

![image](https://user-images.githubusercontent.com/99112106/174278262-5de940db-10a8-4e19-9da4-3f779786aec9.png)

Finally, we managed to escalate to root and we can pull out the last flag:

![image](https://user-images.githubusercontent.com/99112106/174278287-601befe2-8281-4206-b6d0-a6c52945edbe.png)

Congratz!
