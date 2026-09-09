# ETN1011 --- Laboratorio de Sistemas de Comunicación II

## Guía de Laboratorio 1 --- Redes virtuales con Linux

**Duración:** 4 horas\
**Modalidad:** práctica y defensa\
**Entorno:** GNU/Linux Debian o Ubuntu; Linux nativo o WSL2\
**Herramientas:** `iproute2`, OpenCode, Git, GitHub y Markdown

------------------------------------------------------------------------

## 1. Propósito

Construir y verificar una red virtual aislada utilizando las capacidades
de networking del kernel Linux. El estudiante deberá crear nodos
mediante *network namespaces*, interconectarlos con interfaces virtuales
`veth`, asignar direccionamiento IPv4, comprobar la conectividad,
observar el estado de interfaces, rutas y vecinos, provocar una falla y
diagnosticarla.

El laboratorio parte de una infraestructura ya preparada. Se supone que
el estudiante dispone de:

-   GNU/Linux operativo;
-   WSL2 configurado, cuando corresponda;
-   OpenCode instalado y configurado;
-   Git instalado y configurado;
-   cuenta de GitHub y autenticación funcionando;
-   repositorio de la materia clonado;
-   herramientas básicas de networking instaladas.

La instalación de estas herramientas **no forma parte del tiempo de
ejecución del laboratorio**.

------------------------------------------------------------------------

## 2. Resultados esperados

Al finalizar, el estudiante deberá ser capaz de:

1.  identificar interfaces, direcciones, rutas y vecinos de un sistema
    Linux;
2.  explicar qué es un *network namespace*;
3.  crear espacios de red aislados;
4.  explicar el funcionamiento de un par `veth`;
5.  interconectar dos namespaces;
6.  configurar direccionamiento IPv4;
7.  interpretar las rutas generadas por Linux;
8.  comprobar conectividad mediante `ping`;
9.  observar la resolución de vecinos;
10. provocar y diagnosticar una falla de conectividad;
11. documentar el trabajo en Markdown;
12. registrar y publicar su trabajo mediante Git y GitHub;
13. utilizar OpenCode como apoyo sin sustituir la comprensión y
    verificación técnica.

------------------------------------------------------------------------

# 3. Reglas de trabajo

El laboratorio se realizará sobre una red virtual aislada. No se
modificarán redes institucionales ni equipos de producción.

Se permite consultar:

-   páginas de manual de Linux;
-   documentación técnica;
-   apuntes de la materia;
-   OpenCode y otras herramientas autorizadas por el docente.

El uso de una herramienta de IA no convierte su respuesta en evidencia
técnica. Toda propuesta deberá ser comprendida, ejecutada
conscientemente y verificada mediante resultados observables.

El estudiante debe poder explicar durante la defensa cualquier comando o
configuración incluida en su entrega.

------------------------------------------------------------------------

# 4. Preparación del repositorio

Ingresar al repositorio de trabajo de la materia y actualizarlo:

``` bash
cd ETN1011
git pull
```

Crear, cuando la organización del repositorio de la materia así lo
indique, el espacio correspondiente al Laboratorio 1.

Una estructura sugerida es:

``` text
lab01/
├── README.md
├── topologia.md
├── comandos.md
└── evidencias/
```

La estructura concreta podrá ser definida por el docente en el
repositorio oficial.

Antes de iniciar:

``` bash
git status
```

Registrar en `README.md`:

-   nombre del estudiante;
-   fecha;
-   distribución utilizada;
-   si se utiliza Linux nativo, máquina virtual o WSL2;
-   versión del kernel.

La información puede obtenerse con:

``` bash
whoami
uname -a
cat /etc/os-release
```

------------------------------------------------------------------------

# 5. Reconocimiento de la red Linux

Antes de crear la topología virtual, estudiar el entorno de red actual.

Ejecutar:

``` bash
ip link
ip addr
ip route
ip neigh
```

## Actividad 1

Identifique y documente:

-   interfaz o interfaces existentes;
-   estado de cada interfaz;
-   dirección IPv4 principal;
-   longitud de prefijo;
-   ruta por defecto;
-   gateway, cuando exista;
-   interfaz utilizada por la ruta por defecto;
-   vecinos actualmente conocidos.

