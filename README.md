CentOS 6
========

Acordeon para instalar, configurar y operar nuestro servidor CentOS 6.4

**Configuracion basica**

    emacs /root/.bashrc /root/.emacs

    cp /etc/sysconfig/network /etc/sysconfig/network.orig
    emacs /etc/sysconfig/network

    cp /etc/sysconfig/clock /etc/sysconfig/clock.orig
    emacs /etc/sysconfig/clock

    cp /usr/share/zoneinfo/Mexico/General /etc/localtime

**Utilerias y editores de texto**

    yum install yum-utils.x86_64 screen.x86_64 emacs-nox.x86_64 vim-minimal.x86_64

**Depositos EPEL y REMI**

    wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
    wget http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
    rpm -Uvh remi-release-6*.rpm epel-release-6*.rpm

    cp /etc/yum.repos.d/remi.repo /etc/yum.repos.d/remi.repo.orig
    emacs /etc/yum.repos.d/remi.repo

**Actualizar sistema**

    yum update

**Firewall**

    iptables -A INPUT -i lo -j ACCEPT
    iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
    iptables -A INPUT -p icmp --icmp-type 8 -m conntrack --ctstate NEW -j ACCEPT
    iptables -A INPUT -p tcp --dport 22 -j ACCEPT
    iptables -A INPUT -p tcp --dport 80 -j ACCEPT
    iptables -A INPUT -p tcp --dport 8000:8999 -j ACCEPT

    iptables -P INPUT DROP
    iptables -P FORWARD DROP
    iptables -P OUTPUT ACCEPT

    /etc/init.d/iptables save

**OpenSSH**

    cp /etc/ssh/sshd_config /etc/ssh/sshd_config.orig
    emacs /etc/ssh/sshd_config
    /etc/init.d/sshd restart

**PostgreSQL y PostGIS**

    wget http://yum.postgresql.org/9.3/redhat/rhel-6-x86_64/pgdg-centos93-9.3-1.noarch.rpm
    rpm -Uvh pgdg-centos93-*.rpm
    yum install postgis2_93.x86_64 postgresql93-server.x86_64 postgresql93-devel.x86_64 postgresql93.x86_64

    service postgresql-9.3 initdb
    chkconfig postgresql-9.3 on
    /etc/init.d/postgresql-9.3 start

**Configuración PostgreSQL**

    su - postgres -c psql
    \password
    \q

    cp /var/lib/pgsql/9.3/data/pg_hba.conf /var/lib/pgsql/9.3/data/pg_hba.conf.orig
    emacs /var/lib/pgsql/9.3/data/pg_hba.conf
    /etc/init.d/postgresql-9.3 restart

**Nginx y PHP-FPM**

    emacs /etc/yum.repos.d/nginx.repo
    yum install nginx.x86_64 php-fpm.x86_64

    cp /etc/php-fpm.d/www.conf /etc/php-fpm.d/www.conf.orig
    emacs /etc/php-fpm.d/www.conf

    cp /etc/php.ini /etc/php.ini.orig
    emacs /etc/php.ini

    mkdir /var/lib/php/session
    chown nginx:nginx /var/lib/php/session /var/log/nginx

    chkconfig nginx on
    /etc/init.d/nginx start
    chkconfig php-fpm on
    /etc/init.d/php-fpm start

**Extensiones PHP**

    yum install php-pear.noarch php-devel.x86_64 make.x86_64  php-pgsql.x86_64  php-gd.x86_64
    pecl install oauth

**NodeJS y NPM**
    
    yum install nodejs.x86_64 npm.noarch

**Git**

    yum install git.x86_64
    mkdir /srv/repos

**Ruby con RVM**

    groupadd rvm
    curl -L https://get.rvm.io | sudo bash -s stable
    source /etc/profile.d/rvm.sh
    source ~/.bashrc
    usermod -a -G rvm usuario
    rvm install 2.0.0
    
**Rails con RVM**

    gem install rails bundler
    
**Nginx y Passenger**

    gem install passenger
    rvmsudo passenger-install-nginx-module

**Sudo**

    cp /etc/sudoers /etc/sudoers.orig
    visudo

**Sitio codigo.labplc.mx**

    mkdir /srv/sites/codigo.labplc.mx
    mkdir /srv/sites/codigo.labplc.mx/web
    mkdir /srv/sites/codigo.labplc.mx/web/htdocs

    mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.orig
    emacs /etc/nginx/conf.d/default.conf

    echo 'codigo.labplc.mx' > /srv/sites/codigo.labplc.mx/web/htdocs/index.html
    echo '<?php phpinfo(); ?>' > /srv/sites/codigo.labplc.mx/web/htdocs/phpinfo.php

    /etc/init.d/nginx restart
    
**Instalar soporte para UUID postgresql**

    sudo yum install postgresql93-contrib
    
 En sql editor:

    CREATE EXTENSION "uuid-ossp";

**Crear usuario de shell**

    useradd -u 100 -g 100 -G wheel -s /bin/bash -m manuel
    passwd manuel

**Crear carpeta publica en usuario de shell**

    chmod 711 /home/manuel
    mkdir /home/manuel/public_html

**Crear rol en PostgreSQL para usuario**

    createuser -U postgres -s -P manuel

**Nuevo host virtual**

    mkdir /srv/sites/acopia.me
    mkdir /srv/sites/acopia.me/web
    mkdir /srv/sites/acopia.me/web/htdocs

**Crear rol en PostgreSQL para aplicación web**

    createuser -U postgres -I -P acopiame

**Crear base de datos para aplicación web**

    createdb -U postgres acopiame -O acopiame

**Aplicaciones Rails***

    mkdir /srv/sites/dev.codigo.labplc.mx
    mkdir /srv/sites/dev.codigo.labplc.mx/web
    mkdir /srv/sites/dev.codigo.labplc.mx/web/htdocs    
    emacs /etc/nginx/conf.d/rails.conf

**Crear deposito de Git local***

    groupadd -g 502 datos
    usermod -a -G 502 manuel
    mkdir /srv/repos/datos.git
    git init --bare /srv/repos/datos.git --shared=group
    chgrp -R datos /srv/repos/datos.git
