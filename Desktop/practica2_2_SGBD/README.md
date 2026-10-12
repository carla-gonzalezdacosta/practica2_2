#!/bin/bash

# Pr ctica: Instalar tres SGBD

## ASIR 2 - SGBD - Carla Gonz lez Da Costa

### 12/10/2026

# 1. Desplegar Oracle Database 23ai Free con Podman

## 1. He creado una VM nueva con sistema operativo Linux Mint 4GB RAM 2 CPUs y 250GB de disco:

`images/conf_vm.png`

## 2. Voy a descargar Podman:

- `sudo apt update`: actualiza paquetes y versiones.
- `sudo apt install podman`: descarga e instala podman.
- `sudo podman pull container-registry.oracle.com/database/free:23.5.0.0`: descarga la imagen desde la p gina oficial de oracle.
- `sudo podman images`: muestra todas las im genes de contenedores que se han descargado.

`images/primera_imagen.png`

La descarga de la imagen de Oracle se ha completado y ya puedo crear la instancia con ella.

## 3. Crear y comprobar el contenedor

- `sudo podman volume create oracle_23ai_datos`  crea un volumen de podman con nombre oracle_23ai_datos, y es donde oracle guardar  sus datos. Sirve para por si un d¡a se elimina el contenedor, que los datos de Oracle sigan guardados.
- `sudo podman run -d --name cont-oracle -p 1521:1521 --restart=always -v oracle_23ai_datos:/opt/oracle/oradata:U container-registry.oracle.com/database/free:23.5.0.0`  crea un contenedor nuevo con nombre 'cont-oracle', le indicamos tambi‚n el puerto de oracle 1521. Adem s indico que si el contenedor se para, que siempre le haga un restart 'restart=always'. Por otro lado tambi‚n le indico que los datos que guarde en la carpeta /opt/oracle/orabase, tambi‚n se guarden en el volumen oracle_23ai_datos. Por £ltimo, le indico la imagen de oracle que quiero utilizar.

- `sudo podman ps`
- `sudo podman ps -a`
- `sudo podman logs cont-oracle`

Con estos £ltimos comandos verificamos que el contenedor se ha creado correctamente y est  healthy, es decir, est  activo. He marcado en amarillo las partes m s importantes del comando y que nos indican que est  listo el contenedor.

`images/contenedor_healthy.png`

He cambiado la contrase¤a como indicas en las instrucciones, usando este comando: `sudo podman exec -it cont-oracle sqlplus / as sysdba`

A partir de ah¡ he entrado en Oracle y alterado SYSTEM:

```sql
ALTER USER SYSTEM IDENTIFIED BY "Contrase¤a";
```

## 4. Entrar en Oracle y crear el usuario de trabajo

- `sudo podman exec -it cont-oracle /bin/bash`  me aparece bash-4.4$ y significa que estoy dentro del contenedor Oracle.
- `exit`
- `sudo podman exec -it cont-oracle sqlplus / as sysdba`  con este comando entro en oracle, compruebo la PDB y creo el usuario.
  - `SHOW PDBS`

`images/segunda.png`

Hay dos PDB, una es la plantilla (speed) y la otra es la primera.

- `ALTER SESSION SET CONTAINER = FREEPDB1;`
- `CREATE USER orauser IDENTIFIED BY "TU_CLAVE";`
- `GRANT CREATE SESSION, CREATE TABLE TO orauser;`
- `ALTER USER orauser QUOTA 100M ON USERS;`
- `EXIT`

`images/tercera.png`

He alterado la sesi¢n FREEPDB1 para poder crear un usuario en esa base de datos, llamado orauser y le he asignado espacio de almacenamiento de 100MB para guardar datos como por ejemplo tablas, etc.

- `sudo podman exec -it cont-oracle sqlplus orauser@//localhost:1521/FREEPDB1`  compruebo la conexi¢n a ese usuario (orauser).

Hab¡a tenido una incidencia porque al crear el usuario no he introducido el comando de `CREATE TABLE TO orauser;` por lo tanto al intentar conectarme con ese usuario no me dejaba, pero ya lo he solucionado al entrar en esa base de datos y poner el comando, alterando la sesi¢n antes.

