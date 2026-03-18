# 🧨 Explotación de vulnerabilidad - FreeFloat FTP Server

## 📌 Descripción

En esta sección se documenta el proceso completo de explotación de una vulnerabilidad de tipo **stack buffer overflow** en el servicio **FreeFloat FTP Server**, dentro de un entorno de laboratorio controlado.

El objetivo es demostrar cómo, a partir de un fallo en la gestión de memoria, es posible tomar el control del flujo de ejecución y conseguir una shell remota.

---

## 🧪 Entorno de laboratorio

- **Máquina víctima:** Windows con FreeFloat FTP Server
- **Máquina atacante:** Kali Linux
- **Herramientas utilizadas:**
  - IDA Free
  - Immunity Debugger
  - Mona.py
  - Python
  - msfvenom
  - Metasploit

---

## 🔍 1. Análisis estático con IDA

Se abrió el binario del servidor FTP en **IDA Free** para realizar un análisis inicial del código.

Para facilitar la lectura del flujo del programa, se activó la vista en pseudocódigo desde:

`View → Open Subviews → Generate Pseudocode`

Después, se sincronizó el diagrama de flujo con el panel de pseudocódigo usando:

`Click derecho sobre el panel de flujo → Synchronize with → Pseudocode-A`

![Vista de IDA con pseudocódigo](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/vista%20ida%20geenrate.png)

A continuación, se revisaron las cadenas del binario para localizar comandos interesantes del protocolo FTP. Para ello se abrió la subvista **Strings** y se filtró por `USER`.

![Vista sincronizada](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/vista%20sincronize.png)

![Búsqueda de strings](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/vista%20string.png)

A partir de esa cadena se revisaron sus referencias cruzadas para identificar qué función la utilizaba.

![Gráfico de referencias](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/graf.png)

Del análisis se observó que una de las funciones implicadas realizaba operaciones inseguras sobre memoria, como el uso de `strcpy` sin una validación adecuada del tamaño de entrada.

![Uso inseguro de strcpy](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/strcpy.png)

---

## ⚙️ 2. Ejecución y depuración del servicio

Con la información obtenida en el análisis estático, se inició el servidor FTP haciendo doble clic sobre el binario.

![Servidor FTP ejecutándose](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/ftpserver.png)

Después se abrió **Immunity Debugger** y se adjuntó al proceso del servidor para monitorizar su actividad:

`File → Attach → seleccionar proceso → Enter`

![Adjuntar proceso en Immunity](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/attach%20ftp.png)

Una vez adjuntado, se obtuvo la vista del depurador asociada al proceso.

![Proceso adjuntado en Immunity](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/immunity_ftp.png)

> **Nota:** Immunity pausa la ejecución por defecto. Por eso fue necesario pulsar el botón **Play** para que el servicio siguiera funcionando con normalidad.

Antes de comenzar con las pruebas, se verificó que el servicio estaba activo y aceptando conexiones:

```bash
ncat localhost 21
```

![Comprobación del servicio con ncat](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/NCATRUN.png)

---

## 💥 3. Fuzzing

Confirmado que el servicio estaba activo, se pasó a la fase de **fuzzing**, enviando buffers de tamaño creciente para identificar si alguna entrada provocaba un fallo.

![Ejecución del fuzzing](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/fuzzing.png)

Tras varias iteraciones, el servicio perdió la conexión al alcanzar un tamaño determinado. Al revisar el estado del proceso en **Immunity Debugger**, se observó que el registro **EIP** había sido sobrescrito con el valor `41414141`, correspondiente a la letra `A`.

![Sobrescritura del EIP](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/register.png)

Esto confirmó que el desbordamiento era real y que existía la posibilidad de controlar el flujo de ejecución.

---

## ⚙️ 4. Configuración de Mona

Como paso previo a la explotación, se configuró **Mona** para trabajar con un directorio propio:

```text
!mona config -set workingfolder c:\mona
```

![Configuración de Mona](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/mona.png)

---

## 🎯 5. Control del EIP

Para calcular con precisión el punto exacto en el que se sobrescribía el **EIP**, se generó un patrón único de 400 bytes con Mona:

```text
!mona pattern_create 400
```

El patrón se copió al script correspondiente y se envió al servicio.

![Patrón generado e insertado en el script](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/patterm.png)

Tras lanzar la prueba y observar el nuevo valor de **EIP**, se utilizó Mona para calcular el offset exacto:

