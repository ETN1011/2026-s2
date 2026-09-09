# ETN1011 --- Laboratorio de Sistemas de Comunicación II

## Clase de orientación previa al Laboratorio 1

**Duración:** 4 horas\
**Entorno de referencia del instructor:** Debian GNU/Linux\
**Entorno recomendado para estudiantes con Windows:** WSL2 con Ubuntu o
Debian\
**Herramientas transversales:** OpenCode, Git, GitHub y Markdown

------------------------------------------------------------------------

## 1. Propósito de la clase

Esta clase prepara el entorno de trabajo que se utilizará durante el
curso. No se presupone dominio previo de GNU/Linux. El estudiante
aprenderá los elementos necesarios mientras los utiliza y podrá apoyarse
en OpenCode para consultar comandos, interpretar resultados y
diagnosticar problemas.

Al finalizar la clase, el estudiante deberá ser capaz de:

-   explicar qué es GNU/Linux y por qué se utilizará en el laboratorio;
-   instalar y ejecutar una distribución GNU/Linux mediante WSL2;
-   desplazarse por el sistema de archivos y realizar operaciones
    básicas con archivos y directorios;
-   reconocer la estructura principal de directorios de Linux;
-   consultar y reutilizar el historial de comandos;
-   utilizar OpenCode como herramienta de asistencia técnica;
-   comprender la relación entre Git, GitHub, repositorios y archivos
    Markdown;
-   configurar Git y enlazarlo con una cuenta de GitHub;
-   clonar el repositorio de la materia;
-   identificar interfaces, direcciones, rutas y vecinos de red en
    Linux;
-   verificar conectividad;
-   comprender el propósito de los *network namespaces* y de los enlaces
    virtuales `veth`;
-   construir, de forma guiada, una topología virtual mínima.

> **Criterio de trabajo:** se permite y fomenta el uso de herramientas
> de IA. El estudiante sigue siendo responsable de comprender, verificar
> y defender técnicamente los comandos, configuraciones y conclusiones
> que presente.

------------------------------------------------------------------------

# 2. ¿Qué es GNU/Linux?

Linux es el núcleo (*kernel*) de un sistema operativo. Administra
recursos como CPU, memoria, dispositivos, procesos y comunicaciones de
red. En el uso cotidiano, el kernel Linux se combina con herramientas,
bibliotecas y utilidades del proyecto GNU y de otros proyectos para
formar sistemas operativos completos, habitualmente denominados
distribuciones GNU/Linux.

Ejemplos de distribuciones son:

-   Debian;
-   Ubuntu;
-   Fedora;
-   Arch Linux.

Una distribución incorpora el kernel Linux y selecciona herramientas,
instaladores, repositorios de software, mecanismos de actualización y
políticas de mantenimiento.

En esta asignatura el instructor utilizará **Debian**. Los estudiantes
que trabajen desde Windows podrán utilizar **Ubuntu o Debian sobre
WSL2**. El objetivo no es aprender una distribución concreta, sino
utilizar las capacidades de red de Linux y herramientas abiertas que
funcionan de manera similar en ambas.

## 2.1. ¿Por qué Linux en Sistemas de Comunicación II?

Linux es especialmente adecuado para el laboratorio porque permite
observar y modificar directamente elementos fundamentales de una red:

-   interfaces;
-   direcciones IPv4 e IPv6;
-   tablas de encaminamiento;
-   vecinos ARP/ND;
-   puentes;
-   interfaces virtuales;
-   *network namespaces*;
-   captura de paquetes;
-   encaminamiento y reenvío de paquetes.

Sobre esta base se incorporarán posteriormente FRRouting, Docker y
containerlab.

La progresión general será:

``` text
Linux
  │
  ├── interfaces y direccionamiento
  ├── routing
  ├── namespaces y veth
  │
  ├── FRRouting
  ├── contenedores
  └── containerlab
          │
          └── topologías de redes y BGP
```

------------------------------------------------------------------------

# 3. WSL2: Linux dentro de Windows

WSL (*Windows Subsystem for Linux*) permite ejecutar un entorno
GNU/Linux directamente desde Windows. WSL2 utiliza un kernel Linux y
permite ejecutar herramientas de línea de comandos y gran parte del
software habitual de Linux.

WSL se utilizará como mecanismo de acceso a Linux para quienes no
dispongan de una instalación Linux nativa.

## 3.1. Instalación

Abrir **PowerShell como administrador**.

Para instalar WSL con la distribución predeterminada, Ubuntu:

``` powershell
wsl --install
```

Reiniciar Windows si el instalador lo solicita.