Ahora ya he conectado con el usuario orauser y estoy dentro de su base de datos.

`images/cuarta.png`

## 5. Desplegar PostgreSQL

En este paso vas a **instalar PostgreSQL en tu m quina/VM, crear una base de datos y un usuario, darle permisos y despu‚s comprobar que ese usuario puede conectarse**.

- `sudo apt update`
- `sudo apt install postgresql postgresql-contrib php-pgsql`
- `sudo systemctl start postgresql`
- `sudo systemctl status postgresql`

Una vez instalado postgresql comprobamos su estado y verificamos que est  activo.

`images/postgresql_active.png`

- `sudo systemctl enable postgresql`  hacer que postgresql se inicie autom ticamente cuando se encienda la m quina.
- `sudo -i -u postgres`  cambio de usuario de linux a postgres.
- `psql`  entro en postgresql desde ese usuario.

```sql
CREATE DATABASE dbpg;
CREATE USER pguser WITH PASSWORD 'TU_CLAVE';
GRANT ALL PRIVILEGES ON DATABASE dbpg TO pguser;
\q
```

He creado un nuevo usuario llamado pguser y le he asignado todos los privilegios.

`images/usuario_pguser.png`

- `psql -h localhost -U pguser -d dbpg -W`  usando el usuario `pguser`, entrar en la base de datos `dbpg` y que me pida la contrase¤a.

Comprobamos que hay conexi¢n.

`images/conexion_pguser.png`

## 6. Desplegar MariaDB

- `sudo apt update`
- `sudo apt install mariadb-server mariadb-client`  instala el servidor mariadb-server y las herramientas para conectarme y trabajar con mariadb 'mariadb-client'.
- `sudo systemctl enable --now mariadb`  hace que inicie mariadb autom ticamente cuando arranque ubuntu y adem s lo arranca ahora mismo tras el comando.
- `sudo mariadb`  entrar en mariadb como administrador.

Comprobamos que se ha instalado correctamente y que podemos entrar correctamente como administrador en mariadb.

`images/mariadb.png`

- `CREATE DATABASE dbm CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;`  creo una base de datos llamada dbm.
- `CREATE USER 'muser'@'localhost' IDENTIFIED BY 'TU_CLAVE';`  creo al usuario muser y le digo que puede conectarse desde localhost.
- `GRANT ALL PRIVILEGES ON dbm.* TO 'muser'@'localhost';`  darle todos los permisos al usuario.
- `FLUSH PRIVILEGES;`  gestiona permisos.
- `EXIT;`

Despu‚s de haber creado el usuario compruebo que puedo conectarme a la base de datos dbm.

`images/usuario_mariadb.png`

## 7. Resumen y comprobaciones

El objetivo de este ejercicio es verificar si podemos acceder a cada SGBD y a la base de datos Oracle, a trav‚s del servidor Apache y si primero podemos escuchar los puertos de cada SGBD.

Para ello primero vamos a comprobar los puertos TCP con el comando: `nc -zv localhost 'Puerto'`

Posteriormente, nos conectamos a los gestores mediante el comando: `'sgbd' -h localhost -P 'puerto' -u 'usuario' -p 'base de datos'`

### 1. Primero voy a instalar el servicio nc:

- `sudo apt update`
- `sudo apt install netcat-openbsd`

### 2. Pruebo los puertos: `nc -zv localhost 'Puerto'`

He podido establecer conexi¢n con cada puerto porque en los tres aparece `Connection to localhost 'puerto' port [tcp/mysql] succeeded!`, lo que indica que ha sido correcta la conexi¢n.

`images/conexion_puertos.png`

### 3. Conectarse a los GSBD: `'sgbd' -h localhost -P 'puerto' -u 'usuario' -p 'base de datos'`

- `images/conexion_mariadb.png`
- `images/conexion_psql.png`
- Para comprobar si me puedo conectar a Oracle hay que usar un comando diferente ya que este est  en un contenedor: `sudo podman exec -it cont-oracle sqlplus orauser@//localhost:1521/FREEPDB1`

`images/conexion_oracle.png`

### Comprobaciones