```text
!mona pattern_offset 41326941
```

![Cálculo del offset](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/offset.png)

El resultado obtenido fue un **offset de 246 bytes**. Para confirmarlo, se construyó una nueva entrada formada por:

- 246 caracteres `A`
- 4 caracteres `B` para sobrescribir el EIP

![Valor observado en EIP](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/eip.png)

![Script de validación del EIP](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/pythoneip.png)

![EIP sobrescrito con 42424242](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/eip42.png)

![Confirmación visual del offset](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/aaabbb.png)

El valor `42424242` en **EIP** confirmó que el control del registro era correcto.

---

## 🚫 6. Identificación de Bad Characters

Una vez controlado el **EIP**, el siguiente paso fue localizar los caracteres problemáticos o **badchars**, es decir, aquellos bytes que interrumpen o alteran la ejecución del payload.

Para ello se generó un array de bytes con Mona:

```text
!mona bytearray
```

Ese contenido se añadió al script encargado de enviar la prueba.

![Script con bytearray](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/script05.png)

Después de provocar el crash, se siguió el contenido apuntado por **ESP** usando la opción **Follow in Dump** y se comparó la memoria real con el bytearray generado por Mona.

![Seguimiento en dump](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/badchar41.png)

```text
!mona compare -f c:\mona\bytearray.bin -a 0092FBE4
```

![Comparación con Mona](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/0092FBE4.png)

En la primera iteración se detectó el byte `\x00`, por lo que se eliminó del array y se regeneró el bytearray excluyéndolo:

```text
!mona bytearray -b "\x00"
```

Tras repetir el proceso varias veces, se determinó que los **badchars** para este caso eran:

```text
\x00 \x0A \x0D
```

![Badchars finales](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/00.png)

---

## 🔁 7. Búsqueda de JMP ESP

El siguiente paso fue localizar una instrucción **JMP ESP** válida, que permitiera redirigir la ejecución hacia la pila, donde más adelante se colocaría el shellcode.

Primero se obtuvo el opcode correspondiente con Mona:

```text
!mona asm -s "jmp esp"
```

Resultado:

```text
\xff\xe4
```

![Opcode de JMP ESP](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/xe4.png)

Después se buscaron direcciones que contuvieran ese opcode y que no incluyeran ninguno de los badchars detectados:

```text
!mona find -s "\xff\xe4" -cpb "\x00\x0A\x0D"
```

![Búsqueda de dirección JMP ESP](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/758FA007.png)

Se seleccionó una dirección válida y se introdujo en el script para sobrescribir el **EIP**.

![Script con dirección JMP ESP](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/06script.png)

Al ejecutar el script, se comprobó que el flujo continuaba por la zona prevista, comenzando por una secuencia de NOPs.

![Validación del salto](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/mov.png)

---

## 💣 8. Generación del shellcode

Con la parte estructural de la explotación ya preparada, se generó un payload con **msfvenom** desde la máquina Kali Linux.

![Generación de shellcode con msfvenom](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/kali_msf.png)

En este laboratorio, el payload era de tipo **reverse TCP**, por lo que se configuró para que la máquina víctima se conectara a la IP y puerto de la máquina Kali.

---

## 🎯 9. Explotación final

Una vez reemplazado el shellcode de prueba por el shellcode real y ajustados los valores definitivos en el script, se ejecutó la versión final del exploit.

![Script final de explotación](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/07script.png)

El programa continuó en estado **running**, lo que indicaba que el proceso no se había cerrado y que el flujo de ejecución había sido redirigido correctamente.

Por último, en la consola de Kali donde se había dejado preparado el handler de Metasploit, se recibió la shell remota.

![Shell remota obtenida](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/final.png)

---

## ✅ Conclusión

Durante esta práctica se demostró que el servicio **FreeFloat FTP Server** presentaba una vulnerabilidad de tipo **stack buffer overflow** explotable en un entorno controlado.

A lo largo del proceso se consiguió:

- provocar el fallo mediante fuzzing,
- calcular el offset exacto al **EIP**,
- identificar los **badchars**,
- localizar una instrucción **JMP ESP** válida,
- redirigir el flujo de ejecución,
- y finalmente obtener una **shell remota**.

Este laboratorio permitió recorrer de forma práctica todas las fases habituales de una explotación clásica de desbordamiento de búfer, desde el análisis inicial hasta la validación final del impacto.
