# Máquina Amor

Dificultad -> Easy

Enlace a la máquina -> [Dockerlabs](https://dockerlabs.es/)

Creador -> [romabri](https://github.com/romabri/WriteUps/commits?author=romabri)

---

## Reconocimiento



Comenzamos realizando un escaneo general con **nmap** sobre la IP de la máquina víctima para ver que puertos tiene abiertos.

![[Pasted image 20241124190801.png]]
Ya con esto podemos observar que el ttl=64 corresponde a una maquina Linux 

Lanzamos un conjunto de scripts predeterminado con **nmap** para que nos reporte más información relacionada a los puertos descubiertos en el anterior escaneo.
![[Pasted image 20241124190955.png]]
También pueden utilizar el -vvv para ver en tiempo real la información durante el escaneo ,
Podemos observar que tiene el puerto 22 abierto es decir podemos acceder con credenciales SSH además del puerto 80, por lo tanto eso nos hace sospechar que debe tener una pagina levantada. 

Accedemos a la web para inspeccionarla:
![[Pasted image 20241124192843.png]]

Encontramos información importante como un par de usuarios, **carlota** y **juan**. Y que hay usuarios en el sistema con contraseñas débiles.

---

## Explotación


Aplicamos fuerza bruta con **hydra** sobre los usuarios:
![[Pasted image 20241124192945.png]]

Encontramos las credenciales de carlota -> **carlota:babygirl** Accedemos por ssh:
![[Pasted image 20241124193031.png]]
Estamos dentro !

---

## Escalada de privilegios


En el directorio **/home/Desktop/fotos/vacaciones** de Carlota encontramos una imagen.

Nos la descargaremos para verla. Podemos usar **scp**:
![[Pasted image 20241124193205.png]]
Veremos si tiene algo oculto con la herramienta **steghide**:
![[Pasted image 20241124193221.png]]
Obtenemos un **secret.txt** con la siguiente información -> **ZXNsYWNhc2FkZXBpbnlwb24=**

utilizamos  **base64** así que lo decoficamos:

![[Pasted image 20241124193317.png]]
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

Podemos ejecutar **ruby** como el usuario **root**, si no sabemos como abusar de esto, siempre podemos recurrir a [GTFOBins](https://github.com/albertomarcostic/DockerLabs-WriteUps/blob/main)

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