Para conocer las distribuciones disponibles:

``` powershell
wsl --list --online
```

Para instalar explícitamente Ubuntu:

``` powershell
wsl --install -d Ubuntu
```

Para instalar Debian:

``` powershell
wsl --install -d Debian
```

Comprobar las distribuciones instaladas y la versión de WSL:

``` powershell
wsl --list --verbose
```

El laboratorio utilizará **WSL2**.

La primera ejecución de Ubuntu o Debian solicitará crear un usuario y
una contraseña propios del entorno Linux.

## 3.2. Identificación del sistema

Una vez dentro de Linux:

``` bash
whoami
uname -a
cat /etc/os-release
```

Preguntas de comprobación:

1.  ¿Qué distribución está ejecutando?
2.  ¿Qué kernel aparece?
3.  ¿El usuario de Linux es necesariamente el mismo usuario de Windows?

------------------------------------------------------------------------

# 4. Primeros comandos de Linux

No se pretende estudiar toda la administración de Linux. Se introducirán
los comandos necesarios para trabajar durante la asignatura.

## 4.1. ¿Dónde estoy?

``` bash
pwd
```

`pwd` muestra el directorio de trabajo actual.

## 4.2. Listar contenido

``` bash
ls
ls -l
ls -la
```

## 4.3. Cambiar de directorio

``` bash
cd directorio
cd ..
cd ~
```

## 4.4. Crear archivos y directorios

``` bash
mkdir prueba
touch prueba/archivo.txt
```

## 4.5. Visualizar archivos

``` bash
cat archivo.txt
less archivo.txt
```

## 4.6. Copiar, mover y eliminar

``` bash
cp origen destino
mv origen destino
rm archivo
rmdir directorio_vacio
rm -r directorio
```

**Precaución:** Linux normalmente no pide confirmación para todas las
operaciones destructivas. Antes de ejecutar un comando sugerido por una
persona, una página web o una IA, el estudiante debe comprender qué
modifica.

## 4.7. Ayuda

``` bash
man ls
ls --help
```

Una habilidad importante del laboratorio será aprender a consultar
documentación, no memorizar todas las opciones.

------------------------------------------------------------------------

# 5. Estructura de directorios de Linux

Linux organiza el sistema de archivos a partir de una única raíz:

``` text
/
├── bin
├── boot
├── dev
├── etc
├── home
│   └── usuario
├── proc
├── root
├── run
├── sys
├── tmp
├── usr
└── var
```

Directorios relevantes para el curso:

  -----------------------------------------------------------------------
  Directorio                          Propósito general
  ----------------------------------- -----------------------------------
  `/`                                 raíz del sistema de archivos

  `/home`                             directorios personales de los
                                      usuarios

  `/etc`                              configuración del sistema y
                                      aplicaciones

  `/usr`                              programas, bibliotecas y datos
                                      compartidos

  `/var`                              información variable, registros y
                                      otros datos

  `/tmp`                              archivos temporales

  `/proc`                             información expuesta por el kernel
                                      sobre procesos y sistema

  `/sys`                              información y objetos relacionados
                                      con kernel y dispositivos
  
  `/dev`                              dispositivos representados como
                                      archivos
  
  -----------------------------------------------------------------------

Ejercicio:

``` bash
cd /
ls
cd /etc
pwd
cd ~
pwd
```

En WSL también puede observarse la integración con los discos de
Windows, por ejemplo bajo `/mnt/c`.

------------------------------------------------------------------------

# 6. Historial de comandos

La terminal conserva un historial que evita volver a escribir comandos y
permite revisar el trabajo realizado.

``` bash
history
```

Buscar en el historial:

``` bash
history | grep ip
```

También puede utilizarse:

-   `↑` y `↓` para recorrer comandos anteriores;
-   `Ctrl+R` para realizar una búsqueda interactiva en el historial.

El historial será útil durante la elaboración de las bitácoras de
laboratorio, pero **no reemplaza la documentación consciente del
procedimiento**.

------------------------------------------------------------------------

# 7. OpenCode como herramienta de asistencia

OpenCode es un agente de IA de código abierto que puede utilizarse desde
la terminal. En la asignatura será una herramienta transversal para:

-   explicar comandos desconocidos;
-   proponer procedimientos;
-   interpretar mensajes de error;
-   analizar salidas de comandos;
-   ayudar a documentar;
-   apoyar el diagnóstico.

No sustituye la comprensión del estudiante.

Un flujo de trabajo apropiado es:

