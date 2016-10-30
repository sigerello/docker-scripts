# Setup Elasticsearch docker container

Create data volumes:

    mkdir -p /var/data/elasticsearch/5.0.0/data
    mkdir -p /var/data/elasticsearch/5.0.0/config
    mkdir -p /var/data/elasticsearch/5.0.0/config/scripts

Create config files:

    nano /var/data/elasticsearch/5.0.0/config/elasticsearch.yml
    nano /var/data/elasticsearch/5.0.0/config/log4j2.properties

Change some system params:

    sysctl -w vm.max_map_count=262144

    echo 'vm.max_map_count = 262144' > /etc/sysctl.conf

Run container:

    docker run \
      --name elasticsearch \
      -d \
      -p 9200:9200 \
      -p 9300:9300 \
      --restart always \
      -v /var/data/elasticsearch/5.0.0/data:/usr/share/elasticsearch/data \
      -v /var/data/elasticsearch/5.0.0/config:/usr/share/elasticsearch/config \
      -v /etc/localtime:/etc/localtime:ro \
      -v /etc/timezone:/etc/timezone:ro \
      elasticsearch:5.0.0

Watch docker container logs:

    docker logs -f --tail=500 elasticsearch

Restart container:

    docker restart elasticsearch

Stop and remove container:

    docker stop elasticsearch && docker rm elasticsearch
