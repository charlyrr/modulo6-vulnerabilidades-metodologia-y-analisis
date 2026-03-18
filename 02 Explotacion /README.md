# 🧨 Explotación de vulnerabilidad - FreeFloat FTP Server

## 📌 Descripción

En este documento se explica paso a paso el análisis y la explotación de una vulnerabilidad de tipo **stack buffer overflow** en el servicio **FreeFloat FTP Server**, dentro de un entorno de laboratorio controlado.

La idea no es solo enseñar los comandos o las herramientas utilizadas, sino entender qué está pasando en memoria y por qué este fallo permite cambiar el flujo normal del programa hasta conseguir acceso remoto al sistema.

---

## 🧪 Entorno de laboratorio

Para realizar la práctica se ha utilizado el siguiente entorno:

- **Máquina víctima:** Windows con FreeFloat FTP Server
- **Máquina atacante:** Kali Linux

### Herramientas utilizadas

- **IDA Free:** para revisar el binario y entender cómo trata las entradas.
- **Immunity Debugger:** para observar el estado del programa en ejecución.
- **Mona.py:** para automatizar cálculos y búsquedas en memoria.
- **Python:** para crear los scripts de prueba y explotación.
- **msfvenom:** para generar el payload.
- **Metasploit:** para recibir la conexión inversa del shellcode.

---

## 🔍 1. Análisis estático con IDA

La primera parte del trabajo consistió en abrir el binario en **IDA Free** para revisar cómo procesa la aplicación los datos que recibe del usuario.

Para facilitar el análisis se activó la vista en pseudocódigo, ya que ayuda a entender mejor la lógica del programa que ver solo ensamblador:

`View → Open Subviews → Generate Pseudocode`

También se sincronizó el panel del flujo con el pseudocódigo:

`Click derecho en el gráfico → Synchronize with → Pseudocode-A`

![Vista de IDA con pseudocódigo](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/vista%20ida%20geenrate.png)

Después se revisaron las cadenas del binario, buscando comandos típicos del protocolo FTP como `USER`, porque son puntos de entrada directos de datos enviados por el cliente.

![Vista sincronizada](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/vista%20sincronize.png)

![Búsqueda de strings](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/vista%20string.png)

Siguiendo las referencias cruzadas de esas cadenas se localizó una función que utiliza `strcpy`.

![Gráfico de referencias](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/graf.png)

![Uso de strcpy en la función vulnerable](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/strcpy.png)

### ¿Por qué es importante esto?

Porque `strcpy` copia datos de una zona de memoria a otra sin comprobar el tamaño. Si el dato de entrada es más grande que el búfer de destino, termina sobrescribiendo memoria cercana. Esa es precisamente la base de este tipo de vulnerabilidad.

---

## ⚙️ 2. Ejecución y debugging

Con esa primera pista, el siguiente paso fue ejecutar el servidor FTP en Windows y adjuntar el proceso al **Immunity Debugger**.

`File → Attach → seleccionar proceso`

![Servidor FTP ejecutándose](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/ftpserver.png)

![Adjuntando el proceso en Immunity](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/attach%20ftp.png)

![Proceso cargado en Immunity Debugger](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/immunity_ftp.png)

Trabajar con el programa dentro del debugger es importante porque permite parar la ejecución justo cuando falla y revisar el estado de registros clave como **EIP** y **ESP**.

Antes de empezar con las pruebas se comprobó que el servicio estaba activo y aceptando conexiones:

```bash
ncat localhost 21
```

![Comprobación del servicio con ncat](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/NCATRUN.png)

Esta comprobación sirve para asegurar que el servicio realmente está escuchando y que los fallos que aparezcan más adelante se deben a nuestras pruebas y no a que el servidor no estuviera funcionando.

---

## 💥 3. Fuzzing

Una vez confirmado que el servicio estaba operativo, se pasó a la fase de **fuzzing**.

