# 🧨 Explotación de vulnerabilidad - FreeFloat FTP Server

## 📌 Descripción

En este documento se describe paso a paso el proceso seguido para explotar una vulnerabilidad de tipo **buffer overflow en stack** en el servicio **FreeFloat FTP Server**, dentro de un entorno de laboratorio controlado.

El objetivo no es solo ejecutar herramientas, sino entender cómo un error en la gestión de memoria permite modificar el flujo de ejecución de un programa y obtener acceso remoto al sistema.

---

## 🧪 Entorno de laboratorio

- **Máquina víctima:** Windows con FreeFloat FTP Server  
- **Máquina atacante:** Kali Linux  

### Herramientas utilizadas:
- IDA Free  
- Immunity Debugger  
- Mona.py  
- Python  
- msfvenom  
- Metasploit  

---

## 🔍 1. Análisis estático con IDA

Se abrió el binario en **IDA Free** para entender cómo el programa procesa las entradas.

Se activó la vista en pseudocódigo para facilitar la lectura del código.

![IDA](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/vista%20ida%20geenrate.png)

Se buscaron cadenas como `USER`, ya que representan entradas del usuario.

![Strings](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/vista%20string.png)

Siguiendo las referencias se encontró una función que utiliza `strcpy`.

![strcpy](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/strcpy.png)

**Importante:** `strcpy` no valida el tamaño de los datos, lo que puede provocar un desbordamiento de memoria.

---

## ⚙️ 2. Ejecución y debugging

Se ejecutó el servidor FTP en Windows y se abrió **Immunity Debugger**.

Se adjuntó al proceso:

`File → Attach → seleccionar proceso`

![Immunity](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/immunity_ftp.png)

El debugger permite ver registros como EIP y ESP.

Se comprobó el servicio:

```bash
ncat localhost 21
```

---

## 💥 3. Fuzzing

Se enviaron datos cada vez más grandes hasta provocar un fallo.

![Fuzzing](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/fuzzing.png)

En el debugger:

![Crash](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/register.png)

EIP = 41414141

Esto indica que podemos sobrescribir la dirección de retorno.

---

## ⚙️ 4. Configuración de Mona

```text
!mona config -set workingfolder c:\mona
```

Mona ayuda a automatizar el análisis.

---

## 🎯 5. Control del EIP

```text
!mona pattern_create 400
```

```text
!mona pattern_offset 41326941
```

Resultado: 246

Esto significa que a partir del byte 246 controlamos EIP.

---

## 🚫 6. Badchars

```text
!mona bytearray
```

```text
!mona compare -f c:\mona\bytearray.bin -a ESP
```

Badchars:
\x00 \x0A \x0D

Estos bytes rompen el payload.

---

## 🔁 7. JMP ESP

```text
!mona find -s "\xff\xe4" -cpb "\x00\x0A\x0D"
```

Se busca una instrucción que salte al ESP (donde estará nuestro código).

---

## 💣 8. Shellcode

Se genera con msfvenom.

Este payload crea una conexión inversa hacia el atacante.

---

## 🎯 9. Explotación final

Se ejecuta el exploit con todos los valores correctos.

![Shell](https://raw.githubusercontent.com/charlyrr/CVE-2025-5548/main/02%20Explotacion%20/IMG/final.png)

El programa ejecuta nuestro código y se obtiene una shell remota.

---

## ✅ Conclusión

Se ha demostrado cómo un fallo de memoria puede permitir controlar un programa.

Se ha conseguido:

- Control del EIP  
- Redirección del flujo  
- Ejecución remota  

Además, estas vulnerabilidades pueden mitigarse con DEP, ASLR y validación de entradas.
