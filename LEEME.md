﻿# LFTP MIRROR

Es un script escrito en Python que nos permite sincronizar un directorio en un 
servidor remoto con un directorio local a través de FTP. Para ello hace uso del 
estupendo programa 'lftp' de Alexander V. Lukyanov (http://lftp.yar.ru/), que es 
necesario para que funcione este script.

A veces tenemos la necesidad de tener sincronizados dos directorios, uno en un
servidor remoto y otro almacenado localmente y solo disponemos de acceso FTP al 
servidor, sin posibilidad de emplear soluciones más idóneas a través de ssh como 
rsync. Por ejemplo, para hacer copias de seguridad de un sitio web en un hosting
compartido en el que no disponemos más que de una cuenta FTP para intercambiar 
archivos.

El problema con el que nos encontramos entonces es que realizar esta operación 
a través de FTP es lento, porqué nos obliga en cierto modo a descargar todo el 
contenido del directorio cada vez o bien controlar manualmente los cambios. 
Carece además, a diferencia de rsync, de la facultad de descargar únicamente 
aquella parte de los ficheros que ha cambiado. Algunos clientes de FTP (casi 
todos gráficos) nos permiten conocer que ficheros han cambiando y descargar 
estos únicamente, acelerando así el proceso de descargar y disminuyendo el 
tráfico de red. Uno de estos clientes, lftp, es además de uno los más ligeros y 
rápidos que existen, uno de los pocos que incorpora la funcionalidad para 
sincronizar dos carpetas por defecto, mirror. Esta sincronización es 
bidireccional, por lo cual se puede realizar en ambos sentidos: remoto a local 
ó local a remoto. Siendo como es un programa de linea de comandos y soportando 
la importación de configuración a través de script, es ideal para automatizar 
todo el proceso a través de un shell script y crontab, para realizar una 
sincronización programada completamente automática.

Dado que lftp soporta por defecto el empleo de scripts para automatizar las 
tareas de ftp y programarlo automáticamente a través de cron es trivial, ¿qué 
necesidad hay de crear un script como este?

Bueno, este script aporta ciertas ventajas sobre emplear únicamente lftp:

* Proporciona un registro de actividad detallado y legible que es almacenado
  en disco y puede ser enviado por correo electrónico a una o varias 
  direcciones, bien a través de un servidor local o uno externo. 
* Permite crear una copia comprimida por día de la semana del directorio 
  local sincronizado. Esto nos permite tener el directorio actualizado y una
  copia de seguridad por cada uno de los últimos 7 días, para poder revertir
  algún cambio o borrado accidental. 
* Nos proporciona (en el log) el tamaño del espacio ocupado por el 
  directorio local y las copias de seguridad en la unidad de almacenamiento.
* Se centra únicamente en la sincronización (mirror) entre directorios, 
  obviando las otras opciones que nos ofrece lftp.
* Permite tres modos de ejecución distintos, lo que lo convierte en muy 
  versátil:
    * Como tarea programada. En este modo los parámetros de la 
      sincronización se incluyen directamente dentro del script y solo es 
      necesario programar su ejecución para automatizar el proceso. Es ideal
      para la sincronización periódica de un único directorio/servidor FTP 
    * Interactivo. En este modo los parámetros se introducen directamente 
      como argumentos en la línea de comandos. Es ideal para ejecutar una 
      sincronización puntual manual.
    * Importando los parámetros desde un fichero de configuración. Este modo
      es similar al primero, con la diferencia de que en este caso los 
      parámetros los tomamos de un fichero de configuración externo. Este 
      fichero que podemos crear nosotros mismos (se sirve uno de ejemplo) 
      nos permite establecer múltiples operaciones de sincronización que se 
      ejecutaran de manera secuencial una detrás de otra. Así por ejemplo 
      podemos programarlo para que en una sola ejecución sincronice varios 
      directorios/servidores FTP, sin limite en el número de estos. Nos 
      permitiría por ejemplo crear varios ficheros de configuración que 
      podríamos programar para su ejecución automática a diferentes franjas 
      horarias, cubriendo de esto modo situaciones complejas en las que 
      pueda interesar sincronizar diferentes directorios/servidores FTP en 
      días/horas diferentes. Por ejemplo sincronizar unos directorios 
      (e.g. 10) por la noche y otros al mediodía(e.g. 3) durante los días 
      laborables y otros diferentes (e.g. 4) durante el fin de semana. Esto 
      se podría automatizar con solo tres ficheros de configuración (uno por 
      cada franja horaria) para sincronizar los 17 directorios/servidores 
      FTP, lo que nos permite una gran flexibilidad sin tener que crear 
      diferentes scripts para cada directorio/servidor FTP, solo 
      estableciendo los parámetros de cada acción.  
* En sistemas operativos que lo soporten nos muestra notificaciones
  emergentes a través de la librería libnotify de la ejecución del script y 
  su correcta finalización. Por ejemplo, a través de las notificaciones 
  emergentes de Ubuntu. Muy útil para conocer cuando se está ejecutando una 
  tarea programada sin salida por shell.
* Si empleamos los modos de ejecución no interactivos, emplea base64 para 
  una mínima protección de la contraseñas de acceso a los servidores FTP y 
  evitar almacenarlas las mismas en texto claro. No es una fuerte medida de 
  seguridad, pero es lo mínimo que deberíamos tener en cuenta.

## FICHEROS

* `lftp_mirror.py`

 Es el fichero del script

* `sample.cfg`

 Es un ejemplo de fichero de configuración

* `License.txt`

 El testo de la licencia GPLv3

* `LEEME.txt`

 Este fichero que estas leyendo

* `README.txt`

 La versión en ingles de este fichero


## REQUISITOS PREVIOS Y DEPENDENCIAS

### Para Linux (no testeado en Mac):

Lógicamente, lo primero que necesitamos para ejecutarlos es python. Si estamos
en Linux normalmente viene instalado por defecto y no es un problema.

Este script emplea varios módulos que están presentes en la biblioteca estándar
de python.

La versión de python necesaria para ejecutar el script es la 2.7. Ahora bien, es
posible correr el script en la versión 2.6 si instalamos el modulo argparse, que
se incorporó a la librería estándar en la versión 2.7.
 
Instalar el modulo argparse en linux es sencillo, por ejemplo en Debian ó Ubuntu
se haría del siguiente modo:

    $ sudo apt-get install python-argparse


### lftp

Evidentemente es necesario la instalación de este programa, pues es el que hace 
el trabajo sucio.

En Linux es realmente sencillo de instalarlo, pues viene en los repositorios de 
las distribuciones más importantes. En Debian/Ubuntu:

    $ sudo apt-get install lftp


## INSTRUCCIONES

Este es un script pensado para trabajar en la linea de comandos, dada la 
naturaleza de su función, que no es otra que automatizar un proceso que una vez 
lanzado no requiere de más intervención por nuestra parte.

Si ejecutáramos el programa sin argumentos nos encontraríamos con un mensaje de 
error:

    $ python lftp_mirror.py

    usage: lftp_mirror.py [-h] [-v] {cron,cfg,shell} ...
    lftp_mirror.py: error: too few arguments

Qué nos está diciendo que para emplearlo necesitamos como mínimo alguno de estos 
argumentos (`cron`, `cfg`  o `shell`) u opciones (`-h` ó `-v`). 

Estos tres argumentos serán los que definan el modo de ejecución del script, 
como comentábamos en la introducción.

### cron

En este modo se ejecuta tomando como parámetros los incluidos dentro del propio 
script. Este modo es útil para cuando se realiza una sincronización periódica 
sobre un solo servidor FTP/directorio. 

Para ejecutarlo nada más sencillo que localizar un bloque similar a este dentro 
del script y modificar las entradas que se encuentran entre comillas, 
sustituyendo los valores por defecto por los valores que necesitemos.

    #===============================================================================
    # SCRIPT PARAMATERS TO EXECUTE THE SCRIPT AS A PROGRAMMED TASK
    #===============================================================================

        # ftp user name ('user' by default)
        cron_user = 'user'
        # ftp password, with a minimum security measure, encoded by base64
        # ('password' by default)
        cron_pass = 'cGFzc3dvcmQ='
        # ftp server, name or ip ('localhost' by default)
        cron_site = 'localhost'
        # ftp server port. ('' by default)
        cron_port = ''
        # ftp directory
        cron_remote = 'directory'
        # local directory
        cron_local = '/your/path/to/your/local/directory/'
        # options, same as the shell mode. See shell mode help for more info
        cron_options = ''

    #===============================================================================
    # END PARAMETERS
    #===============================================================================

Finalmente solo tenemos que añadir a las tareas programadas (cron) esta entrada:

    lftp_mirror.py cron


### cfg

Este modo es parecido al anterior, con la diferencia de que los parámetros de 
ejecución se toman desde un fichero externo y que permite la ejecución 
consecutiva de varias operaciones de sincronización. Esto es ideal para ejecutar
en una sola operación la sincronización de varios servidores de FTP/directorios.
Evidentemente este modo también podría ser ejecutado periódicamente a través de 
cron.
 
Para utilizar este modo solo es necesario crear un fichero de texto con una 
estructura determinada. Se incluye un fichero de configuración de ejemplo como 
plantilla para crear los nuestros, sample.cfg. Este fichero no necesita 
necesariamente esa extensión, aunque nos sirve para identificarlo.

La estructura es la siguiente:

    [section]
    site = {ftp server URL or IP}
    port = (ftp server port)
    remote = {remote directory}
    local = {local directory}
    user = (ftp server username)
    password = (user password encoded in base64)
    options = (other options)

Donde *section* es un identificador de la operación de sincronización. 
Normalmente seria el nombre del servidor FTP o directorio a sincronizar. 

Los valores encerrados entre paréntesis son opcionales, mientras que los 
encerrados entre llaves son obligatorios. En el caso de que no especifiquemos un
usuario y una contraseña si es necesario añadir la opción *-a* que especifica 
que la conexión se realiza con el usuario anónimo.

Es necesario crear una sección como esta por cada operación de sincronización 
que deseemos realizar. 

Un ejemplo real se puede ver en el contenido de sample.cfg: 

    [debian]
    site = ftp.debian.org
    port =
    remote = /debian/doc
    local = debian
    user = 
    password = 
    options = -aenP --exclude-glob '*.txt' --include-glob 'social*'

    [FreBSD]
    site = ftp.freebsd.org
    port =
    remote = /pub/FreeBSD/ERRATA
    local = FreeBSD
    user = 
    password =
    options = -aenP
     
En este ejemplo real se ejecutarían las dos operaciones de forma consecutiva,
primero sincronizaría con el servidor de Debian y al finalizar seguiría con el 
servidor de FreeBSD. 

Para ejecutar las operaciones contenidas en este fichero de configuración 
usaríamos este comando:

    python lftp_mirror.py cfg sample.cfg


### shell

Este modo se utiliza para realizar una operación de sincronización especificando
los argumentos directamente en la linea de comandos. Es útil para 
sincronizaciones puntuales.

Para unas instrucciones completas, mejor acudir a la ayuda del script:

    $ python lftp_mirror.py shell -h

Aunque se podría resumir en esta linea de comandos:

    $ python lftp_mirror.py shell site remote local -l user password [options]

donde *site* es el servidor FTP, *remote* el directorio remoto y *local* el 
directorio local. *user* y *password* son usuario y contraseña del servidor FTP.
El resto de las opciones es extenso, por lo cual es mejor recurrir a la ayuda 
del script.

El conjunto de argumentos y opciones se detallan a continuación: 

#### Argumentos del modo shell:

* `site`

 El servidor FTP, se puede indicar como una URL o como una dirección IP

* `remote`

 El directorio remoto en el servidor FTP

* `local` 

 El directorio local en nuestro equipo

#### Opciones de lftp disponibles en el modo shell:

* `-h, --help`

 Muestra la ayuda de este modo

* `-l user password, --login user password`

 El usuario y la contraseña del servidor FTP

* `-a, --anon`

 Establece una conexión anónima con el servidor

* `-p port, --port port`

 Para especificar un puerto diferente al estándar de FTP (21)

* `-s, --secure`

 Establece una conexión segura empleando SFTP en lugar de FTP

* `-e, --erase`
 
 Borra en destino los ficheros que ya no se encuentran disponibles en origen

* `-n, --newer`
 
 Descarga únicamente los archivos nuevos

* `-P [N], --parallel [N]`

 Descarga N archivos en paralelo, empleando varias conexiones FTP al mismo tiempo. N=2 si no se especifica ningún valor

* `-r, --reverse`

 Modo inverso. En vez de descargar los archivos desde el directorio remoto al local, sube los archivos desde el local al servidor FTP

* `--delete-first`

 Antes de realizar la transferencia de los ficheros nuevos, elimina los antiguos

* `--depth-first`

 Antes de realizar la transferencia de ficheros, recorre todos los subdirectorios

* `--no-empty-dirs`

 No crea en destino directorios vacíos que puedan existir en origen. Requiere que sea usada la opción `--depth-first`

* `--no-recursion`

 No entra dentro de los subdirectorios

* `--dry-run`

 Simulación. No ejecuta ninguna operación, pero realiza una simulación y lo registra en el log

* `--use-cache`

 Emplea cache para conocer la estructura de subdirectorios

* `--del-source`

 Elimina archivos (que no directorios) en origen después de ser transferidos. **EMPLEAR CON CUIDADO**

* `--only-missing`

 Transfiere solo los ficheros que por alguna razón hayan desaparecido en el destino

* `--only-existing`

 Transfiere únicamente los ficheros que ya existan en destino

* `--loop`

 Se ejecuta continuamente en bucle hasta que no existan diferencias entre origen y destino

* `-ignore-size`

 Ignora el tamaño de los archivos a la hora de decidir que ficheros bajar y cuales no

* `--ignore-time`

 Ignora la fecha de los archivos a la hora de decidir que ficheros bajar y cuales no

* `--no-perms`
  
 No conserva los permisos en los ficheros

* `--no-umask`
 
 No aplica umask a los ficheros

* `--no-symlinks`

 No crea enlaces simbólicos

* `--allow-suid`

 Establece los bits suid/sgid de acuerdo al destino

* `--allow-chown`
 
 Intenta establecer el propietario y el grupo en los ficheros

* `--dereference`

 Descarga los enlaces simbólicos como ficheros

* `--exclude-glob GP`

 Excluye a los ficheros que coincidan con el patrón. Donde GP es un patrón de tipo glob, como por ejemplo: `*.zip`

* `--include-glob GP`

 Incluye a los ficheros que coincidan con el patrón. Donde GP es un patrón de tipo glob, como por ejemplo: `*.zip`. Empleado para ficheros que hayan sido excluidos por la opción anterior, pero que queramos establecer como excepciones


#### Opciones propias del script y no presentes en lftp:

* `-q, --quiet`

 Esta opción nos permite especificar que la salida detallada del 
proceso de sincronización no se visualizara por la linea de comandos y será 
añadida al registro de actividad (tanto en el fichero como en el correo).  

* `--no-compress` 

 Con esta opción desactivamos las copias de seguridad comprimidas 
del directorio local. 

* `--no-email` 

 Desactiva el envío del correo con el registro de actividad.

El correo se envía por defecto al usuario local que ejecuta el script, usando 
para ello el servidor de correo local. si se quiere emplear un servidor de 
correo diferente o enviárselo a otro(s) destinatario(s), entonces es necesario 
emplear las siguientes opciones:

* `--smtp_server` 

 El servidor de correo que deseamos emplear

* `--smtp_user` 

 El usuario del servidor de correo      

* `--smtp_pass` 

 La contraseña de ese usuario

* `--from_addr` 

 La dirección de email que ha de constar como remitente

* `--to_addrs email` 

 La(s) dirección(es) de correo de a quien(es) queremos enviar el correo


## REPOSITORIO

El código está alojado en un repositorio de Mercurial (hg) en BitBucket, así que
 puedes clonarlo directamente así:

    hg clone http://bitbucket.org/joedicastro/lftp-mirror

También está alojado en un repositorio Git en GitHub, emplea este comando para
poder clonarlo:

    git clone git://github.com/joedicastro/lftp-mirror.git


## CARACTERÍSTICAS

Cada vez que finaliza una ejecución añade el registro del actividad al fichero 
de log que se crea en la carpeta raíz del directorio local y si no se indica lo 
contrario envía un correo con el mismo contenido al usuario local. Un ejemplo de
este registro de actividad se puede ver en este correo electrónico que se envía 
después de ejecutar el fichero de configuración de ejemplo, sample.cfg:


    De:        youruser@yourcomputer
    Para:      youruser@yourcomputer
    Asunto:    FTP Sync - miércoles 08/12/10, 13:52:13
    Fecha:     Wed, 08 Dec 2010 13:52:13 +0100


    SCRIPT =========================================================================
    lftp_mirror (ver. 0.7)
    http://joedicastro.com

    Connected to ftp.debian.org as anonymous
    Mirror /debian/doc to debian
    ================================================================================

    START TIME =====================================================================
                                                        miércoles 08/12/10, 13:51:58
    ================================================================================

    CREATED NEW DIRECTORY __________________________________________________________

    debian


    LFTP OUTPUT ____________________________________________________________________

    Enviando archivo `00-INDEX'
    Enviando archivo `debian-manifesto'
    Enviando archivo `social-contract.txt'
    Replicando directorio `FAQ'
    Creando directorio `FAQ'
    Replicando directorio `dedication'
    Creando directorio `dedication'
    Enviando archivo `FAQ/debian-faq.en.html.tar.gz'
    Enviando archivo `FAQ/debian-faq.en.pdf.gz'
    Enviando archivo `dedication/dedication-2.2.sigs.tar.gz'
    Enviando archivo `FAQ/debian-faq.en.ps.gz'
    Enviando archivo `dedication/dedication-5.0.sigs.tar.gz'
    Enviando archivo `FAQ/debian-faq.en.txt.gz'



    ROTATE COMPRESSED COPIES _______________________________________________________

    Created file:

    /your/path/debian_08dic2010_13:52_mie.tar.gz


    DISK SPACE USED ================================================================
                                                                            1.35 MiB
    ================================================================================

    END TIME =======================================================================
                                                        miércoles 08/12/10, 13:52:06
    ================================================================================





    SCRIPT =========================================================================
    lftp_mirror (ver. 0.7)
    http://joedicastro.com

    Connected to ftp.freebsd.org as anonymous
    Mirror /pub/FreeBSD/ERRATA to FreeBSD
    ================================================================================

    START TIME =====================================================================
                                                        miércoles 08/12/10, 13:52:06
    ================================================================================

    CREATED NEW DIRECTORY __________________________________________________________

    FreeBSD


    LFTP OUTPUT ____________________________________________________________________

    Replicando directorio `notices'
    Replicando directorio `patches'
    Creando directorio `patches'
    Creando directorio `notices'
    Replicando directorio `patches/EN-04:01'
    Creando directorio `patches/EN-04:01'
    Enviando archivo `notices/FreeBSD-EN-04:01.twe.asc'
    Enviando archivo `notices/FreeBSD-EN-05:01.nfs.asc'
    Enviando archivo `patches/EN-04:01/twe.patch'
    Enviando archivo `notices/FreeBSD-EN-05:02.sk.asc'
    Enviando archivo `notices/FreeBSD-EN-05:03.ipi.asc'
    Enviando archivo `patches/EN-04:01/twe.patch.asc'
    Enviando archivo `notices/FreeBSD-EN-05:04.nfs.asc'
    Enviando archivo `notices/FreeBSD-EN-06:01.jail.asc'
    Enviando archivo `notices/FreeBSD-EN-06:02.net.asc'



    ROTATE COMPRESSED COPIES _______________________________________________________

    Created file:

    /your/path/FreeBSD_08dic2010_13:52_mie.tar.gz


    DISK SPACE USED ================================================================
                                                                           38.18 KiB
    ================================================================================

    END TIME =======================================================================
                                                        miércoles 08/12/10, 13:52:13
    ================================================================================


## ALTERNATIVAS 

Si mi script no encaja con lo que quieres, aquí tienes un resumen de alternativas para plataformas UNIX/Linux (las que conozco). Adjunto la mía como referencia.

* __lftp-mirror__

  - Lenguaje: Python
  - Tipo: script
  - Características: Las mencionadas arriba
  - Licencia: GPLv3
  - Autor(es): Yo 

* __[lftp](http://lftp.yar.ru)__

  - Lenguaje: C++
  - Tipo: Aplicación linea comandos
  - Características: Ligero, rápido y potente. Quizá el mejor cliente ftp disponible para linea de comandos. Lleno de opciones y muy versátil
  - Licencia: GPLv3
  - Autor(es): Alexander V. Lukyanov

* __[wget -m](http://www.gnu.org/software/wget)__

  - Lenguaje: C
  - Tipo: Aplicación linea comandos
  - Características: Solo funciona en una dirección: remoto a local
  - Licencia: GPLv3
  - Autor(es): Hrvoje Nikšić, Mauro Tortonesi, Steven Schubiger, Micah Cowan, Giuseppe Scrivano

* __[csync](http://www.csync.org)__

  - Lenguaje: C
  - Tipo: Aplicación linea comandos
  - Características: Bidireccional pero solo trabaja con sftp. No tan configurable como lftp
  - Licencia: GPLv2
  - Autor(es): Andreas Schneider

* __[weex](http://weex.sourceforge.net/)__

  - Lenguaje: C
  - Tipo: Aplicación linea comandos
  - Características: Solo funciona en una dirección: local a remoto
  - Licencia: GPLv2 
  - Autor(es): Yuuki Ninomiya, Ludovic Drolez

* __[ftpsync](http://sourceforge.net/projects/ftpsync)__

  - Lenguaje: Perl
  - Tipo: script
  - Características: Bidireccional, no soporta sftp. Sin tantas opciones como lftp
  - Licencia: GPLv2
  - Autor(es): Christoph Lechleitner

* __[ncftp](http://www.ncftp.com/ncftp)__

  - Lenguaje: C
  - Tipo: Aplicación linea comandos
  - Características: Bidireccional. Un poco lioso.
  - Licencia: Clarified Artistic License
  - Autor(es): Mike Gleason

* __[curlftpfs](http://curlftpfs.sourceforge.net) + [rsync](http://rsync.samba.org)__

  - Lenguaje: C
  - Tipo: Aplicación linea comandos
  - Características: Se monta un sistema de archivos con curlftpfs en local apuntando al servidor FTP y luego se emplea rsync para hacer la sincronización
  - Licencia: GPLv2 (curlftpfs) & GPLv3 (rsync)
  - Autor(es): Robson Braga Araujo (curlftpfs) & Andrew Tridgell, Paul Mackerras, Wayne Davison (rsync)


## CONTRIBUCIONES

Las contribuciones y las ideas son bienvenidas. Para contribuir a la mejora y 
evolución de este script, puedes enviar sugerencias o errores a través de el
sistema de issues.

## LICENCIA

Este script están sujeto a la [Licencia GPLv3 ](http://www.gnu.org/licenses/gpl.html)