No es suficiente copiar la salida completa. Explique qué elementos
relevantes encontró.

## Prueba de conectividad

Ejecute:

``` bash
ping -c 4 8.8.8.8
ping -c 4 google.com
```

Cuando esté disponible:

``` bash
traceroute 8.8.8.8
```

o:

``` bash
mtr -r -c 5 8.8.8.8
```

### Pregunta

Si `ping 8.8.8.8` funciona pero `ping google.com` no funciona, ¿qué
componente adicional investigaría primero? Justifique.

------------------------------------------------------------------------

# 6. Problema de laboratorio

Se dispone de una sola computadora Linux y se requiere representar dos
nodos de red independientes.

Debe construirse la siguiente topología:

``` text
             Linux

       hostA                           hostB
┌───────────────────┐          ┌───────────────────┐
│ network namespace │          │ network namespace │
│                   │          │                   │
│  10.10.1.1/30     │          │  10.10.1.2/30     │
│       vethA       ├══════════┤       vethB       │
│                   │   veth   │                   │
└───────────────────┘          └───────────────────┘
```

Características:

  Elemento            Valor
  ------------------- ----------------
  Namespace 1         `hostA`
  Namespace 2         `hostB`
  Interfaz de hostA   `vethA`
  Interfaz de hostB   `vethB`
  Dirección hostA     `10.10.1.1/30`
  Dirección hostB     `10.10.1.2/30`

El objetivo es obtener conectividad IPv4 entre `hostA` y `hostB`.

------------------------------------------------------------------------

# 7. Construcción de la topología

A diferencia de una receta de instalación, esta sección proporciona los
elementos necesarios y exige que el estudiante comprenda la secuencia.

## 7.1. Crear los namespaces

Comandos de referencia:

``` bash
sudo ip netns add hostA
sudo ip netns add hostB
```

Verifique:

``` bash
ip netns list
```

### Evidencia requerida

Documente los namespaces creados y explique qué aislamiento proporciona
un *network namespace*.

------------------------------------------------------------------------

## 7.2. Crear el enlace virtual

Crear un par `veth`:

``` bash
sudo ip link add vethA type veth peer name vethB
```

Antes de mover las interfaces, compruebe que existen:

``` bash
ip link
```

### Pregunta

¿Por qué las dos interfaces aparecen inicialmente en el espacio de red
principal?

------------------------------------------------------------------------

## 7.3. Mover las interfaces

Asignar cada extremo al namespace correspondiente:

``` bash
sudo ip link set vethA netns hostA
sudo ip link set vethB netns hostB
```

Compruebe nuevamente:

``` bash
ip link
```

Después:

``` bash
sudo ip netns exec hostA ip link
sudo ip netns exec hostB ip link
```

### Preguntas

1.  ¿Por qué `vethA` y `vethB` dejaron de aparecer en el `ip link` del
    espacio principal?
2.  ¿Dónde existe ahora cada interfaz?

------------------------------------------------------------------------

# 8. Direccionamiento

Configure las direcciones indicadas en la topología.

Para ejecutar un comando dentro de un namespace se utiliza:

``` bash
sudo ip netns exec NAMESPACE COMANDO
```

Configure:

``` bash
sudo ip netns exec hostA ip addr add 10.10.1.1/30 dev vethA
sudo ip netns exec hostB ip addr add 10.10.1.2/30 dev vethB
```

Verifique:

``` bash
sudo ip netns exec hostA ip addr
sudo ip netns exec hostB ip addr
```

### Actividad 2

Determine para `10.10.1.0/30`:

-   dirección de red;
-   direcciones utilizables;
-   dirección de broadcast;
-   cantidad de direcciones utilizables.

Explique por qué `10.10.1.1` y `10.10.1.2` pueden pertenecer al mismo
enlace.

------------------------------------------------------------------------

# 9. Puesta en servicio de interfaces

Una interfaz configurada no necesariamente se encuentra operativa.

Compruebe el estado:

