# 01 - Preparación del entorno de laboratorio

## Objetivo

En esta primera parte de la práctica se ha preparado un entorno de laboratorio para poder analizar aplicaciones vulnerables en Windows.

El entorno se ha construido dentro de una máquina virtual utilizando VirtualBox y Windows 11. Trabajar dentro de una VM permite realizar pruebas de seguridad de forma aislada sin afectar al sistema principal.

---

# Máquina virtual utilizada

El laboratorio se ha creado utilizando:

- VirtualBox
- Windows 11

La máquina virtual se utiliza como entorno de pruebas donde se instalan todas las herramientas necesarias para el análisis de vulnerabilidades.

<img alt="image" src="https://github.com/charlyrr/modulo6-vulnerabilidades-metodologia-y-analisis/blob/main/IMG/Win11_VM.png"/>

---

# Instalación de herramientas

Siguiendo lo explicado en clase, las herramientas se han instalado manualmente descargándolas desde las páginas oficiales de los fabricantes.

Esto permite comprender mejor el papel de cada herramienta dentro del proceso de análisis.

<img alt="image" src="https://github.com/charlyrr/modulo6-vulnerabilidades-metodologia-y-analisis/blob/main/IMG/herramientas.png"/>

---

# Lenguajes instalados

## Python

Python se utiliza para ejecutar scripts y herramientas utilizadas durante el análisis de vulnerabilidades.
Para conseguir python puedes acceder a su pagina oficial https://www.python.org/downloads/ o en la instalacion de Immunity Debugger se instalara la vcersion 2.7 de forma automatica.


<img alt="image" src="https://github.com/charlyrr/modulo6-vulnerabilidades-metodologia-y-analisis/blob/main/IMG/python.png"/>

---

## Java

Java es necesario para ejecutar herramientas de reversing como Ghidra.
Para descargar java desde su pagina oficial https://www.oracle.com/es/java/technologies/downloads/

![Java instalado](img/java_instalado.png)

---

# IDEs y editores

Durante la práctica se han instalado varios editores de código.

## Notepad++

Editor ligero utilizado para revisar scripts y notas rápidas.

![Notepad++](img/notepad.png)

---

## Visual Studio Code

Editor avanzado utilizado para trabajar con scripts y organizar el código.

![VSCode](img/vscode.png)

---

## PyCharm

IDE orientado a desarrollo en Python.

![PyCharm](img/pycharm.png)

---

# Herramientas de reversing y debugging

## Ghidra

Herramienta de ingeniería inversa utilizada para analizar el binario del programa.

![Ghidra](img/ghidra.png)

---

## IDA Free

Desensamblador utilizado para analizar la estructura del programa.

![IDA Free](img/ida.png)

---

## Immunity Debugger

Debugger utilizado para observar el comportamiento del programa durante la ejecución.

![Immunity Debugger](img/immunity.png)

---

# Mona

Mona es una herramienta que se utiliza dentro de Immunity Debugger para facilitar diferentes fases del análisis de explotación.

El script de Mona se ha copiado en la carpeta `PyCommands` del debugger.

![Mona instalado](img/mona.png)

---

# Utilidades adicionales

## Git

Git se utiliza para clonar los repositorios proporcionados en el curso.

![Git instalado](img/git.png)

---

## Nmap / Ncat

Ncat se utiliza para comunicarse con aplicaciones vulnerables durante el laboratorio.

![Ncat](img/ncat.png)

---

# Aplicaciones vulnerables

Para el laboratorio se han descargado diferentes aplicaciones vulnerables utilizadas para practicar análisis y explotación.

Entre ellas se encuentra Vulnserver, un servidor vulnerable diseñado para practicar explotación de software.

![Vulnserver](img/vulnserver.png)

---

# Conclusión

La preparación del entorno de laboratorio es una parte fundamental del proceso de análisis de vulnerabilidades.

Instalar y configurar manualmente las herramientas permite comprender mejor cómo se utilizan durante un análisis real y cómo se integran dentro del flujo de trabajo de un analista de seguridad.