| SGBD | puerto TCP | conexi¢n con cliente | resultado |
|---|---|---|---|
| Oracle 23ai Free | 1521 | SQL*Plus | hay conexi¢n |
| PostgreSQL | 5432 | psql | hay conexi¢n |
| MariaDB | 3306 | Cliente mariadb | hay conexi¢n |

Conclusi¢n: he podido conectarme a los tres puertos y a mariadb, sqlplus y oracle de forma correcta y sin incidencias.

## h) Conexiones desde otra m quina

En este apartado he comprobado si puedo conectarme desde una segunda m quina virtual a PostgreSQL y MariaDB instalados en la VM servidor. Para ello, he instalado los clientes de ambos gestores en la VM cliente.

### Comandos a utilizar

1. He decidido utilizar otra vm como cliente y es por eso que voy a a¤adir otra interfaz a mi VM de host-only.

### PostgreSQL

- `ip a`  verifico que me sale una interfaz llamada enp0s8 con IP 192.168.56.101.
- `ping 192.168.56.102`  compruebo que tengo conexi¢n con la nueva vm.
- Ahora voy a modificar el archivo de configuraci¢n de postgresql para que acepte conexiones por la IP de mi VM: `sudo nano /etc/postgresql/*/main/postgresql.conf`

- Abro el archivo de permisos: `sudo nano /etc/postgresql/*/main/pg_hba.conf`

Al final de archivo a¤ado esta frase:

```text
host    dbpg    pguser    192.168.56.101/32    scram-sha-256
```

Esta l¡nea permite al usuario `pguser` conectarse a `dbpg` desde esa direcci¢n, usando autenticaci¢n por contrase¤a.

- Compruebo que PostgreSQL est  activo y compruebo que mi usuario puede conectarse desde el host (cliente).

Para ello he a¤adido en el fichero de permisos la l¡nea:

```text
host    dbpg    pguser    192.168.56.102/32    scram-sha-256
```

con la IP de mi host.

Y ahora ejecuto este comando para comprobar si se conecta al gestor:

```bash
psql -h 192.168.56.101 -U pguser -d dbpg -W
```

`images/conf_psql_nuevavm.png`

### MariaDB

- Abro el archivo de configuraci¢n: `sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf`
- Cambio la IP por la del equipo 192.168.56.102, para indicarle que escuche en la IP de ubuntu.

`images/windows_mariadb.png`

- `sudo mariadb`  creamos un usuario que pueda conectarse desde windows.

`images/conexion_mariadb.png`

- Instalo mariadb y psql en mi vm cliente: `sudo apt install postgresql-client mariadb-client`
- Ahora me conecto desde la VM cliente a PostgreSQL y MariaDB:
  - `psql -h 192.168.56.101 -U pguser -d dbpg -W`  cuando desde mi vm cliente me sale un error porque la configuraci¢n de mi m quina cliente no permite conexi¢n desde la direcci¢n IP 192.168.56.1.

`images/error_conexion_cliente.png`

Para solucionar el problema, voy a a¤adir esta l¡nea de comando:

```text
host    dbpg    pguser    192.168.56.1/32    scram-sha-256
```

al fichero: `sudo nano /etc/postgresql/*/main/pg_hba.conf`, lo que va a permitir a esa red acceder a la base de datos dbpg.

`images/arreglo_error.png`

Comprobamos que he conseguido entrar desde mi VM cliente a PostgreSQL.

`images/entrar_psql_cliente.png`

- Ahora voy a entrar en MariaDB  `sudo mariadb`.

He puesto este comando `mariadb -h 192.168.56.101 -u muser -p dbm` y no me ha funcionado porque me pon¡a error: ssl is required. Por ello, he buscado qu‚ significaba ese error y me ha indicado que deb¡a ejecutar este otro comando `mariadb --disable-ssl -h 192.168.56.101 -u muser -p dbm`, a¤ad¡ la opci¢n disable para comprobar si pod¡a acceder a mariadb sin utilizar ssl (conexi¢n cifrada) y ah¡ me funcion¢ correctamente y pude conectarme a MariaDB.

image/conexion_cliente_mariadb.png