``` bash
sudo ip netns exec hostA ip link
sudo ip netns exec hostB ip link
```

Habilite las interfaces requeridas:

``` bash
sudo ip netns exec hostA ip link set lo up
sudo ip netns exec hostA ip link set vethA up

sudo ip netns exec hostB ip link set lo up
sudo ip netns exec hostB ip link set vethB up
```

Compruebe nuevamente.

### Pregunta

¿Qué diferencia observa entre una interfaz existente, una interfaz
direccionada y una interfaz en estado `UP`?

------------------------------------------------------------------------

# 10. Tabla de rutas

Observe las tablas de rutas:

``` bash
sudo ip netns exec hostA ip route
sudo ip netns exec hostB ip route
```

### Actividad 3

Sin agregar ninguna ruta manual, explique:

1.  qué ruta existe en `hostA`;
2.  qué ruta existe en `hostB`;
3.  por qué apareció esa ruta;
4.  qué interfaz se utilizará para llegar al otro nodo;
5.  si es necesario configurar un gateway para esta topología.

**No continúe hasta poder justificar la respuesta a la pregunta 5.**

------------------------------------------------------------------------

# 11. Prueba de conectividad

Desde `hostA`:

``` bash
sudo ip netns exec hostA ping -c 4 10.10.1.2
```

Desde `hostB`:

``` bash
sudo ip netns exec hostB ping -c 4 10.10.1.1
```

Registre los resultados.

Si la prueba falla, **no destruya inmediatamente la topología para
comenzar de nuevo**. Diagnostique.

Una secuencia razonable de investigación puede considerar:

``` text
¿Existe el namespace?
        ↓
¿Existe la interfaz?
        ↓
¿Está UP?
        ↓
¿Tiene la dirección correcta?
        ↓
¿Existe la ruta?
        ↓
¿Existe el vecino?
        ↓
¿Llegan los paquetes?
```

------------------------------------------------------------------------

# 12. Tabla de vecinos

Después de realizar los `ping`, ejecute:

``` bash
sudo ip netns exec hostA ip neigh
sudo ip netns exec hostB ip neigh
```

### Actividad 4

Documente:

-   dirección IP del vecino;
-   dirección MAC observada;
-   estado de la entrada.

Explique por qué una comunicación IPv4 entre dos nodos del mismo enlace
necesita conocer una dirección de capa 2.

------------------------------------------------------------------------

# 13. Observación del tráfico

Abra una terminal adicional y capture tráfico en uno de los namespaces:

``` bash
sudo ip netns exec hostA tcpdump -n -i vethA
```

Desde `hostB`, genere tráfico:

``` bash
sudo ip netns exec hostB ping -c 4 10.10.1.1
```

Observe la captura.

### Actividad 5

Identifique, cuando sean visibles:

-   tráfico ARP;
-   solicitudes ICMP Echo Request;
-   respuestas ICMP Echo Reply.

Explique la secuencia observada.

Detenga `tcpdump` con `Ctrl+C`.

------------------------------------------------------------------------

# 14. Falla controlada y diagnóstico

El objetivo de esta parte no es únicamente romper la red, sino
**predecir, observar y explicar** el comportamiento.

Antes de ejecutar el siguiente comando, escriba qué espera que ocurra.

Baje la interfaz de `hostB`:

``` bash
sudo ip netns exec hostB ip link set vethB down
```

Desde `hostA`:

``` bash
sudo ip netns exec hostA ping -c 4 10.10.1.2
```

Investigue mediante:

``` bash
sudo ip netns exec hostA ip link
sudo ip netns exec hostA ip addr
sudo ip netns exec hostA ip route
sudo ip netns exec hostA ip neigh

sudo ip netns exec hostB ip link
sudo ip netns exec hostB ip addr
sudo ip netns exec hostB ip route
```

Restaure el servicio:

``` bash
sudo ip netns exec hostB ip link set vethB up
```

Compruebe nuevamente la conectividad.

### Actividad 6

Documente:

1.  predicción previa;
2.  síntoma observado;
3.  comandos utilizados para diagnosticar;
4.  causa de la falla;
5.  acción correctiva;
6.  prueba que demuestra la recuperación.

