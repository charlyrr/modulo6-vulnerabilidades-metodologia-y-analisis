# modulo6-vulnerabilidades-metodologia-y-analisis
# Módulo 6 – Vulnerabilidades  
## Práctica individual – Metodología de análisis y explotación

Autor: Carlos Ruesca
Máster en Ciberseguridad  

---

# 1. Introducción

En esta práctica se presenta una propuesta técnica sobre la metodología de análisis de vulnerabilidades y explotación de software. El objetivo principal del trabajo no es únicamente ejecutar una prueba técnica, sino entender cómo trabaja un analista de seguridad cuando encuentra un posible fallo en un programa o servicio.

Para realizar esta práctica se han utilizado como referencia el manual del módulo, la tarea propuesta y las explicaciones vistas durante las clases. A partir de estos recursos se ha definido una metodología de trabajo, se ha preparado un entorno de laboratorio y se ha documentado un caso práctico en un entorno controlado.

El trabajo se ha organizado en varias secciones: primero se explica la metodología de análisis de vulnerabilidades, después se describe el entorno de laboratorio utilizado y las herramientas necesarias, posteriormente se documenta un caso práctico realizado en laboratorio y finalmente se describe una posible aproximación ante una vulnerabilidad desconocida o 0day.

---

# 2. Objetivo de la práctica

El objetivo de esta práctica es desarrollar una visión clara sobre cómo abordar el análisis de vulnerabilidades desde un punto de vista técnico y metodológico.

Los objetivos principales son:

- Comprender el proceso de análisis de una vulnerabilidad.
- Preparar un entorno de laboratorio adecuado para realizar pruebas de seguridad.
- Utilizar herramientas de análisis estático y dinámico.
- Documentar un caso práctico de forma clara y ordenada.
- Describir un posible flujo de trabajo ante una vulnerabilidad desconocida.

Esta práctica intenta reflejar cómo trabajaría un analista de seguridad en un entorno real, donde es necesario seguir un proceso estructurado para identificar, analizar y documentar fallos en software.

---

# 3. Metodología de análisis de vulnerabilidades

Para el desarrollo de esta práctica se ha seguido una metodología basada en varias fases. Esta metodología permite analizar un posible fallo de seguridad de forma estructurada.

Las fases principales son las siguientes:

## 3.1 Reconocimiento del objetivo

En esta fase se intenta entender el software que se va a analizar. Esto incluye identificar el tipo de programa, la versión utilizada, los servicios que expone y la forma en la que los usuarios o clientes interactúan con él.

También se analiza el contexto del sistema para entender qué partes del programa pueden ser más sensibles desde el punto de vista de seguridad.

---

## 3.2 Identificación de comportamientos anómalos

Una vez comprendido el objetivo, el siguiente paso es comprobar cómo se comporta el programa ante diferentes entradas.

Se realizan pruebas controladas enviando distintos tipos de datos para observar si el software responde correctamente o si aparece algún comportamiento inesperado.

El objetivo de esta fase es detectar posibles fallos o errores que puedan indicar la existencia de una vulnerabilidad.

---

## 3.3 Clasificación de la vulnerabilidad

Cuando se detecta un comportamiento anómalo, es importante clasificar el tipo de vulnerabilidad que puede existir.

Algunos ejemplos comunes de vulnerabilidades pueden ser:

- errores de validación de entrada
- corrupción de memoria
- desbordamientos de buffer
- fallos en la gestión de memoria
- errores en el control de acceso

Clasificar correctamente el fallo ayuda a entender mejor su impacto y su posible explotación.

---

## 3.4 Análisis estático y dinámico

En esta fase se utilizan herramientas de análisis para entender mejor el funcionamiento interno del programa.

El análisis puede realizarse de dos formas:

**Análisis estático**

Consiste en analizar el programa sin ejecutarlo. Para ello se utilizan herramientas de reversing que permiten revisar funciones, estructuras y pseudocódigo.

**Análisis dinámico**

Consiste en ejecutar el programa y observar su comportamiento en tiempo real utilizando un debugger. Esto permite analizar qué ocurre cuando el programa recibe ciertas entradas.

---

## 3.5 Validación del fallo

Cuando se identifica un posible fallo, es necesario comprobar que realmente es reproducible.

Para ello se repiten las pruebas en un entorno controlado hasta confirmar que el comportamiento anómalo aparece siempre bajo determinadas condiciones.

Esto permite asegurar que el problema no es un error puntual.

---

## 3.6 Evaluación del impacto

Una vez confirmado el fallo, el siguiente paso es evaluar qué impacto podría tener.

Por ejemplo:

- si puede provocar un crash del programa
- si permite manipular memoria
- si podría afectar al control del flujo de ejecución

El objetivo es entender la gravedad del problema desde el punto de vista de seguridad.

---

## 3.7 Documentación del análisis

Finalmente, todo el proceso debe documentarse correctamente.

La documentación debe incluir:

