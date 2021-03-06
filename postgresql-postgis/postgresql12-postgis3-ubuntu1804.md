# Instalación de PostgreSQL 12 y PostGIS 3 en Ubuntu 18.04

## Documentación
* Página oficial de PostgreSQL: [PostgreSQL: The World's Most Advanced Open Source Relational Database](https://www.postgresql.org/)
* Página oficial de PostGIS: [PostGIS - Spatial and Geographic objects for PostgreSQL](https://postgis.net/)
* Procedimientos para la instalación de PostgreSQL y PostGIS: [install_qgis_postgis_linux](https://github.com/qgises/install_qgis_postgis_linux/blob/master/Install_QGIS_POSTGIS.sh)
* Guía para la instalación de PostgreSQL 12 en Ubuntu 16.04/Ubuntu 18.04: [How To Install PostgreSQL 12 on Ubuntu 18.04 / Ubuntu 16.04](https://computingforgeeks.com/install-postgresql-12-on-ubuntu/)
* Guía para la instalación de PostGIS: [UsersWikiPostGIS24UbuntuPGSQL10Apt](https://trac.osgeo.org/postgis/wiki/UsersWikiPostGIS24UbuntuPGSQL10Apt)

## Actualización del sistema
```terminal
$ sudo apt update
$ sudo apt upgrade -y
$ sudo reboot
```

## Adición del repositorio de PostgreSQL 12
Importación de la llave GPG
```terminal
$ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
```

Adición del repositorio
```terminal
$ echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
```

Actualización
```terminal
$ sudo apt update
```

## Revisión de las versiones disponibles
```terminal
$ sudo apt-cache madison postgresql postgresql-contrib postgis postgresql-12-postgis-3-scripts
```
```terminal
postgresql | 12+210.pgdg18.04+1 | http://apt.postgresql.org/pub/repos/apt bionic-pgdg/main amd64 Packages
postgresql | 10+190ubuntu0.1 | http://mirrors.digitalocean.com/ubuntu bionic-updates/main amd64 Packages
postgresql | 10+190ubuntu0.1 | http://security.ubuntu.com/ubuntu bionic-security/main amd64 Packages
postgresql |     10+190 | http://mirrors.digitalocean.com/ubuntu bionic/main amd64 Packages
postgresql-contrib | 12+210.pgdg18.04+1 | http://apt.postgresql.org/pub/repos/apt bionic-pgdg/main amd64 Packages
postgresql-contrib | 10+190ubuntu0.1 | http://mirrors.digitalocean.com/ubuntu bionic-updates/main amd64 Packages
postgresql-contrib | 10+190ubuntu0.1 | http://security.ubuntu.com/ubuntu bionic-security/main amd64 Packages
postgresql-contrib |     10+190 | http://mirrors.digitalocean.com/ubuntu bionic/main amd64 Packages
   postgis | 3.0.0+dfsg-2~exp1.pgdg18.04+1 | http://apt.postgresql.org/pub/repos/apt bionic-pgdg/main amd64 Packages
   postgis | 2.4.3+dfsg-4 | http://mirrors.digitalocean.com/ubuntu bionic/universe amd64 Packages
postgresql-12-postgis-3-scripts | 3.0.0+dfsg-2~exp1.pgdg18.04+1 | http://apt.postgresql.org/pub/repos/apt bionic-pgdg/main amd64 Packages
```

## Instalación
```terminal
$ sudo apt update -y
$ sudo apt install -y postgresql postgresql-contrib postgis postgresql-12-postgis-3-scripts
```

## Revisión del servicio postgresql
```terminal
$ systemctl status postgresql.service
```
```terminal
● postgresql.service - PostgreSQL RDBMS
   Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
   Active: active (exited) since Fri 2019-11-15 07:28:12 UTC; 56s ago
 Main PID: 3413 (code=exited, status=0/SUCCESS)
    Tasks: 0 (limit: 4915)
   CGroup: /system.slice/postgresql.service
```

```terminal
$ systemctl status postgresql@12-main.service
```
```terminal
● postgresql@12-main.service - PostgreSQL Cluster 12-main
   Loaded: loaded (/lib/systemd/system/postgresql@.service; indirect; vendor preset: enabled)
   Active: active (running) since Fri 2019-11-15 07:28:17 UTC; 1min 56s ago
 Main PID: 4482 (postgres)
    Tasks: 7 (limit: 4915)
   CGroup: /system.slice/system-postgresql.slice/postgresql@12-main.service
           ├─4482 /usr/lib/postgresql/12/bin/postgres -D /var/lib/postgresql/12/main -c config_file=/etc/postgresql/12/main/postgresql.conf
           ├─4494 postgres: 12/main: checkpointer
           ├─4495 postgres: 12/main: background writer
           ├─4496 postgres: 12/main: walwriter
           ├─4498 postgres: 12/main: autovacuum launcher
           ├─4499 postgres: 12/main: stats collector
           └─4501 postgres: 12/main: logical replication launcher
```

```terminal
$ systemctl is-enabled postgresql
```
```terminal
enabled
```

## Modificación de archivos de configuración
Modificación de postgresql.conf
```terminal
$ sudo nano /etc/postgresql/12/main/postgresql.conf
```
```terminal
# Descomentar y cambiar la línea
listen_addresses = 'localhost'
# Por
listen_addresses = '*'
```

Modificación de pg_hba.conf
```terminal
$ sudo nano /etc/postgresql/12/main/pg_hba.conf
```
```terminal
# Añadir la siguiente línea (en la sección de IPv4) para dar acceso a todas las direcciones IP
host    all             all             0.0.0.0/0               md5
```

Reinicio del servicio postgresql
```terminal
$ sudo service postgresql restart
```

## Creación de un usuario y de extensiones de PostGIS
Ingreso a la interfaz de psql
```terminal
$ sudo -u postgres psql
```

Creación de un rol que incluye las extensiones postgis y postgis_topology
```terminal
postgres=# CREATE ROLE gisadmin login PASSWORD 'postgres' SUPERUSER CREATEDB CREATEROLE NOINHERIT;
postgres=# CREATE EXTENSION postgis;
postgres=# CREATE EXTENSION postgis_topology;
postgres=# \q
```

## Creación de una base de datos geoespacial
Ingreso a la interfaz de psql
```terminal
$ sudo -u postgres psql
```

Creación de una base de datos geoespacial, basada en el rol creado en la sección anterior
```terminal
postgres=# CREATE DATABASE geodb TEMPLATE postgres OWNER gisadmin;
postgres=# \q
```

## Carga de datos
Descarga de un shapefile
```terminal
$ wget https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/10m/cultural/ne_10m_admin_0_countries.zip
$ sudo apt install -y unzip
$ unzip ne_10m_admin_0_countries.zip
```

Carga del shapefile en la base de datos
```terminal
$ shp2pgsql -s 4326 ne_10m_admin_0_countries public.ne_10m_admin_0_countries | psql -h localhost -d geodb -U gisadmin
```

## Prueba de acceso a los datos
```terminal
$ sudo -u postgres psql -d geodb -c "SELECT name FROM ne_10m_admin_0_countries;"
```
