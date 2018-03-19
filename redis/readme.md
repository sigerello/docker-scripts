# Setup Redis docker container

Create data volumes:

    mkdir -p /var/data/redis/4.0.8/data
    mkdir -p /var/data/redis/4.0.8/config
    
Copy example config from here [http://download.redis.io/redis-stable/redis.conf](http://download.redis.io/redis-stable/redis.conf) and save it to `/var/data/redis/4.0.8/config/redis.conf`

Change `bind` param in config to make redis listen on private network interface (change IP to your current private IP):

    bind 127.0.0.1 10.0.0.1

Change some system params:

    sysctl -w net.core.somaxconn=65535
    sysctl -w vm.overcommit_memory=1

    echo 'net.core.somaxconn = 65535' >> /etc/sysctl.conf
    echo 'vm.overcommit_memory = 1' >> /etc/sysctl.conf

    # nano /etc/rc.local
    if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
      echo never > /sys/kernel/mm/transparent_hugepage/enabled
    fi

    if test -f /sys/kernel/mm/transparent_hugepage/defrag; then
      echo never > /sys/kernel/mm/transparent_hugepage/defrag
    fi

Run container:

    docker run \
      --name redis \
      -d \
      -p 6379:6379 \
      --restart always \
      --net host \
      -v /var/data/redis/4.0.8/data:/data \
      -v /var/data/redis/4.0.8/config:/usr/local/etc/redis \
      -v /etc/localtime:/etc/localtime:ro \
      -v /etc/timezone:/etc/timezone:ro \
      redis:alpine \
      redis-server /usr/local/etc/redis/redis.conf

Watch docker container logs:

    docker logs -f --tail=500 redis

Restart container:

    docker restart redis

Stop and remove container:

    docker stop redis && docker rm redis
