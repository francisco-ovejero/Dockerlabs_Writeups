#  Diario de aprendizaje | Ciberseguridad y Pentesting

Este documento sirve como registro continuo de mi progreso, conceptos clave aprendidos e informes de laboratorios resueltos a lo largo de mi formación en hacking ético y seguridad ofensiva.

---

##  Resumen Global
* **Laboratorios resueltos:** 13
* **Plataforma principal:** DockerLabs

---

##  Bitácora de aprendizaje


###  19/Oct/2026
* **Laboratorio:** FirstHacking
* **Dificultad:** Very Easy
* **Técnicas y conceptos clave:**
  * **Nmap:** escaneo de puertos y detección de versiones de servicios usando el parámetro `-sCV`.
  * **Netcat:** conexiones remota al especificar la dirección IP y el puerto.
  * **Exploit de la version `vsftpd 2.3.4` FTP (CVE-2011-2523).**
* **💡 Lección y aprendizaje clave:**
    > Esta práctica me permitió familiarizarme con el proceso de reconocimiento inicial, el análisis y reconocimiento de vulnerabilidades y comprender los peligros de mantener software o servicios desactualizados.
* **Writeup:** [/VeryEasy/firsthacking.md](./VeryEasy/firsthacking.md)

---

###  21/Oct/2026 -  22/Oct/2026
* **Laboratorio:** BreakMySSH y Vacaciones
* **Dificultad:** Very Easy
* **Técnicas y conceptos clave:**
  * **Ataques de fuerza bruta:** obtención de credenciales utilizando `hydra` y diccionarios.
  * **Shells y SSH:** establecimiento e interacción segura con consolas remotas.
  * **Escalada de privilegios:** Identificación de vectores de elevación de permisos en entornos Linux.
* **💡 Lección y aprendizaje clave:**
  > Comprendí el riesgo de utilizar contraseñas débiles, dejar información sensible en comentarios de código y la importancia de aplicar el principio de menor privilegio.
  > A partir de este momento, comencé a sugerir acciones de mitigación para las vulnerabilidades que encuentro en los laboratorios.
* **Writeups:** [/VeryEasy/breakmySSH.md](./VeryEasy/breakmySSH.md) | [/VeryEasy/vacaciones.md](./VeryEasy/vacaciones.md)

---

###  23/Oct/2026 - 24/Oct/2026
* **Laboratorios:** Trust y BorazuwarahCTF
* **Dificultad:** Very Easy
* **Técnicas y conceptos clave:**
  * **Enumeración web con `gobuster`:** se utiliza para encontrar directorios, DNS ocultos mediante diccionarios. Puedo usar el parámetro `-x html,phptxt` para especificar el tipo de archivo que quiero encontrar.
  * **Esteganografía:** ocultar metadatos en un archivo (imágen,video,audio,documentos). Aprendí a leer dichos metadatos utilizando la herramienta `exiftool`.
* **💡 Lección y aprendizaje clave:**
  > Comprendí que pueden extraerse archivos ocultos en una web mediante ataques de fuerza bruta y que los archivos subidos a la web contienen metadatos que pueden divulgar información sensible. 
* **Writeups:** [/VeryEasy/trust.md](./VeryEasy/trust.md) | [/VeryEasy/borazuwarahCTF.md](./VeryEasy/borazuwarahCTF.md)

---

###  26/Oct/2026
* **Laboratorios:** Obsession
* **Dificultad:** Very Easy
* **Técnicas y conceptos clave:**
  * **Comando `ping`:** debería ser el primer comando utilizado ya que sirve para saber si el servicio web se encuentra activo mediante el envío de paquetes.
  * **Parámetro `-oN` en Nmap:** sirve para almacenar los datos devueltos por **Nmap** en un archivo `.txt`, esto permite acceder a la información sin necesidad de volver a realizar el scan. Para acceder al archivo de texto simplemente se ejecuta: `cat nombre_archivo.txt`.
* **💡 Lección y aprendizaje clave:**
  > Utilizar parámetros apropiados ayuda a que la resolución de laboratorios sea más fluida. A partir de este momento, voy a implementar `ping` y `-oN` en cada una de mis prácticas.
* **Writeup:** [/VeryEasy/obsession.md](./VeryEasy/obsession.md)

---

###  28/Oct/2026
* **Laboratorio:** Hedgehog
* **Dificultad:** Very Easy
* **Técnicas y conceptos clave:**
  * **Comando `curl`:** puedo usarlo para acceder a los datos de una página web sin necesidad de utilizar el navegador. Con el parámetro `-I` puedo consultar los headers de la página.
  * **Manipular diccionarios con `tac`.**
