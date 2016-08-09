# Setup Mysql docker container

Create data volumes:

    mkdir -p /var/data/mysql/5.7.10/data
    mkdir -p /var/data/mysql/5.7.10/config/conf.d

Create ```my.cnf``` config file:

    nano /var/data/mysql/5.7.10/config/my.cnf

Generate password and save it to file:

    openssl rand -hex 8 > /var/data/mysql/5.7.10/ROOTPASSWORD
    chmod 600 /var/data/mysql/5.7.10/ROOTPASSWORD

Run container:

    docker run \
      --name mysql \
      -d \
      -p 3306:3306 \
      --restart always \
      -v /var/data/mysql/5.7.10/data:/var/lib/mysql \
      -v /var/data/mysql/5.7.10/config:/etc/mysql \
      -v /etc/localtime:/etc/localtime:ro \
      -v /etc/timezone:/etc/timezone:ro \
      -e MYSQL_RANDOM_ROOT_PASSWORD=yes \
      -e MYSQL_ONETIME_PASSWORD=yes \
      mysql:5.7.10

(Watch ```docker logs mysql``` to see generated password)

Access to mysql console:

    docker exec -it mysql mysql -u root -p

Generate new password and set it to root user:

    mysql> ALTER USER root IDENTIFIED BY 'ROOTPASSWORD';

Check proper encoding configuration:

    mysql> SHOW VARIABLES LIKE "%character%"; SHOW VARIABLES LIKE "%collation%";

Create user for each project:

    mysql> CREATE DATABASE `proj` DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
    mysql> GRANT ALL PRIVILEGES ON `proj`.* TO `proj>`@`%` IDENTIFIED BY 'NEWPASSWORD';

Check proper encoding configuration:

    mysql> SELECT Host, User FROM mysql.user;

Remove container and data:

    $ docker stop mysql && docker rm mysql
    $ rm -rf /var/data/mysql/5.7.10/data/*