``` text
problema
   │
   ▼
análisis del estudiante
   │
   ▼
consulta a OpenCode
   │
   ▼
propuesta
   │
   ▼
comprender antes de ejecutar
   │
   ▼
ejecutar
   │
   ▼
verificar con evidencia
```

## 7.1. Instalación

Actualizar primero el sistema:

``` bash
sudo apt update
sudo apt upgrade
```

Instalar `curl` si fuera necesario:

``` bash
sudo apt install curl
```

Instalación mediante el procedimiento oficial de OpenCode:

``` bash
curl -fsSL https://opencode.ai/install | bash
```

Comprobar:

``` bash
opencode --version
```

Iniciar:

``` bash
opencode
```

La configuración del proveedor/modelo de IA dependerá de los recursos
disponibles para el estudiante y se realizará siguiendo las indicaciones
vigentes de OpenCode.

## 7.2. Primer ejercicio

Consultar, por ejemplo:

``` text
Explícame qué hacen pwd, ls -la y cd ~.
No ejecutes nada; quiero comprenderlos antes de utilizarlos.
```

Posteriormente se utilizará OpenCode con información real de networking.

------------------------------------------------------------------------

# 8. GitHub, repositorios y Markdown

## 8.1. ¿Qué es un repositorio?

Un repositorio es un espacio de trabajo cuyo contenido y cambios pueden
ser gestionados mediante un sistema de control de versiones.

Durante la asignatura cada laboratorio generará configuraciones,
documentación y evidencias. En lugar de manejar versiones como:

``` text
lab1-final.md
lab1-final2.md
lab1-ahorasi-final.md
```

se utilizará Git.

## 8.2. Git y GitHub no son lo mismo

``` text
Git
│
├── control de versiones
├── historial
└── repositorio local
        │
        │ push / pull
        ▼
GitHub
├── repositorio remoto
├── colaboración
└── publicación del trabajo
```

**Git** es el sistema de control de versiones.

**GitHub** es una plataforma que permite alojar y colaborar sobre
repositorios Git.

## 8.3. Markdown

La documentación de los laboratorios se realizará principalmente en
archivos de texto Markdown, identificados normalmente por la extensión
`.md`.

Ejemplo:

``` markdown
# Laboratorio 1

## Objetivo

Construir una topología virtual sobre Linux.

## Prueba de conectividad

Se ejecutó:

`ping 10.10.1.2`

**Resultado:** conectividad correcta.
```

Markdown permite que la documentación sea:

-   legible como texto;
-   sencilla de editar;
-   adecuada para Git;
-   renderizada directamente por GitHub.

------------------------------------------------------------------------

# 9. Creación de una cuenta de GitHub

Cada estudiante deberá disponer de una cuenta personal en GitHub y de
una dirección de correo verificada.

Crear la cuenta desde el sitio oficial de GitHub y conservar el nombre
de usuario elegido, ya que formará parte de la identidad utilizada en
los repositorios.

Una vez creada la cuenta, verificar que se puede iniciar sesión
correctamente antes de continuar.

------------------------------------------------------------------------

# 10. Git en Linux

Instalar Git:

``` bash
sudo apt install git
```

Comprobar:

``` bash
git --version
```

Configurar identidad:

``` bash
git config --global user.name "Nombre Apellido"
git config --global user.email "correo@example.com"
```

Comprobar:

``` bash
git config --global --list
```

## 10.1. Comandos básicos

Durante esta primera etapa utilizaremos principalmente:

``` bash
git clone
git status
git add
git commit
git pull
git push
git log
```

Interpretación:

  |Comando      |  Función                                    |
  |-------------|---------------------------------------------|
  |`git clone`  |  obtiene una copia de un repositorio        |
  |`git status` |  muestra el estado del directorio de trabajo|
  |`git add`    |  selecciona cambios para el próximo commit  |
  |`git commit` |  registra un cambio en el historial local   |
  |`git pull`   |  incorpora cambios del repositorio remoto   |
  |`git push`   |  publica commits en el repositorio remoto   |
  |`git log`    |  consulta el historial                      |

------------------------------------------------------------------------

# 11. Enlace de Git con GitHub mediante SSH

Para los laboratorios se recomienda utilizar SSH.

## 11.1. Crear una clave

``` bash
ssh-keygen -t ed25519 -C "correo@example.com"
```

Aceptar la ubicación propuesta o seleccionar conscientemente otra.

Comprobar los archivos:

``` bash
ls -la ~/.ssh
```

Mostrar la **clave pública**:

``` bash
cat ~/.ssh/id_ed25519.pub
```

> Nunca publicar, enviar ni copiar a GitHub el archivo `id_ed25519` sin
> `.pub`. Ese archivo contiene la clave privada.

