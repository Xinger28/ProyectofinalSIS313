Configuración de los Servidores de Bases de Datos
VMs: DBServer1 (192.168.58.103 en Laptop 1, Maestro) y DBServer2 (192.168.58.105 en Laptop 2, Esclavo).
Herramienta: MariaDB.
4.1 Configuración del Maestro (DBServer1)
Instala MariaDB:
bash

Contraer

Ajuste

Ejecutar


sudo apt update && sudo apt install mariadb-server -y
sudo mysql_secure_installation
Configura una contraseña root segura.
Responde "Y" a todas las opciones de seguridad.
Configura la replicación:
bash

Contraer

Ajuste

Ejecutar

sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
Agrega en el bloque [mysqld]:
text

Contraer

Ajuste

bind-address = 0.0.0.0
server-id = 1
log_bin = /var/log/mysql/mariadb-bin
binlog_do_db = mydb
Reinicia MariaDB:
bash

Contraer

Ajuste

Ejecutar

sudo systemctl restart mariadb
Crea la base de datos y el usuario:
bash

Contraer

Ajuste

Ejecutar

sudo mysql -u root -p
En el prompt de MySQL:
sql

Contraer

Ajuste

CREATE DATABASE mydb;
USE mydb;
CREATE TABLE items (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255));
CREATE USER 'appuser'@'%' IDENTIFIED BY 'securepassword';
GRANT ALL PRIVILEGES ON mydb.* TO 'appuser'@'%';
CREATE USER 'replica'@'%' IDENTIFIED BY 'replicapass';
GRANT REPLICATION SLAVE ON *.* TO 'replica'@'%';
FLUSH PRIVILEGES;
SHOW MASTER STATUS;
Anota el File (ej. mariadb-bin.000001) y Position (ej. 1234).
4.2 Configuración del Esclavo (DBServer2)
Instala MariaDB como en el maestro.
Configura:
bash

Contraer

Ajuste

Ejecutar

sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
Agrega en el bloque [mysqld]:
text

Contraer

Ajuste


bind-address = 0.0.0.0
server-id = 2
relay_log = /var/log/mysql/relay-bin
Reinicia:
bash

Contraer

Ajuste

Ejecutar

sudo systemctl restart mariadb
Configura la replicación:
bash

Contraer

Ajuste

Ejecutar

sudo mysql -u root -p
En el prompt:
sql

Contraer

Ajuste

CHANGE MASTER TO 
    MASTER_HOST='192.168.58.103',
    MASTER_USER='replica',
    MASTER_PASSWORD='replicapass',
    MASTER_LOG_FILE='mariadb-bin.000001',
    MASTER_LOG_POS=1234;
START SLAVE;
SHOW SLAVE STATUS\G