En esta etapa se enviaron datos cada vez más largos con el objetivo de ver en qué momento el programa dejaba de responder. La idea aquí no es explotar todavía, sino encontrar el punto en el que la aplicación empieza a romperse por no gestionar bien el tamaño de la entrada.

![Ejecución del fuzzing](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/fuzzing.png)

Después de varias pruebas, el servidor terminó fallando. Al mirar el estado del proceso en Immunity Debugger se vio que el registro **EIP** había sido sobrescrito con el valor `41414141`, que corresponde a `AAAA`.

![Sobrescritura de EIP con 41414141](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/register.png)

### ¿Qué significa esto?

El registro **EIP** guarda la dirección de memoria de la siguiente instrucción que la CPU va a ejecutar. Si conseguimos que ahí aparezcan nuestros propios datos, significa que ya no solo hacemos caer el programa, sino que empezamos a controlar por dónde seguirá la ejecución.

Ese es el momento clave en el que el fallo pasa de ser un simple crash a convertirse en una vulnerabilidad explotable.

---

## ⚙️ 4. Configuración de Mona

Para facilitar el trabajo posterior se configuró **Mona** dentro de Immunity Debugger con un directorio de trabajo propio:

```text
!mona config -set workingfolder c:\mona
```

![Configuración inicial de Mona](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/mona.png)

### ¿Para qué sirve Mona?

Mona automatiza muchas tareas que, hechas a mano, serían lentas y propensas a error. Por ejemplo:
- generar patrones únicos,
- calcular offsets,
- crear arrays de bytes,
- buscar instrucciones útiles en memoria.

Gracias a esto el proceso de explotación es mucho más preciso.

---

## 🎯 5. Control del EIP

Después de comprobar que era posible sobrescribir el **EIP**, el siguiente objetivo fue averiguar exactamente en qué posición del input ocurría eso.

Para hacerlo se generó un patrón único de 400 bytes:

```text
!mona pattern_create 400
```

![Patrón generado con Mona](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/patterm.png)

Se envió ese patrón al servidor y, cuando volvió a fallar, se tomó el valor concreto que apareció en **EIP**.

![Valor de EIP tras enviar el patrón](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/eip.png)

Con ese dato se calculó el offset exacto:

```text
!mona pattern_offset 41326941
```

![Cálculo del offset](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/offset.png)

El resultado fue un **offset de 246 bytes**.

### ¿Qué quiere decir ese número?

Significa que los primeros 246 bytes de la cadena llenan el espacio previo al registro, y que justo después vienen los 4 bytes que sobrescriben el **EIP**.

Para comprobarlo se construyó una entrada con:
- 246 bytes de relleno (`A`)
- 4 bytes con `BBBB`

![Script de prueba para validar el EIP](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/pythoneip.png)

![EIP sobrescrito con 42424242](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/eip42.png)

![Confirmación visual de A y B en memoria](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/aaabbb.png)

Al ver `42424242` en el EIP, quedó confirmado que el cálculo era correcto y que el control sobre el flujo del programa era real.

---

## 🚫 6. Badchars (caracteres problemáticos)

Antes de preparar el payload final, era necesario identificar los **badchars**.

Los badchars son bytes que el programa interpreta de forma especial y que pueden romper el exploit. Por ejemplo, algunos terminan cadenas o modifican el contenido en memoria, haciendo que el shellcode se corte antes de tiempo.

Para encontrarlos se generó un array completo de bytes con Mona:

```text
!mona bytearray
```

![Script preparado con bytearray](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/script05.png)

Después se comparó ese array con la memoria real tras el desbordamiento:

```text
!mona compare -f c:\mona\bytearray.bin -a ESP
```

![Seguimiento en dump del contenido de ESP](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/badchar41.png)

![Comparación de Mona con el bytearray](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/0092FBE4.png)

Tras revisar los resultados, se vio que los bytes problemáticos eran:

```text
\x00
\x0A
\x0D
```

