# Practica_Instalar_3_SGBD

**1. Características de la VM y versiones instaladas de Oracle 23ai Free, PostgreSQL y MariaDB.**
   La maquina virtual creada en virtualbox posee un sistema operativo Linux Mint de 4 GB de RAM, 2 CPUS y 50 GB de almacenamiento. Mientras que las versiones instaladas de cada Sistema Gestor de Base de Datos son:
   
| SGBD | Versión |
| --- | --- |
| Oracle 23ai Free | 23.5.0.0 |
| PostgreSQL | 16.15-0ubuntu0.24.04.1 |
| MariaDB | 15.1 Distribución 10.11.14-MariaDB |

**2. Pasos y comandos que realmente ejecutaste, explicados con tus palabras.**

Para preparar Podman y descargar la imagen ejecute los siguientes comandos
```
sudo apt update
sudo apt install podman
sudo podman pull container-registry.oracle.com/database/free:23.5.0.0
sudo podman images
```

Al crear y comprobar el contenedor, podremos ver que el volumen con nombre conserva los datos si se para o se recrea el contenedor. Recuerda, el se publica el puerto 1521 de la VM; usa una red de prácticas y comprueba el cortafuegos.

```
sudo podman volume create oracle_23ai_datos
sudo podman run -d --name cont-oracle -p 1521:1521 --restart=always -v oracle_23ai_datos:/opt/oracle/oradata:U container-registry.oracle.com/database/free:23.5.0.0
sudo podman pssudo podman ps -a
sudo podman logs cont-oracle
```

Entrar en Oracle y crear el usuario de trabajo
```
sudo podman exec -it cont-oracle /bin/bashexit
sudo podman exec -it cont-oracle sqlplus / as sysdba
```

En SQL>, comprueba la PDB y crea el usuario dentro de FREEPDB1:
```
SHOW PDBSALTER SESSION SET CONTAINER = FREEPDB1;
CREATE USER orauser IDENTIFIED BY "TU_CLAVE";
GRANT CREATE SESSION, CREATE TABLE TO orauser;
ALTER USER orauser QUOTA 100M ON USERS;
EXIT
```
sudo podman exec -it cont-oracle sqlplus orauser@//localhost:1521/FREEPDB1
2. Desplegar PostgreSQL
```
sudo apt update
sudo apt install postgresql postgresql-contrib php-pgsql
sudo systemctl start postgresql
sudo systemctl enable postgresql
sudo -i -u postgrespsql
```
En psql:
```
CREATE DATABASE dbpg;
CREATE USER pguser WITH PASSWORD 'TU_CLAVE';
GRANT ALL PRIVILEGES ON DATABASE dbpg TO pguser;
\q
```
Sal de la sesión del usuario postgres y comprueba la conexión:
```
exitpsql -h localhost -U pguser -d dbpg -W
```
3. Desplegar MariaDB
```
sudo apt update
sudo apt install mariadb-server mariadb-client
sudo systemctl enable --now mariadb
sudo mariadb
```
En el indicador de MariaDB:
```
CREATE DATABASE dbm CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'muser'@'localhost' IDENTIFIED BY 'TU_CLAVE';
GRANT ALL PRIVILEGES ON dbm.* TO 'muser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
El original crea 'muser'@'localhost' pero concede permisos a 'muser'@'%': no son la misma cuenta. Aquí se mantiene el mismo origen en ambas instrucciones. Comprueba el acceso local:
```
mariadb -u muser -p dbm
```

**3. Cómo comprobaste el estado de los tres gestores y la conexión con orauser, pguser y muser.**
Para comprobar la conexión, decidí conectar dos maquinas (la de Linux Mint y la maquina cliente, una Linux Server 24.04) mediante adaptador puente, tras modificar el modo de red en la VM y asignar la IP del equipo a los SGBD mediante los siguientes comandos:
Para modificar la conexion en PostgreSQL (en el archivo /etc/postgresql/XX/main/postgresql.conf, donde XX es la versión):
listen_addresses = 'IP_DE_LA_VM'

Tambien, en pg_hba.conf en el mismo directorio, añadiremos una regla limitada a la base, usuario y la IP que tiene la Virtual:
```
host dbpg pguser 'IP_DE_LA_VM'/24 scram-sha-256
```

Guarda los dos archivos de configuración y reinicia PostgreSQL, teniendo UFW activo, probaremos desde otra máquina:
```
sudo ufw allow from 192.168.1.0/24 to any port 5432 proto tcppsql -h IP_DE_LA_VM -U pguser -d dbpg -W
```

MariaDB
Busca el archivo de configuración efectivo, normalmente /etc/mysql/my.cnf o /etc/mysql/mariadb.conf.d/50-server.cnf. En la sección [mysqld] cambia bind-address para escuchar en la IP de la VM:
```
bind-address = IP_DE_LA_VM
```

Ejecuta estas instrucciones desde sudo mariadb:
```
CREATE USER 'muser'@'IP_DEL_CLIENTE' 
IDENTIFIED BY 'TU_CLAVE';
GRANT ALL PRIVILEGES ON dbm.* TO 'muser'@'IP_DEL_CLIENTE';
FLUSH PRIVILEGES;
EXIT;
```

Vuelve a la terminal y reinicia MariaDB:
```
sudo systemctl restart mariadb
```

Si UFW está activo, permite solo esa IP en el puerto 3306. Prueba desde el cliente:
```
sudo ufw allow from IP_DEL_CLIENTE to any port 3306 proto tcpmariadb -h IP_DE_LA_VM -u muser -p dbm
```

**4. Configuración de red realizada, si habilitaste conexiones desde otra máquina.**
Para hacer la conexion entre maquinas, decidi habilitar una conexión mediante adaptador puente, Mint tiene una ip 192.168.0.77 y la maquina a la que me conecte (Un linux server con la ip 192.168.0.33)

**5. Problemas encontrados, mensajes de error relevantes y cómo los resolviste; si no hubo incidencias, indícalo.**
No hubo incidencias tan criticas como se podrían esperar, pero, un detalle fue que cancele accidentalmente la creación del contenedor durante una ronda de prueba, llevando a tener que realizar el proyecto desde el comienzo.

**6. Fuentes consultadas y, si aportas capturas o salidas de comandos, una breve explicación de lo que demuestran.**
- [Información Podman](https://docs.podman.io/en/latest/)
- [Información postgresql](https://www.postgresql.org/docs/16/index.html)
- [Información mariadb](https://mariadb.com/docs/server/mariadb-quickstart-guides/mariadb-remote-connection-guide)

**EVIDENCIAS**
[Acceso a la carpeta de evidencias](evidencias)
