#  Diario de aprendizaje | Ciberseguridad y Pentesting

Este documento sirve como registro continuo de mi progreso, conceptos clave aprendidos e informes de laboratorios resueltos a lo largo de mi formación en hacking ético y seguridad ofensiva.

---

##  Resumen Global
* **Laboratorios resueltos:** 8
* **Plataforma orincipal:** DockerLabs

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
  * **Comando `curl`:** puedo usarlo para acceder a los datos de una página web sin necesidad de utilizar el navegador. Con el parámetro `-I` puedo consultar los headers de la página
  * **Manipular diccionarios con `tac`.**
* **💡 Lección y aprendizaje clave:**
  > Es importante encontrar nuevas formas de utilizar un mismo diccionario para comparar credenciales. Es posible tanto alterar su orden como eliminar/agregar criterios al utilizar `sed`.
* **Writeup:** [/VeryEasy/hedgehog.md](./VeryEasy/hedgehog.md)

--- 