* **💡 Lección y aprendizaje clave:**
  > Es importante encontrar nuevas formas de utilizar un mismo diccionario para comparar credenciales. Es posible tanto alterar su orden como eliminar/agregar criterios al utilizar `sed`.
* **Writeup:** [/VeryEasy/hedgehog.md](./VeryEasy/hedgehog.md)

--- 

###  30/Oct/2026
* **Laboratorio:** Acme
* **Dificultad:** Very Easy
* **Técnicas y conceptos clave:**
  * **Comando `find`:** se utiliza para buscar archivos/directorios dentro del sistema basandose en el criterio especificado. Puede buscar por nombre, filtrar caracteristicas e incluso realizar acciones como ejecutar o eliminar.
  * **SUID:** permiso asignable a archivo ejecutable. Al ejecutar archivo con permiso SUID, el proceso se ejecuta con los privilegios del dueño del archivo, que suele ser root.
* **💡 Lección y aprendizaje clave:**
  > SUID puede resultar peligroso cuando no se lo utiliza como corresponde. Si se encuentra en `/bin/bash`, cualquier usuario puede ejecutarlo y obtener privilegios root. Es de vital importancia para la seguridad conceder solo los permisos indispensables, tanto a los usuarios como a los archivos del sistema.
  > Al ejecutar archivos con SUID, se usa el parametro **-p** para mantener los privilegios del autor.

  > `find` es un comando extremadamente útil que no solo sirve para encontrar archivos/directorios, sino que también puede realizar acciones como ejecutar `-exec`. Gracias a los criterios de búsqueda utilizados,`find` se vuelve una herramienta muy útil y flexible.
* **Writeup:** [/VeryEasy/acme.md](./VeryEasy/acme.md)

--- 

###  2/Nov/2026
* **Laboratorios:** -Pn
* **Dificultad:** Easy
* **Técnicas y conceptos clave:**
  * **Reverse shell:** técnica donde la víctima se conecta de forma activa con el atacante mediante la ejecución de payloads o malwares.
  * **msfvenom:** es una de las herramientas de explotación de Metasploit. Sirve para generar payload malicioso empaquetado en un archivo. En el caso de este laboratorio, se utilizó para un archivo `.war` (Web Application Archive) que contiene los componentes necesarios para ejecutar una aplicación web Java.
* **💡 Lección y aprendizaje clave:**
  > Como las reglas de salida del sistema suelen ser más flexibles que las de entrada, utilizar reverse shell permite evadir las restricciones haciendo que sea la máquina vícitma quien inicie la conexión.

  > RCE (Remote Code Execution). Permitir que usuarios no autenticados o no autorizados puedan desplegar archivos `.war` representa una falla critica de seguridad. Si un atacante despliega un `.war` malicioso, el server lo descomprime y ejecuta, otorgando una consola remota interactiva.
  > Para evitar que esto ocurra, se debe utilizar autenticación fuerte de usuarios, principio de menor privilegio, hardening de red y control y monitoreo de los archivos desplegados.
  
  > Diferencias entre malware, payload y script malicioso. 
* **Writeup:** [/easy/pn.md](./easy/pn.md)

---

###  5/Nov/2026
* **Laboratorios:** WalkingCMS
* **Dificultad:** Easy
* **Técnicas y conceptos clave:**
  * **wpscan:** scaner de sitios web Wordpress (plugins, usuarios, credenciales).
  * **Generación de reverse shells utilizando plataformas como [Reverse Shell Generator](https://www.revshells.com/).**
* **💡 Lección y aprendizaje clave:**
  > Archivo **xmlrcp.php** de Wordpress: permite conexión remota con el sitio web, automatizar tareas y validar credenciales de forma masiva y rápida, por lo que es vulnerable a ataques de fuerza bruta con `wpscan`.
* **Writeup:** [/easy/walkingCMS.md](./easy/walkingCMS.md)

---

###  9/Nov/2026
* **Laboratorios:** Amor
* **Dificultad:** Easy
* **Técnicas y conceptos clave:**
  * **Transferencia de archivos de shell remota a máquina atacante.**
  * **steghide.**
* **💡 Lección y aprendizaje clave:**
  > Es posible transferir archivos de shell remota utilizando `python3 -m http.server 8000`, lo que abre un servidor web en el directorio actual. Teniendo el servidor activo y escuchando en el puerto 8000, se usa `wget` en una terminal local para completar la transferencia especificando la dirección IP víctima, el puerto y el nombre del archivo.
  
  > `steghide` es una herramienta de esteganografía que permite ocultar archivos completos dentro de un archivo portador modificando sus bits. Puede utilizarse para guardar (embed) o extraer (extract) archivos.
  
  > Diferencias de uso entre `steghide` y `exiftool`. 
* **Writeup:** [/easy/amor.md](./easy/amor.md)

---