Agregar la clave pública a la cuenta de GitHub siguiendo la opción de
claves SSH de la configuración de la cuenta.

Probar:

``` bash
ssh -T git@github.com
```

La primera conexión puede solicitar confirmar la identidad del servidor.

------------------------------------------------------------------------

# 12. Clonado del repositorio de la materia

El instructor proporcionará la dirección SSH del repositorio oficial de
ETN1011.

Ejemplo de forma general:

``` bash
git clone git@github.com:ORGANIZACION/ETN1011.git
```

Entrar al repositorio:

``` bash
cd ETN1011
git status
```

Examinar:

``` bash
ls -la
```

A partir de este punto el repositorio será una de las vías de
distribución de guías, topologías, configuraciones y otros materiales de
trabajo de la materia.

------------------------------------------------------------------------

# 13. Preparación de Linux para networking

Instalar las herramientas iniciales:

``` bash
sudo apt update
sudo apt install iproute2 iputils-ping traceroute mtr-tiny tcpdump iperf3 git curl
```

## 13.1. Interfaces

``` bash
ip link
```

Pregunta:

> ¿Qué interfaces existen y cuáles están activas?

## 13.2. Direccionamiento

``` bash
ip addr
```

Identificar:

-   nombre de interfaz;
-   dirección IPv4;
-   longitud de prefijo;
-   dirección IPv6, si existe.

## 13.3. Tabla de rutas

``` bash
ip route
```

Identificar:

-   redes directamente conectadas;
-   ruta por defecto;
-   gateway;
-   interfaz de salida.

## 13.4. Vecinos

``` bash
ip neigh
```

Relacionar esta información con ARP en IPv4 y Neighbor Discovery en
IPv6.

## 13.5. Conectividad

``` bash
ping 8.8.8.8
ping google.com
```

Si el primero funciona y el segundo no, investigar qué servicio
adicional podría estar fallando.

Consultar el recorrido:

``` bash
traceroute 8.8.8.8
```

o:

``` bash
mtr 8.8.8.8
```

### Ejercicio con OpenCode

Ejecutar:

``` bash
ip addr
ip route
```

Proporcionar las salidas relevantes a OpenCode y solicitar:

``` text
Analiza estas salidas de ip addr e ip route.

1. Identifica mi interfaz principal.
2. Identifica mi dirección IPv4 y prefijo.
3. Identifica mi gateway predeterminado.
4. Explica cómo saldría un paquete destinado a 8.8.8.8.
5. Indica qué afirmaciones debo verificar manualmente.
```

El estudiante debe comprobar la respuesta contra la salida original.

------------------------------------------------------------------------

# 14. Network namespaces

Hasta ahora se ha trabajado con el espacio de red principal de Linux.

Un *network namespace* permite disponer de un espacio de red aislado con
sus propias interfaces, direcciones, rutas y otros elementos de
networking.

Esto permite utilizar una sola computadora para representar varios
nodos.

Conceptualmente:

``` text
                     Linux

        ┌────────────────────────┐
        │ namespace hostA        │
        │                        │
        │ interfaces             │
        │ direcciones            │
        │ rutas                  │
        └────────────────────────┘

        ┌────────────────────────┐
        │ namespace hostB        │
        │                        │
        │ interfaces             │
        │ direcciones            │
        │ rutas                  │
        └────────────────────────┘
```

Listar namespaces:

``` bash
ip netns list
```

Crear dos:

``` bash
sudo ip netns add hostA
sudo ip netns add hostB
```

Comprobar:

``` bash
ip netns list
```

Ejecutar un comando dentro de un namespace:

``` bash
sudo ip netns exec hostA ip link
```

Observar que el entorno de red visto desde `hostA` es diferente del
espacio de red principal.

------------------------------------------------------------------------

# 15. Enlaces virtuales veth

Un par `veth` puede imaginarse como un cable Ethernet virtual con dos
extremos.

``` text
extremo A ================= extremo B
  vethA          veth         vethB
```

Los paquetes que entran por un extremo aparecen en el otro.

Crear el par:

``` bash
sudo ip link add vethA type veth peer name vethB
```

Mover cada extremo:

``` bash
sudo ip link set vethA netns hostA
sudo ip link set vethB netns hostB
```

Ahora:

``` text
┌───────────────────┐              ┌───────────────────┐
│ hostA             │              │ hostB             │
│                   │              │                   │
│      vethA        ├══════════════┤       vethB       │
│                   │              │                   │
└───────────────────┘              └───────────────────┘
```

Comprobar:

``` bash
sudo ip netns exec hostA ip link
sudo ip netns exec hostB ip link
```

