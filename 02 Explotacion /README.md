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
```text
!mona jmp -r esp -cpb "\x00\x0a\x0d"
