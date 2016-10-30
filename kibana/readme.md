# Setup Kibana docker container

Create data volumes:

    mkdir -p /var/data/kibana/5.0.0/config

Create config files:

    nano /var/data/kibana/5.0.0/config/kibana.yml

Run container:

    docker run \
      --name kibana \
      -d \
      -p 5601:5601 \
      --restart always \
      -v /var/data/kibana/5.0.0/config:/etc/kibana \
      -v /etc/localtime:/etc/localtime:ro \
      -v /etc/timezone:/etc/timezone:ro \
      -e ELASTICSEARCH_URL=http://192.168.99.100:9200 \
      kibana:5.0.0

Watch docker container logs:

    docker logs -f --tail=500 kibana

Restart container:

    docker restart kibana

Stop and remove container:

    docker stop kibana && docker rm kibana