En la clase de orientación el instructor puede completar de forma guiada
una topología mínima o detenerse aquí para que el direccionamiento y la
puesta en servicio formen parte del Laboratorio 1.

------------------------------------------------------------------------

# 16. Topología que se desarrollará en el Laboratorio 1

El laboratorio partirá de dos nodos Linux aislados:

``` text
        hostA                              hostB

┌───────────────────┐              ┌───────────────────┐
│ network namespace │              │ network namespace │
│                   │              │                   │
│      vethA        ├══════════════┤       vethB       │
│                   │              │                   │
└───────────────────┘              └───────────────────┘
```

El estudiante deberá completar el direccionamiento, habilitar las
interfaces, obtener conectividad, inspeccionar rutas y vecinos, provocar
una falla controlada, diagnosticarla y documentar el resultado.

El Laboratorio 1 utilizará conjuntamente:

``` text
WSL2 / Linux
     │
     ├── OpenCode ── asistencia
     │
     ├── Git ─────── control de versiones
     │
     ├── GitHub ──── repositorio remoto
     │
     ├── Markdown ── documentación
     │
     └── iproute2
           │
           ├── interfaces
           ├── direccionamiento
           ├── routing
           ├── namespaces
           └── veth
```

------------------------------------------------------------------------

# 17. Cierre y preparación para el laboratorio

Antes de la siguiente clase cada estudiante deberá comprobar:

-   [ ] WSL2 funcionando, si utiliza Windows.
-   [ ] Ubuntu o Debian instalado.
-   [ ] Acceso a Internet desde Linux.
-   [ ] OpenCode instalado y operativo.
-   [ ] Cuenta de GitHub creada.
-   [ ] Git instalado y configurado.
-   [ ] Autenticación SSH con GitHub funcionando.
-   [ ] Repositorio de la materia clonado.
-   [ ] `ip`, `ping`, `traceroute`, `mtr` y `tcpdump` disponibles.
-   [ ] Capacidad de crear y eliminar un *network namespace*.
-   [ ] Capacidad de consultar documentación y explicar los comandos
    utilizados.

## Preguntas de repaso

1.  ¿Qué diferencia existe entre Linux y una distribución GNU/Linux?
2.  ¿Por qué utilizamos Linux en este laboratorio?
3.  ¿Qué diferencia existe entre Debian, Ubuntu y WSL?
4.  ¿Qué representa `/` en Linux?
5.  ¿Para qué sirven `/home`, `/etc`, `/proc` y `/dev`?
6.  ¿Qué diferencia existe entre Git y GitHub?
7.  ¿Qué es un repositorio?
8.  ¿Por qué Markdown resulta adecuado para documentar laboratorios?
9.  ¿Qué información proporcionan `ip link`, `ip addr`, `ip route` e
    `ip neigh`?
10. ¿Qué es un *network namespace*?
11. ¿Qué representa un par `veth`?
12. ¿Qué responsabilidad conserva el estudiante cuando utiliza OpenCode?

------------------------------------------------------------------------

# 18. Distribución sugerida de las cuatro horas

  -----------------------------------------------------------------------
                                    Tiempo Actividad
  ---------------------------------------- ------------------------------
                                0:00--0:30 GNU/Linux, distribuciones y
                                           propósito en la asignatura

                                0:30--1:00 WSL2, instalación de
                                           Ubuntu/Debian y primer acceso

                                1:00--1:30 archivos, directorios,
                                           estructura Linux e historial

                                1:30--2:00 OpenCode: concepto,
                                           instalación y uso responsable

                                2:00--2:30 GitHub, repositorios y
                                           Markdown

                                2:30--3:00 Git, SSH, GitHub y clonado del
                                           repositorio de la materia

                                3:00--3:30 interfaces, direccionamiento,
                                           routing, vecinos y
                                           conectividad

                                3:30--4:00 namespaces, veth y
                                           presentación del problema del
                                           Laboratorio 1
  -----------------------------------------------------------------------

> Los tiempos son orientativos. Las instalaciones que requieran descarga
> o reinicio pueden adelantarse o realizarse parcialmente antes de la
> clase para preservar el tiempo destinado a conceptos y demostraciones.

------------------------------------------------------------------------

# Referencias de consulta

-   Microsoft Learn --- documentación oficial de Windows Subsystem for
    Linux.
-   OpenCode --- documentación oficial.
-   Git --- documentación oficial.
-   GitHub Docs --- documentación de cuentas, repositorios y
    autenticación SSH.
-   `man` y páginas de ayuda de las herramientas instaladas en Linux.