![Badchars finales detectados](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/00.png)

### ¿Por qué es importante esto?

Porque estos caracteres no pueden aparecer ni en la dirección que escribamos en **EIP** ni en el shellcode, ya que romperían la ejecución y el exploit dejaría de funcionar.

---

## 🔁 7. Localización del JMP ESP

El siguiente paso fue encontrar una forma fiable de hacer que el programa salte a la zona de memoria donde estará nuestro shellcode.

En lugar de apuntar a una dirección cualquiera, la técnica más habitual es buscar una instrucción **JMP ESP** dentro de un módulo cargado por el proceso.

Esto funciona porque **ESP** apunta a la parte superior de la pila, que es donde quedarán colocados nuestros datos después del desbordamiento. Si conseguimos que el **EIP** apunte a una instrucción `JMP ESP`, el programa saltará directamente a nuestro código.

La búsqueda se realizó con Mona, excluyendo los badchars detectados:

```text
!mona find -s "\xff\xe4" -cpb "\x00\x0A\x0D"
```

![Búsqueda del opcode JMP ESP](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/xe4.png)

![Dirección válida para JMP ESP](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/758FA007.png)

![Script con la dirección escogida](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/06script.png)

![Validación del salto en ejecución](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/mov.png)

El resultado permitió elegir una dirección válida para redirigir la ejecución.

---

## 💣 8. Generación del shellcode

Con la parte estructural del exploit ya preparada, se generó el payload con **msfvenom**.

En este caso se utilizó un shellcode de tipo **reverse TCP**, que hace que la máquina víctima se conecte de vuelta a la máquina atacante.

Durante la generación se excluyeron los badchars encontrados antes, para evitar que el payload se corrompiera al copiarse en memoria.

![Generación del shellcode con msfvenom](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/kali_msf.png)

### ¿Por qué usar un reverse shell?

Porque en un laboratorio es una forma clara y sencilla de comprobar que el código se ha ejecutado correctamente: si todo sale bien, el atacante recibe una conexión y obtiene una shell interactiva.

---

## 🎯 9. Explotación final

Con todos los elementos ya calculados, se construyó el exploit final con la siguiente estructura:

- bytes de relleno hasta llegar al **EIP**,
- dirección de retorno hacia `JMP ESP`,
- una pequeña zona de **NOPs** como margen de seguridad,
- el shellcode generado.

![Script final de explotación](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/07script.png)

Al lanzar el script final contra el servidor, el programa no se cerró de forma brusca, sino que redirigió la ejecución hacia la pila y terminó ejecutando el payload.

El resultado fue una conexión inversa recibida en Metasploit, lo que confirmó que se había conseguido una **shell remota** sobre la máquina víctima.

![Shell remota obtenida en Metasploit](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/final.png)

### ¿Qué demuestra este punto?

Demuestra que ya no estamos solo ante un fallo de estabilidad, sino ante una vulnerabilidad con capacidad real de producir **ejecución remota de código (RCE)** en un entorno controlado.

---

## ✅ Conclusión

Esta práctica ha permitido recorrer de forma completa el proceso de explotación de un **stack buffer overflow**, desde el análisis inicial del binario hasta la obtención de una shell remota.

A lo largo del laboratorio se consiguió:

- identificar una función insegura en el binario,
- provocar un fallo controlado mediante fuzzing,
- calcular el offset exacto al **EIP**,
- encontrar y excluir los **badchars**,
- localizar una instrucción **JMP ESP** válida,
- y finalmente ejecutar código arbitrario en la máquina vulnerable.

Además, este trabajo deja claro por qué este tipo de errores siguen siendo peligrosos: una gestión insegura de memoria puede terminar comprometiendo por completo un sistema.

Como medidas de mitigación, conviene:
- sustituir funciones inseguras como `strcpy` por alternativas seguras,
- validar correctamente el tamaño de las entradas,
- y apoyarse en protecciones modernas como **DEP** y **ASLR**.
