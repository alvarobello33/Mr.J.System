# 🖥️ Mr.J.System — Sistema Distribuido Gotham, Fleck, Enigma y Harley

**Miembros:**  
- Carla Francos Molina  
- Álvaro Bello Garrido  

## 🚀 Descripción general

Este proyecto implementa un **sistema distribuido** con arquitectura cliente-servidor y varios procesos especializados que cooperan mediante mecanismos de concurrencia.

El sistema consta de los siguientes programas principales:

- **Gotham** → Servidor central que coordina toda la red.  
- **Fleck** → Cliente que solicita procesamiento (“distorsión”) de archivos.  
- **Enigma** → *Worker* especializado en distorsión de **texto**.  
- **Harley** → *Worker* especializado en distorsión de **media (audio/video)**.  
- **Arkham** → Proceso de *logging* independiente que registra los eventos del sistema.

El proyecto ha sido desarrollado en **C a bajo nivel** para la asignatura de *Sistemas Operativos*, y utiliza **sockets TCP**, **hilos POSIX (pthreads)**, **memoria compartida** y **pipes** para coordinar conexiones concurrentes, transferir archivos de forma fiable y proporcionar tolerancia a fallos mediante *heartbeats* y *failover automático*.

## ⚙️ Características clave

- **Gotham** gestiona dos servidores listeners TCP independientes: uno para **Fleck** y otro para **Workers**.  
  Cada conexión se atiende con un **hilo dedicado**, protegido por mutex, y se monitoriza mediante **heartbeats** para detectar caídas.

- **Workers (Enigma y Harley)** se registran en Gotham.  
  - Se elige un *worker principal* por tipo (texto o media).  
  - En caso de fallo, Gotham reasigna automáticamente el rol principal (*failover*).  

- **Fleck** solicita una operación de distorsión a Gotham.  
  - Gotham responde con la información del *worker principal*.  
  - Fleck transfiere el archivo en **tramas de 256 bytes** con verificación MD5 y protocolo de reintento (*CheckOK / CheckKO*).  

- **Arkham** es un proceso hijo creado con `fork()`.  
  - Recibe los mensajes de log desde Gotham mediante **pipe** y los escribe secuencialmente en un fichero de logs, evitando intercalado concurrente.

- **Concurrencia y sincronización** sobre estructuras globales compartidas gestionadas con `pthread_mutex`.  

## ⚙️ Configuración de archivos (Project/data/)

Antes de compilar el proyecto se deben configurar los archivos dentro de `Project/data/` con el siguiente formato:

`gotham.dat`:
```
<IP_Gotham>
<Puerto_Servidor_Flecks_In_Gotham>
<IP_Gotham>
<Puerto_Servidor_Workers_In_Gotham>
```
`worker.dat` (Enigma o Harley):
```
<IP_Gotham>
<Puerto_Servidor_Workers_In_Gotham>
<IP_Worker>
<Puerto_Servidor_Flecks_Worker>
```
`fleck.dat`:
```
<IP_Gotham>
<Puerto_Servidor_Flecks_In_Gotham>
```

---

## 🛠️ Compilación con Makefile

RECORDATORIO: Antes de compilar el proyecto se debe realizar la configuración de archivos `Project/data/` indicada en el apartado anterior.

El proyecto incluye un **Makefile** que ofrece los siguientes comandos los cuales pueden ser ejecutados por terminal dentro del directorio `Project/`:

| Objetivo | Descripción |
|-----------|--------------|
| `make` | Compilación estándar |
| `make debug` | Compilación en modo depuración |
| `make clean` | Limpieza de objetos y binarios |

>💡 Se debe compilar utilizando el compilador **GCC** y se recomienda ejecutar en un entorno **Linux**.

---

▶️ Ejecución

Los programas Gotham, Fleck y Workers (Enigma y Harley) se pueden ejecutar en máquinas diferentes utilizando cada uno una IP diferente. Pero todas las instancias de los programas de tipo Worker (Enigma y Harley) deben ser ejecutadas en la misma máquina para que compartan la dirección IP.

### Ejemplo de orden de ejecución:

```bash
# Servidor central (Ordenador 1)
./gotham.exe data/gotham.dat

# Workers
## Worker Enigma (Ordenador 2)
./enigma.exe data/enigma.dat
## Worker Harley (Ordenador 2)
./harley.exe data/harley.dat

# Cliente1 (Ordenador 3)
./fleck.exe data/fleck.dat
# Cliente2 (Ordenador 4)
./fleck.exe data/fleck.dat
```
