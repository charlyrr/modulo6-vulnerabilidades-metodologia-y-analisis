# 🏗️ 01 - Preparación del Entorno de Laboratorio: El Taller del Analista

## 🎯 Objetivo
Antes de lanzarnos a explotar el **CVE-2025-5548**, necesitamos construir nuestro "taller". En esta primera fase, hemos preparado un entorno de laboratorio seguro y equipado con todas las herramientas necesarias para analizar aplicaciones vulnerables en Windows.

Entender **por qué** usamos cada herramienta es la diferencia entre ejecutar un script a ciegas y ser un verdadero analista de vulnerabilidades.

---


## 🛡️ 1. La Base: Máquina Virtual (Aislamiento)

El laboratorio se ha construido sobre:
* **Hipervisor:** VirtualBox
* **Sistema Operativo:** Windows 11

**¿Por qué una VM?** Analizar malware o probar exploits (como el que desarrollaremos para el CVE-2025-5548) puede corromper el sistema operativo o dejar puertas traseras abiertas. Trabajar en una máquina virtual nos proporciona un entorno aislado (Sandbox).

> 💡 **Tip de Analista:** Tal como vimos en las sesiones técnicas, la regla de oro aquí es el uso de **Snapshots (Instantáneas)**. Antes de ejecutar un exploit que pueda crashear el sistema, tomamos una snapshot para poder revertir los cambios en segundos.

<img alt="Máquina Virtual Windows 11" src="https://github.com/charlyrr/modulo6-vulnerabilidades-metodologia-y-analisis/blob/main/IMG/Win11_VM.png"/>

---

## ⚙️ 2. Dependencias del Sistema

Para que nuestro arsenal funcione, necesitamos instalar los motores de ejecución base:

### 🐍 Python
Python es el lenguaje por excelencia para el desarrollo de exploits. Lo usaremos para escribir los scripts que enviarán los *payloads* maliciosos al servidor vulnerable (usando librerías como `socket` o `pwntools`).
* *Nota:* Al instalar Immunity Debugger, se instala automáticamente la versión 2.7, muy común en exploits antiguos, aunque hoy en día es recomendable tener también la versión 3.x.

<img alt="Python" src="https://github.com/charlyrr/modulo6-vulnerabilidades-metodologia-y-analisis/blob/main/IMG/python.png"/>

### ☕ Java
**¿Por qué Java?** Herramientas avanzadas de ingeniería inversa creadas por la NSA, como **Ghidra**, están desarrolladas en Java y requieren el JDK/JRE para poder ejecutarse.

<img alt="java" src='https://github.com/charlyrr/modulo6-vulnerabilidades-metodologia-y-analisis/blob/main/IMG/java.png'/>
---

## 📝 3. IDEs y Editores de Código

Para escribir nuestros exploits y tomar notas durante el análisis, necesitamos buenos editores:

* **Notepad++:** El bloc de notas con esteroides. Perfecto para revisar logs rápidos o hacer modificaciones ligeras en un script sin consumir recursos.
<img alt="notepad" src='https://github.com/charlyrr/modulo6-vulnerabilidades-metodologia-y-analisis/blob/main/IMG/notepad.png'>


* **Visual Studio Code:** Mi editor principal para estructurar el código del exploit y tener una terminal integrada.
<img alt="visual" src='https://github.com/charlyrr/modulo6-vulnerabilidades-metodologia-y-analisis/blob/main/IMG/visaul.png'>

* **PyCharm:** IDE profesional para cuando el exploit en Python requiere de un desarrollo más complejo y un buen autocompletado.
<img alt="PyCharm" src='https://github.com/charlyrr/modulo6-vulnerabilidades-metodologia-y-analisis/blob/main/IMG/pycharm.png'>
---

## 🔬 4. El Arsenal Principal: Reversing y Debugging

Aquí está el corazón de nuestro laboratorio. Estas son las herramientas que nos permitirán "ver las tripas" del CVE-2025-5548.

### Análisis Estático (Reversing)
Analizamos el código sin ejecutarlo para entender su estructura y encontrar posibles fallos en las funciones de memoria.

