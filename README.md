<div align="center">
  <img src="https://github.com/charlyrr/modulo6-vulnerabilidades-metodologia-y-analisis/raw/main/IMG/portada_cve.png" alt="Portada CVE-2025-5548" width="800">

  <h1>🛡️ Análisis y Explotación: CVE-2025-5548</h1>
  <p><strong>Máster en Ciberseguridad | Módulo 6: Vulnerabilidades</strong></p>

  <img src="https://img.shields.io/badge/OS-Windows%2011-blue?style=flat-square&logo=windows" alt="Windows 11">
  <img src="https://img.shields.io/badge/Language-Python%203-yellow?style=flat-square&logo=python" alt="Python">
  <img src="https://img.shields.io/badge/Vulnerability-Buffer%20Overflow-red?style=flat-square" alt="Buffer Overflow">
  <img src="https://img.shields.io/badge/Tools-Immunity%20Debugger%20%7C%20Mona-lightgrey?style=flat-square" alt="Tools">
</div>

---

## 👋 ¡Hola! Bienvenido a mi Taller de Análisis

En este repositorio documento mi inmersión práctica en el análisis de vulnerabilidades a nivel de binario y el desarrollo de *exploits* para entornos Windows. 

El objetivo principal de este laboratorio es interiorizar la metodología técnica para pasar del descubrimiento inicial de un fallo de corrupción de memoria (**Stack Buffer Overflow**) hasta lograr la ejecución remota de código (**RCE**) explotando el **CVE-2025-5548**.

No se trata solo de lanzar herramientas, sino de entender cómo se corrompe la memoria, cómo tomar el control del flujo de ejecución (Registro EIP) y cómo evadir las restricciones para inyectar nuestro propio *Shellcode*.

---

## 📂 Estructura del Repositorio

Para mantener el proyecto organizado y simular el flujo de trabajo real de un analista, he dividido la investigación en dos fases principales:

### 🛠️ [01 - Preparación del Entorno](./01%20Entorno/)
El taller del analista. Aquí documento cómo he configurado una máquina virtual aislada (Windows 11) y el despliegue del arsenal necesario para el análisis:
* Configuración de la VM y Snapshots.
* Instalación de herramientas de *Reversing* y *Debugging* (Ghidra, IDA Free, Immunity Debugger + Mona.py).
* Herramientas de red y lenguajes base (Python, Ncat).

### 🚀 [02 - Análisis y Explotación](./02%20Explotacion/)
La materialización del riesgo. Contiene el paso a paso metodológico del desarrollo del *exploit* para el **CVE-2025-5548**:
* **Fase 1:** Fuzzing y provocación del *crash*.
* **Fase 2 y 3:** Cálculo del Offset y secuestro del registro EIP.
* **Fase 4:** Identificación de *Badchars*.
* **Fase 5:** Búsqueda del trampolín (`JMP ESP`).
* **Fase 6:** Generación del *Shellcode* y obtención de la *Reverse Shell*.

---

## ⚠️ Aviso Legal y Ético

> **Nota:** Este proyecto se ha realizado en un entorno de laboratorio estrictamente controlado y aislado. Todos los experimentos, análisis y desarrollo de *exploits* tienen fines **única y exclusivamente educativos** como parte de mi formación en ciberseguridad. No me hago responsable del mal uso que se le pueda dar a la información o a los scripts aquí publicados.

---
**Autor:** [Tu Nombre / Charlyrr]  
**Asignatura:** Metodología y Análisis de Vulnerabilidades








------------------------------
# modulo6-vulnerabilidades-metodologia-y-analisis

# Práctica Módulo 6: Vulnerabilidades - Máster en Ciberseguridad

¡Hola! Bienvenid@ a mi repositorio de prácticas para el Módulo 6 de Vulnerabilidades. 

En este proyecto documento mi primera aproximación al análisis de vulnerabilidades a nivel de binario y al desarrollo de *exploits* para entornos Windows (Stack Buffer Overflow). El objetivo principal de este laboratorio es interiorizar la metodología técnica para pasar del descubrimiento inicial de un fallo de desbordamiento de memoria hasta la ejecución remota de código (RCE).

## Estructura del Repositorio

Para mantener el proyecto organizado y facilitar su revisión, he dividido el trabajo en dos fases principales:

* 📁 **[01 Entorno](./01%20Entorno/):** Contiene la documentación sobre cómo he configurado la máquina virtual (Windows 11), las herramientas de depuración instaladas y las configuraciones de seguridad.
* 📁 **[02 explotación](./02%20Explotacion/):** Contiene el paso a paso metodológico del desarrollo del *exploit*, desde la fase de *fuzzing* inicial hasta la obtención de la *Reverse Shell*.

> **Nota:** Este proyecto se ha realizado en un entorno de laboratorio controlado y ético, con fines estrictamente educativos como parte de mi formación en ciberseguridad.