- descripción del entorno
- herramientas utilizadas
- pasos realizados
- capturas del proceso
- conclusiones del análisis

Una buena documentación permite que otros analistas puedan entender el proceso seguido.

---

# 4. Entorno de laboratorio

Para realizar esta práctica se ha utilizado un entorno de laboratorio aislado. Trabajar en un entorno controlado es importante para evitar afectar a otros sistemas y poder analizar el comportamiento del software de forma segura.

El laboratorio se ha preparado utilizando una máquina virtual dedicada a pruebas de seguridad.

### Captura del entorno de laboratorio

[CAPTURA 1 – Máquina virtual o entorno de laboratorio]

---

# 5. Herramientas utilizadas

Durante el análisis se han utilizado varias herramientas que permiten estudiar el comportamiento del programa.

## 5.1 Debugger

El debugger permite observar el estado del programa cuando se ejecuta. Gracias a esta herramienta es posible analizar registros, memoria y flujo de ejecución cuando ocurre un fallo.

[CAPTURA 2 – Debugger abierto]

---

## 5.2 Herramienta de análisis estático

Las herramientas de análisis estático permiten cargar el binario del programa y revisar su estructura interna, incluyendo funciones, llamadas y pseudocódigo.

Esto ayuda a comprender mejor cómo está construido el software.

[CAPTURA 3 – Binario cargado en herramienta de análisis]

---

## 5.3 Editor o entorno de desarrollo

También se ha utilizado un editor para revisar scripts de prueba y organizar el análisis realizado durante el laboratorio.

[CAPTURA 4 – Editor o IDE]

---

## 5.4 Herramientas de comunicación con el servicio

Para interactuar con el servicio objetivo se han utilizado utilidades de red que permiten enviar datos al programa y observar su respuesta.

Esto permite comprobar cómo reacciona el software ante diferentes entradas.

[CAPTURA 5 – Conexión al servicio]

---

# 6. Caso práctico en laboratorio

En esta sección se documenta el caso práctico realizado durante el laboratorio.

---

## 6.1 Ejecución del programa objetivo

Primero se ha ejecutado el programa vulnerable dentro del entorno de laboratorio para comprobar su funcionamiento normal.

[CAPTURA 6 – Programa ejecutándose]

---

## 6.2 Verificación de comunicación

A continuación se ha comprobado que el servicio responde correctamente a entradas normales.

[CAPTURA 7 – Respuesta normal del servicio]

---

## 6.3 Pruebas de entradas controladas

Después se han realizado pruebas con entradas de diferente tamaño para observar si el programa mantiene un comportamiento estable.

Estas pruebas permiten detectar posibles errores o fallos.

[CAPTURA 8 – Pruebas realizadas]

---

## 6.4 Observación de un crash

Durante las pruebas se ha observado que el programa deja de responder cuando recibe determinadas entradas.

En ese momento se ha utilizado el debugger para analizar el estado del proceso.

[CAPTURA 9 – Crash observado]

---

## 6.5 Análisis del estado del programa

El debugger permite observar qué registros y qué zonas de memoria se ven afectadas cuando ocurre el fallo.

Esto ayuda a comprender mejor el origen del problema.

[CAPTURA 10 – Vista del debugger]

---

## 6.6 Análisis del comportamiento del fallo

Tras varias pruebas se ha podido confirmar que el fallo es reproducible en determinadas condiciones.

Este comportamiento indica que existe un problema en la forma en la que el programa maneja ciertos datos de entrada.

---

# 7. Aproximación ante una posible vulnerabilidad 0day

Si tuviera que analizar una vulnerabilidad desconocida, seguiría un proceso similar al siguiente:

1. Preparar un entorno de laboratorio aislado.
2. Identificar el software y la versión analizada.
3. Reproducir el comportamiento anómalo.
4. Realizar pruebas controladas para detectar fallos.
5. Analizar el crash utilizando un debugger.
6. Revisar el binario mediante análisis estático.
7. Comparar diferentes versiones del software si es posible.
8. Evaluar el impacto de la vulnerabilidad.
9. Documentar todas las evidencias.

Este proceso permite investigar vulnerabilidades de forma ordenada y reproducible.

---

# 8. Conclusiones

Esta práctica ha permitido entender mejor cómo se realiza el análisis de vulnerabilidades en un entorno controlado.

Más allá del uso de herramientas, lo más importante es seguir una metodología clara que permita identificar fallos, analizarlos y documentarlos correctamente.

También se ha comprobado que el análisis de software requiere combinar varias técnicas, como el análisis estático, el análisis dinámico y la observación del comportamiento del programa.

En conclusión, esta práctica ha ayudado a comprender mejor el proceso completo que sigue un analista de seguridad cuando estudia una vulnerabilidad.

---

# 9. Referencias

Manual del módulo 6 – Vulnerabilidades  
Material de clase del módulo  
Documentación técnica de herramientas de análisis