------------------------------------------------------------------------

# 15. Reto de diagnóstico

El docente introducirá o indicará **una modificación adicional** en la
topología. El estudiante deberá diagnosticarla sin recibir la solución.

Ejemplos de fallas que el docente puede utilizar:

-   dirección IPv4 incorrecta;
-   longitud de prefijo incorrecta;
-   interfaz deshabilitada;
-   interfaz asignada al namespace equivocado;
-   ausencia de una dirección;
-   cambio de uno de los nodos a otra subred.

El estudiante puede utilizar OpenCode, documentación y páginas de
manual.

La defensa deberá explicar **por qué** la falla impedía la conectividad
y qué evidencia permitió localizarla.

------------------------------------------------------------------------

# 16. Uso de OpenCode

OpenCode puede utilizarse durante todo el laboratorio.

Ejemplos de consultas apropiadas:

``` text
Tengo dos network namespaces Linux conectados mediante un par veth.
No me des una solución completa.

Ayúdame a diagnosticar por qué no existe conectividad.
Indícame qué debería verificar primero y explícame por qué.
```

o:

``` text
Explícame esta salida de "ip route".
Distingue hechos observables de tus inferencias.
```

Se recomienda evitar consultas del tipo:

``` text
Hazme completo el laboratorio 1.
```

El propósito es utilizar la IA para **ampliar la capacidad de
análisis**, no para eliminarla.

En la documentación puede registrarse una intervención de IA que haya
sido especialmente útil:

``` markdown
## Uso de OpenCode

### Problema
...

### Consulta
...

### Propuesta obtenida
...

### Verificación
...

### Explicación técnica
...
```

------------------------------------------------------------------------

# 17. Documentación del laboratorio

El archivo principal `README.md` deberá contener como mínimo:

``` markdown
# Laboratorio 1 — Redes virtuales con Linux

## 1. Datos del entorno

## 2. Objetivo

## 3. Topología

## 4. Direccionamiento

## 5. Construcción

## 6. Verificación

### Interfaces
### Direcciones
### Rutas
### Vecinos
### Conectividad

## 7. Captura y análisis de tráfico

## 8. Falla y diagnóstico

## 9. Uso de OpenCode

## 10. Conclusiones
```

No es necesario pegar indiscriminadamente todas las salidas de terminal.
Seleccione aquellas que constituyan evidencia de las afirmaciones
realizadas.

------------------------------------------------------------------------

# 18. Registro con Git

Durante el trabajo deben realizarse commits que representen avances
comprensibles.

Por ejemplo:

``` bash
git status
git add .
git commit -m "Documenta topologia del laboratorio 1"
```

Posteriormente:

``` bash
git add .
git commit -m "Agrega pruebas de conectividad"
```

Y al finalizar:

``` bash
git add .
git commit -m "Completa diagnostico y conclusiones del laboratorio 1"
```

Consultar:

``` bash
git log --oneline
```

Publicar según el flujo establecido para el repositorio:

``` bash
git push
```

El objetivo no es producir muchos commits, sino que el historial
represente el desarrollo del trabajo.

------------------------------------------------------------------------

# 19. Evidencias mínimas de entrega

La entrega deberá permitir comprobar:

-   topología utilizada;
-   direccionamiento;
-   existencia de ambos namespaces;
-   interfaces dentro de cada namespace;
-   tablas de rutas;
-   conectividad entre `hostA` y `hostB`;
-   tabla de vecinos;
-   observación de tráfico;
-   falla controlada;
-   diagnóstico y recuperación;
-   conclusiones;
-   historial Git;
-   publicación del trabajo según las instrucciones del repositorio.

El repositorio debe permitir que otra persona comprenda y reproduzca el
procedimiento.

------------------------------------------------------------------------

# 20. Limpieza del entorno

Después de que el docente haya revisado las evidencias, elimine la
topología:

``` bash
sudo ip netns del hostA
sudo ip netns del hostB
```

Compruebe:

``` bash
ip netns list
```

