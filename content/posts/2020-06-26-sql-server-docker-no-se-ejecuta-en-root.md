---
title: "SQL Server en Docker que no se ejecuta como root"
author: eiximenis
description: "Desde hace algún tiempo muchas imágenes ya no ejecutan su proceso como root dentro del contenedor. Esto puede generar algunos problemillas de permisos, especialmente al usar volúmenes."
date: 2020-06-26T18:00:00
draft: false
categories:
  - docker
---

Muchos contenedores, por motivos de seguridad, se están moviendo de ejecutar su proceso como `root` a hacerlo con otro usuario. Un ejemplo es SQL Server (`mcr.microsoft.com/mssql/server:2019-latest`). Eso te puede dar algunos problemas, especialmente si usas volúmenes, pero nada que, afortunadamente, no pueda arreglarse. ¡Vamos allá!

## Ejecutando contenedores con Docker for Windows desde WSL2 / Ejecutando contendores en Linux con Docker

Ambos escenarios son equivalentes: tanto si usas directamente Linux y Docker, como si usas Docker for Windows **con el backend de WSL2** y ejecutas los contenedores **desde la CLI de Docker en WSL2** (no en Windows).

En este caso, si ejecutas:

```
docker run -e ACCEPT_EULA=Y -e SA_PASSWORD=Pass2w0rd! -v ~/data/mssql:/var/opt/mssql mcr.microsoft.com/mssql/server:2019-latest
```

Seguramente obtendrás **el siguiente error**:

```
SQL Server 2019 will run as non-root by default.
This container is running as user mssql.
To learn more visit https://go.microsoft.com/fwlink/?linkid=2099216.
/opt/mssql/bin/sqlservr: Error: The system directory [/.system] could not be created.  Errno [13]
```

Observa como la segunda línea nos informa de que el usuario con el que se ejecuta el contenedor es `mssql`. El problema es, simplemente, que dicho usuario **no tiene permisos en la carpeta `~/data/mssql`**:

{{< terminal "eiximenis@locrestia" "~" >}}
$ ls ~/data -l
total 4
drwxr-xr-x 2 root root 4096 Jun 26 10:39 mssql
{{</terminal>}}

La carpeta `mssql` **pertence a root** y no a al usaurio `mssql` (que ni existe en mi máquina), así que cuando el contenedor intenta escribir en esa carpeta, obtiene el error de permiso denegado.

> Una solución rápida es dar permisos totales a la carpeta (`chmod 777 ~/data/mssql`) y listos, pero eso quizá no es lo mejor en cuanto a seguridad. Así, que veamos como podemos hacerlo manteniendo los permisos al mínimo.

Para ello lo suyo es dar la propiedad de la carpeta a este usuario `mssql` en mi máquina. No es necesario que el usuario exista, basta con que sepa su ID de usuario. Para ello puedo abrir una sesión con el contenedor:

```
docker run -it --entrypoint /bin/sh mcr.microsoft.com/mssql/server:2019-latest
```

Y el la sesión interactiva que se abre, teclear:

{{< terminal "user@mssql_container" "~" >}}
$ id mssql
uid=10001(mssql) gid=0(root) groups=0(root)
{{</terminal>}}

Me dice el id de usuario (`uid` que en el ejemplo `10001`) así como su grupo principal (`gid`) y todos los grupos en los que ese usuario esté. Lo que nos interesa aquí es su `uid`. Ahora simplemente debo crear esa carpeta en el sistema de ficheros **de mi máquina**, y cambiarle el propietario a `10001`:

{{< terminal "eiximenis@locrestia" "~" >}}
$ mkdir ~/data
$ mkdir ~/data/mssql
$ sudo chown 10001 ~/data/mssql
$ ls ~/data -l
total 4
drwxr-xr-x 2 10001 eiximenis 4096 Jun 26 10:53 mssql
{{</terminal >}}

Como se puede observar ahora el propietario es el `10001` (no me sale `mssql` porque el usuario `mssql` en mi máquina no existe, pero como dije antes no es necesario). ¡Ahora el contenedor ya se tiene que poder levantar sin problemas!

## Ejecutando contenedores con Docker for Windows contra WSL2 desde Windows

Este escenario es distinto (y propio solo de Windows). Como antes usamos Docker for Windows contra el backend de WSL2 pero ahora levantamos los contenedores **desde la CLI de Windows** no desde la CLI de WSL2. Solo para que quede constancia es un escenario soportado, aunque no recomendado, por temas de rendimiento (desde WSL2 el acceso al sistema de ficheros de Windows no destaca por su rapidez). Así si tecleas (recuerda, desde Windows):