* **Ghidra:** Nos permite descompilar el binario (pasar de código máquina a un pseudo-código en C) para leer la lógica del programa.
<img alt="ghidra" src='https://github.com/charlyrr/modulo6-vulnerabilidades-metodologia-y-analisis/blob/main/IMG/ghidra.png'>

**IDA Free:** El estándar de la industria para desensamblar. Nos muestra el código en ensamblador puro para un análisis quirúrgico.
  
  > 🕵️‍♂️ **Tip de Privacidad:** Para descargar IDA Free, el fabricante (Hex-Rays) te exige introducir un correo electrónico al que te envían el enlace y la licencia gratuita. Como buena práctica de privacidad (y para mantener mi bandeja de entrada limpia de *spam*), he utilizado un correo temporal de 10 minutos a través de [10minemail.com](https://10minemail.com/es/). ¡Muy recomendable para registrarse en este tipo de herramientas!
> <img alt="ida" src='https://github.com/charlyrr/modulo6-vulnerabilidades-metodologia-y-analisis/blob/main/IMG/idafree.png'>

### Análisis Dinámico (Debugging)
Ejecutamos el programa paso a paso para ver cómo interactúa con la memoria de la CPU (Registros, Pila/Stack, etc.).

* **Immunity Debugger:** Fundamental para la explotación de *Buffer Overflows*. Nos permite adjuntarnos (attach) al proceso vulnerable, enviarle nuestro código y ver exactamente en qué momento "crashea" y si logramos sobrescribir el registro EIP.

<img alt="immunity" src='https://github.com/charlyrr/modulo6-vulnerabilidades-metodologia-y-analisis/blob/main/IMG/nmmunity.png'>

* **Mona.py (El ayudante estrella):** Mona es un script creado por Corelan Team que se integra en Immunity Debugger (en la carpeta `PyCommands`). 
  * **¿Por qué lo necesitamos?** Nos automatiza tareas tediosas como crear patrones cíclicos (para medir el tamaño del *buffer*), encontrar *Badchars* (caracteres malos que rompen el exploit) o buscar saltos a nuestro código (instrucciones `JMP ESP`).
  <img alt="immunity" src='https://github.com/charlyrr/modulo6-vulnerabilidades-metodologia-y-analisis/blob/main/IMG/mona.png'>


---

## 🌐 5. Utilidades de Red y Control de Versiones

* **Git:** Para clonar repositorios del curso, descargar exploits públicos de GitHub o gestionar las versiones de nuestro propio código.

<img alt="git" src='https://github.com/charlyrr/modulo6-vulnerabilidades-metodologia-y-analisis/blob/main/IMG/git.png'>
  
* **Nmap / Ncat:** Ncat (la navaja suiza de red) nos servirá para conectarnos manualmente al puerto de la aplicación vulnerable, entender cómo responde e interactuar con ella antes de automatizar el ataque con Python.
<img alt="nmap" src='https://github.com/charlyrr/modulo6-vulnerabilidades-metodologia-y-analisis/blob/main/IMG/nmap.png'>

---

## 🎯 6. Target de Pruebas: Aplicaciones Vulnerables

Antes de enfrentarnos al objetivo final (CVE-2025-5548), entrenaremos con **Vulnserver**. 
Vulnserver es un servidor TCP escrito intencionadamente con fallos de memoria. Es el "saco de boxeo" perfecto para dominar Immunity Debugger y Mona antes de pasar a un entorno real.

<img alt="vulnserver" src='https://github.com/charlyrr/modulo6-vulnerabilidades-metodologia-y-analisis/blob/main/IMG/vulnserver.png'>
---

## 🏁 Conclusión de la Fase 1

La preparación manual de este entorno de laboratorio no es un capricho. Instalar las herramientas paso a paso y entender la función de cada una (Python para el arma, Immunity para apuntar y Mona para calcular la distancia) nos proporciona la base sólida que todo analista de seguridad necesita antes de enfrentarse a un reto de *Exploit Development*. 

¡Entorno listo! Pasamos a la fase de análisis.
