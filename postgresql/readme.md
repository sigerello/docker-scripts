# Setup Postgresql docker container

Create data volumes:

    mkdir -p /var/data/postgresql/9.5.22/data

Generate password and save it to files:

    echo "$(openssl rand -hex 64 | sha256sum | base64 | head -c 16 ; echo)" | awk '{print tolower($0)}' > /var/data/postgresql/9.5.22/ROOTPASSWORD
    chmod 600 /var/data/postgresql/9.5.22/ROOTPASSWORD

    echo "$(openssl rand -hex 64 | sha256sum | base64 | head -c 16 ; echo)" | awk '{print tolower($0)}' > /var/data/postgresql/9.5.22/PROJPASSWORD
    chmod 600 /var/data/postgresql/9.5.22/PROJPASSWORD

Run container:

    docker run \
      --name postgresql \
      -d \
      -p 5432:5432 \
      --restart always \
      -v /var/data/postgresql/9.5.22/data:/var/lib/postgresql/data \
      -v /etc/localtime:/etc/localtime:ro \
      -v /etc/timezone:/etc/timezone:ro \
      postgres:9.5.22

(Watch ```docker logs postgresql``` to check DB init process is ok)

Access to psql console:

    docker exec -it postgresql psql -U postgres

    postgres> \?    - help
    postgres> \du+  - list roles
    postgres> \l+   - list databases
    postgres> \q    - exit


Change root user password:

    postgres> ALTER USER postgres WITH ENCRYPTED PASSWORD 'rootpassword';

Revoke privileges on system tables from PUBLIC:

    postgres> REVOKE ALL ON DATABASE postgres, template0, template1 FROM PUBLIC;

Create user and database for each project (substitute ```proj``` with your actual project/user name):

    postgres> CREATE USER proj WITH ENCRYPTED PASSWORD 'projpassword';
    postgres> CREATE DATABASE proj OWNER proj;
    postgres> REVOKE ALL ON DATABASE proj FROM PUBLIC;
    postgres> GRANT ALL ON DATABASE proj TO proj;

Secure connection with SSL:

    openssl req -new -text -out server.req
    openssl rsa -in privkey.pem -out server.key
    rm privkey.pem
    openssl req -x509 -in server.req -text -key server.key -out server.crt
    chmod 600 server.key
    rm server.req

Change some settings:

    sed -ie "s/max_connections = 100/max_connections = 1000/" /var/data/postgresql/9.5.22/data/postgresql.conf
    sed -ie "s/#ssl = off/ssl = on/" /var/data/postgresql/9.5.22/data/postgresql.conf
    sed -ie "s/#ssl_ciphers = .*/ssl_ciphers = 'AES256+EECDH:AES256+EDH'/" /var/data/postgresql/9.5.22/data/postgresql.conf
    sed -ie "s/#log_line_prefix = ''/log_line_prefix = '%t '/" /var/data/postgresql/9.5.22/data/postgresql.conf


Change host-based authentication settings:

    nano /var/data/postgresql/9.5.22/data/pg_hba.conf

    local all all md5
    host all all 127.0.0.1/32 md5
    host all all ::1/128 md5
    host all all 0.0.0.0/0 md5

Watch docker container logs:

    docker logs -f --tail=500 postgresql

Restart container:

    docker restart postgresql

Stop and remove container:

    docker stop postgresql && docker rm postgresql