Al eliminar un namespace se eliminan también las interfaces virtuales
contenidas en él cuando ya no tienen otro extremo persistente que las
mantenga.

------------------------------------------------------------------------

# 21. Defensa individual

La defensa forma parte del laboratorio. El docente podrá solicitar
modificaciones y preguntas sobre la topología en ejecución.

Preguntas posibles:

1.  ¿Qué es un *network namespace*?
2.  ¿Qué recursos de networking quedan aislados?
3.  ¿Qué es un par `veth`?
4.  ¿Por qué `vethA` no aparece en el `ip link` del espacio principal
    después de moverla?
5.  ¿Qué significa `/30`?
6.  ¿Qué ruta utiliza `hostA` para llegar a `10.10.1.2`?
7.  ¿Por qué no se necesita un gateway en esta topología?
8.  ¿Qué función cumple ARP?
9.  ¿Qué diferencia existe entre `ip link`, `ip addr`, `ip route` e
    `ip neigh`?
10. ¿Qué ocurrirá si se baja `vethB`?
11. ¿Qué ocurriría si `hostB` se configura como `10.10.2.1/30`?
12. ¿Cómo demostraría que un problema es de capa 2, direccionamiento o
    routing?
13. ¿Qué información obtuvo de OpenCode y cómo comprobó que era
    correcta?
14. ¿Qué aporta Git al trabajo realizado?
15. ¿Podría otra persona reproducir su topología a partir del
    repositorio?

El docente podrá modificar una dirección, interfaz o estado de enlace y
solicitar al estudiante que diagnostique el problema.

------------------------------------------------------------------------

# 22. Criterios de evaluación

De acuerdo con el criterio general de evaluación de laboratorios de la
asignatura:

  Criterio                   Peso
  ------------------------ ------
  Funcionamiento             40 %
  Pruebas y diagnóstico      30 %
  Reproducibilidad           20 %
  Explicación individual     10 %

## Funcionamiento --- 40 %

Se comprobará que:

-   la topología solicitada existe;
-   el direccionamiento es correcto;
-   las interfaces se encuentran en los namespaces correspondientes;
-   existe conectividad extremo a extremo.

## Pruebas y diagnóstico --- 30 %

Se evaluará:

-   selección adecuada de comandos de verificación;
-   interpretación de rutas y vecinos;
-   observación del tráfico;
-   diagnóstico de la falla;
-   demostración de la recuperación.

## Reproducibilidad --- 20 %

Se evaluará:

-   claridad del Markdown;
-   topología y direccionamiento documentados;
-   comandos relevantes;
-   evidencias seleccionadas;
-   historial Git;
-   posibilidad de reproducir el ejercicio.

## Explicación individual --- 10 %

El estudiante deberá explicar y modificar su configuración durante la
defensa sin depender de una secuencia memorizada.

------------------------------------------------------------------------

# 23. Distribución sugerida de las cuatro horas

        Tiempo Actividad
  ------------ -----------------------------------------------------
    0:00--0:20 preparación del repositorio y reconocimiento de red
    0:20--1:20 construcción de namespaces, veth y direccionamiento
    1:20--2:00 rutas, conectividad y vecinos
    2:00--2:30 captura y análisis de tráfico
    2:30--3:00 falla controlada y reto de diagnóstico
    3:00--3:20 documentación, commits y publicación
    3:20--4:00 defensa individual y modificaciones solicitadas

Los tiempos son orientativos. La topología deberá permanecer disponible
hasta concluir la defensa.

------------------------------------------------------------------------

# 24. Resultado conceptual del Laboratorio 1

Al finalizar se habrá construido:

``` text
una computadora
       │
       ▼
     Linux
       │
       ├───────────────┐
       ▼               ▼
     hostA           hostB
 namespace         namespace
       │               │
     vethA ═════════ vethB
       │               │
10.10.1.1/30      10.10.1.2/30
```

Sobre esta base podrán construirse posteriormente nodos que actúen como
routers y utilizar herramientas de routing dinámico como **FRRouting**.

El objetivo del siguiente nivel ya no será únicamente comunicar dos
hosts de una misma red, sino comprender cómo interconectar **redes
diferentes**.
