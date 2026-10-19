# Writeup: FirstHacking - DockerLabs

- **Plataforma:** DockerLabs
- **Máquina:** FirstHacking
- **Dificultad:** Muy Fácil
- **Categoría:** Pentesting / Web & Network Hacking
- **Fecha:** Octubre 2026

---

## 🎯 Objetivo
Identificar vulnerabilidades en el entorno de pruebas, obtener acceso inicial al sistema y lograr comprometer el objetivo siguiendo metodologías éticas de prueba de penetración.

---

## 🛠️ Herramientas Utilizadas
* **Nmap:** Escaneo de puertos y descubrimiento de servicios[cite: 2].
* **Gobuster / Dirbuster:** Escaneo y fuzzing de directorios web.
* **Netcat / Shells:** Establecimiento de conexiones e interacción[cite: 2].

---

## 📌 Metodología y Ejecución

### 1. Reconocimiento y Escaneo 📡
Se inicia el descubrimiento de puertos activos en la dirección IP objetivo mediante `nmap`:

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn <IP-OBJETIVO> -oN targeted
