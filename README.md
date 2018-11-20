# Practica04

# Creación de una segunda máquina virtual para Apache.

La arquitectura estará formada por:

* Una capa front-end, formada por 2 servidores web con Apache HTTP server -
* Una capa back-end, formada por un servidor MySQL.

Configuración IP de las máquinas:

* Frontal Web 1 --> 192.168.33.11
* Frontal Web 2 --> 192.168.33.12
* Mysql --> 192.168.33.13

## VagrantFile:
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/xenial64"

  # Apache 1
  config.vm.define "web1" do |app|
    app.vm.hostname = "web1"
    app.vm.network "private_network", ip: "192.168.33.10"
    app.vm.provision "shell", path: "apache.sh"
  end

  # Apache 2
  config.vm.define "web2" do |app|
    app.vm.hostname = "web2"
    app.vm.network "private_network", ip: "192.168.33.11"
    app.vm.provision "shell", path: "apache.sh"
  end

  # MySQL
  config.vm.define "db" do |app|
    app.vm.hostname = "db"
    app.vm.network "private_network", ip: "192.168.33.12"
    app.vm.provision "shell", path: "mysql.sh"
  end

end
```
## Scripts.

Tendremos que crear 2 scripts: uno para Apache y otro para Mysql.

### Script de Apache:

```#!/bin/bash

# instalación de apache
apt-get update
apt-get install -y apache2
apt-get install -y php libapache2-mod-php php-mysql
sudo /etc/init.d/apache2 restart


#clonar repositorios
apt-get install -y git
cd /tmp
rm -rf iaw-practica-lamp 
git clone https://github.com/josejuansanchez/iaw-practica-lamp.git

#copiar repositorio
cd iaw-practica-lamp
cp src/* /var/www/html/

#modificar la base de datos que queremos usar
sed -i 's/localhost/192.168.33.12/' /var/www/html/config.php
chown www-data:www-data /var/www/html/* -R

#borramos el index
rm -rf /var/www/html/index.html
```

### Script para MySQL:

```
#!/bin/bash

# Actualización e instalación de paquetes y repositorios.
apt-get update
apt-get -y install debconf-utils

DB_ROOT_PASSWD=root
debconf-set-selections <<< "mysql-server mysql-server/root_password password $DB_ROOT_PASSWD"
debconf-set-selections <<< "mysql-server mysql-server/root_password_again password $DB_ROOT_PASSWD"

# Instalación de MySQL.
apt-get install -y mysql-server
sed -i -e 's/127.0.0.1/0.0.0.0/' /etc/mysql/mysql.conf.d/mysqld.cnf
/etc/init.d/mysql restart

mysql -uroot mysql -p$DB_ROOT_PASSWD <<< "GRANT ALL PRIVILEGES ON *.* TO root@'%' IDENTIFIED BY '$DB_ROOT_PASSWD'; FLUSH PRIVILEGES;"

# Clonación de repositorios.
apt-get install -y git
cd /tmp
rm -rf iaw-practica-lamp 
git clone https://github.com/josejuansanchez/iaw-practica-lamp.git

# Creación de la base de datos.
mysql -u root -p$DB_ROOT_PASSWD < /tmp/iaw-practica-lamp/db/database.sql
/etc/init.d/mysql restart
```