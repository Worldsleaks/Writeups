![image](https://user-images.githubusercontent.com/99112106/178605888-040e4c00-ff48-472a-bf85-344d3598c696.png)

# Late

We start with a simple scanning ports tool such as Nmap:

```
nmap -sC -sV -oA nmap/late 10.10.11.156             
Starting Nmap 7.92 ( https://nmap.org ) at 2022-07-12 22:00 CEST
Nmap scan report for 10.10.11.156
Host is up (0.086s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 02:5e:29:0e:a3:af:4e:72:9d:a4:fe:0d:cb:5d:83:07 (RSA)
|   256 41:e1:fe:03:a5:c7:97:c4:d5:16:77:f3:41:0c:e9:fb (ECDSA)
|_  256 28:39:46:98:17:1e:46:1a:1e:a1:ab:3b:9a:57:70:48 (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Late - Best online image tools
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.34 seconds
```

As this is a simple CTF, we have the typical ports open; 80 (HTTP) and 22 (SSH).

At a glance we identify the main page "index.html":

![image](https://user-images.githubusercontent.com/99112106/178605953-c8c24477-00f6-4db0-90bb-0dddf48d5274.png)

As well as a page with a small contact form, "contact.html":

![image](https://user-images.githubusercontent.com/99112106/178605987-a308413f-bf41-4c3a-8eae-629128c9999e.png)

Reading the source code we found one interesting subdomain:

![image](https://user-images.githubusercontent.com/99112106/178606021-e724adcf-fcc8-4e21-9e39-d63ec1fac41c.png)

In order to access that subdomain we need to edit our hosts file first:

![image](https://user-images.githubusercontent.com/99112106/178606064-6ff6b87b-5896-4cb0-8385-8bb36b816430.png)

It should look like this:

![image](https://user-images.githubusercontent.com/99112106/178606099-055a4fa5-4dff-4a68-8343-9b53585927ba.png)

Now that we added the subdomain to our hosts file, we can visit it:

![image](https://user-images.githubusercontent.com/99112106/178606132-505d7699-221d-451e-95ce-84b34d81238e.png)

First thing we can see it's that the webpage uses Flask in order to convert images to text. This Flask application processes the text in the image and returns it inside an HTML paragraph. 

This vulnerability is called SSTI(Server-side Template Injection). A SSTI attack occurs when an attacker is able to use native template syntax to inject a malicious payload into a template, which is then executed server-side. We can use some payloads from Hacktricks in order to test it:

![image](https://user-images.githubusercontent.com/99112106/178606178-aedd2c77-a052-4cc8-80cd-51425327e06c.png)

https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection

I will open a Word document and write "{{7*7}}" as our payload. Once we upload the image, if the application is vulnerable it should return a results.txt with 49:

![image](https://user-images.githubusercontent.com/99112106/178606218-c5c98515-a50b-4de0-95a6-3c953922d41f.png)

Now that we know that the server is vulnerable to SSTI, let's try to read some sensitive files. In order to do that, we're going to use some payloads given by Hacktricks:

![image](https://user-images.githubusercontent.com/99112106/178606290-2e62e04e-ccd1-4335-9857-b14c7433a008.png)

Once we upload our payload, we see that it has been successful:

![image](https://user-images.githubusercontent.com/99112106/178606313-40d5dc86-814c-48b5-ba01-d8db9314aa7f.png)

We detect the user "svc_acc" in the file "/etc/passwd". If we remember, previously we detected that the port 22 (SSH) was open, therefore, maybe the user has the private key to access via SSH with his username. Let's try to modify the payload used before to achive this:

![image](https://user-images.githubusercontent.com/99112106/178606366-c7c0cb57-fcf4-47f1-8c07-5caf8c2a755f.png)

Once we upload our payload, we see that it has been successful, we have his private key!!

![image](https://user-images.githubusercontent.com/99112106/178606395-d7c33dce-87c2-4d01-918e-e0c3384ad2d2.png)

So now, all we need to do is change the permissions on this file (chmod 600 id_rsa) and then SSH into the server.

![image](https://user-images.githubusercontent.com/99112106/178606429-c9f20262-0cbb-494f-821b-e89d7243b722.png)

From here, we can retrieve the user flag:

![image](https://user-images.githubusercontent.com/99112106/178606459-47690f6b-b944-47a3-a7de-fe13abdcf644.png)

Once we have imported "linpeas.sh" from our machine, we proceed to run it to try to discover possible privilege escalation vectors:

![image](https://user-images.githubusercontent.com/99112106/178606513-f4a414b3-7b08-4428-95a7-6aeecd7007be.png)

We found out that the machine is vulnerable by PwnKit, which would be a very easy win and I don't think it was intended that way:

![image](https://user-images.githubusercontent.com/99112106/178606601-faff8f8d-57b4-4370-a832-f92d40065d61.png)

We found an interesting file "ssh-alert.sh":

![image](https://user-images.githubusercontent.com/99112106/178606647-5860a340-96ff-4515-8156-be4a495e0b5d.png)

Let's take a look at it:

![image](https://user-images.githubusercontent.com/99112106/178606683-24612e75-a05e-4799-800c-b41c543bf7a9.png)

We can see that it is a script that is in charge of sending an email every time a connection via SSH takes place. Let's check the permissions of the script:

![image](https://user-images.githubusercontent.com/99112106/178606750-374c39b2-5e5c-4865-ab31-35c8e48f2d5e.png)

Even though we cannot write to the file, we are able to append to it. Meaning that we can create another file and use it to append it to this file. Let's create a new file and add our reverse shell payload in there:

![image](https://user-images.githubusercontent.com/99112106/178606780-81aa715a-4882-4ae8-9f65-8761ba7b65a9.png)

Now append the new file into the script:

![image](https://user-images.githubusercontent.com/99112106/178606814-c447d98c-e3aa-455b-ae76-34726acb90bb.png)

Once our reverse shell has been added to the script, we set the port to listen:

![image](https://user-images.githubusercontent.com/99112106/178606850-d6fbc022-c2a6-43a4-a02a-646802e68aad.png)

Now we simply log back in via SSH to trigger the script and we're in!

![image](https://user-images.githubusercontent.com/99112106/178606882-87902459-809b-4a2a-9d13-8db48c7ff8b9.png)

Finally, we can remove the last flag and end the CTF.

![image](https://user-images.githubusercontent.com/99112106/178606903-07966dfc-7819-4465-8b9f-ac18e6953d31.png)

Congratz!!