```
docker run -e ACCEPT_EULA=Y -e SA_PASSWORD=Pass2w0rd! -v %USERPROFILE%/data/mssql:/var/opt/mssql mcr.microsoft.com/mssql/server:2019-latest
```

El contenedor dará un error (al cabo de un ratito) con varios mensajes del estilo:

```
/usr/bin/find: '/proc/9/task/46/ns': Permission denied
```

Si investigas los permisos de la carpeta **desde una terminal de WSL2** el resultado es que el directorio tiene todos los permisos:

{{< terminal "eiximenis@locrestia" "~" >}}
$ ls /mnt/c/Users/etoma/data -l
total 0
drwxrwxrwx 1 eiximenis eiximenis 4096 Jun 23 11:25 mssql
{{</ terminal>}}

Esto es habitual cuando accedemos desde WSL2 al sistema de ficheros de Windows: El directorio se monta con **permisos totales**. La idea bajo esa aproximación es que **WSL2 no se encarga realmente de la seguridad**, si no que la difiere en Windows. Desde Linux (WSL2) se puede acceder al directorio, pero luego cuando se produce el acceso real, es Windows quien gestiona la seguridad. Por lo que, el error de acceso denegado es por parte de Windows, no de WSL2.

Tengo que reconocer que **no tengo claro por que no funciona en este caso** pero parece ser alguna limitación en Docker for Windows, y además [está documentada](https://docs.microsoft.com/en-us/sql/linux/sql-server-linux-configure-docker?view=sql-server-ver15#mount-a-host-directory-as-data-volume). Como se informa en ese enlace no se soporta montar todo el directorio `/var/opt/mssql` y debe montarse `/var/opt/mssql/data`. Algo raro debe hacer SQL Server por ahí xD

Así pues desde la CLI de Windows ese comando funciona correctamente:

```
docker run -e ACCEPT_EULA=Y -e SA_PASSWORD=Pass2w0rd! -v C:\Users\etoma/data/mssql:/var/opt/mssq/data mcr.microsoft.com/mssql/server:2019-latest
```

## Otro ejemplo: MySQL

MySQL es otro contenedor que también usa un usuario no root (en este caso es el `999`) para ejecutar su proceso. Pero **a diferencia de SQL Server, funciona directamente**:

```
docker run -v ~/data/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=Pass2w0rd! -e MYSQL_DATABASE=mydb -e MYSQL_USER=myuser  -e MYSQL_PASSWORD=Pass2w0rd! mysql:8.0.20
```

Y si lo miras, verás que el directorio `~/data/mysql` estará creado con el usuario `999`:

{{< terminal "eiximenis@locrestia" "~" >}}
$ ls ~/data -l
total 4
drwxr-xr-x 7 999 root 4096 Jun 26 12:20 mysql
{{< /terminal >}}

**Por qué MySQL funciona directamente y SQL Server no?**. 

Eso tiene que ver como está hecha la imagen. A pesar de que ambas imágenes (la de MySQL y la de SQL Server) ejecutan su proceso con un usuario que no es root, lo hacen de formas bastante distintas. SQL Server usa la instrucción `USER` del Dockerfile para establecer un usuario, pero lo hace al final (porque antes debe hacer varias tareas como `root`). Básicamente, en runtime, lo que la imagen de SQL Server hace es ejecutar el proceso de SQL Server con el usuario `mssql` y ya. Por su parte, la imagen de MySQL **se ejecuta como `root`** y lo que hace es lanzar un script que termina ejecutando el proceso de MySQL con el usuario `mysql`, pero ese script se está ejecutando como `root`. Eso lo puedes comprobar fácilmente:

![Terminales interactivas contra MySql y SQL Server](/images/posts/2020-06-26-sqlserver-mysql-users.png)

En la imagen puedes verificar que si abres un terminal interactivo contra el contenedor de SQL Server, el usuario es `mssql` mientras que en MySQL el usuario es `root`. La imagen de MySQL cuando se pone en marcha, se encarga de obtener y cambiar los permisos del directorio asociado del volumen (puede hacerlo porque la imagen es `root`). A nivel de seguridad la aproximación de SQL Server me gusta más, ya que la imagen de MySQL se podría explotar de algún modo y realizar acciones como `root` en el host. Un ejemplo:

![Borrar un fichero de root del host desde dentro del contenedor](/images/posts/2020-06-26-root-container-access.png)

Eso mismo, con la imagen de SQL Server no podrías hacerlo porque la imagen ya no se ejecuta como `root`. 

¡Un saludo!