Source:
https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_v2config.html#docker-multicontainer-dockerrun-privaterepo





    
,
    {
      "name": "certbot",
      "image": "certbot/certbot",
      "essential": true,
      "memory": 32,
      "mountPoints": [
        {
          "sourceVolume": "certbot-conf",
          "containerPath": "/etc/letsencrypt",
          "readOnly": false
        },
        {
          "sourceVolume": "certbot-www",
          "containerPath": "/var/www/certbot",
          "readOnly": false
        }
      ]
    }




  rest:
    image: opinionatedstack/rest-node
    container_name: rest
    restart: unless-stopped
    ports:
      - 3000:3000
    networks:
      - opsnet
    stdin_open: true
    tty: true

  payments:
    image: opinionatedstack/stripe-payments
    container_name: payments
    restart: unless-stopped
    ports:
      - 3001:3001
    networks:
      - opsnet
    stdin_open: true
    tty: true

  web:
    image: opinionatedstack/web-angular
    container_name: web
    restart: unless-stopped
    depends_on:
      - rest
      - payments
    ports:
      - 80:80
      - 443:443
    networks:
      - opsnet
    stdin_open: true
    tty: true
    volumes:
      #- certbot:/etc/letsencrypt
      #- letsencrypt:/var/www/certbot
      - ./data/certbot/conf:/etc/letsencrypt
      - ./data/certbot/www:/var/www/certbot
    command: "/bin/sh -c 'while :; do sleep 6h & wait $${!}; nginx -s reload; done & nginx -g \"daemon off;\"'"




,
    {
      "name": "certbot",
      "image": "certbot/certbot",
      "essential": true,
      "memory": 32,
      "mountPoints": [
        {
          "sourceVolume": "certbot-conf",
          "containerPath": "/etc/letsencrypt"
        },
        {
          "sourceVolume": "certbot-www",
          "containerPath": "/var/www/certbot"
        }
      ],
      "command": ["/bin/sh", "-c", "'trap exit TERM; while :; do certbot renew; sleep 12h & wait $${!}; done;'"]
    }
    
    
  "volumes": [
    {
      "name": "certbot-conf",
      "host": {
        "sourcePath": "/var/app/current/certbot/conf"
      }
    },
    {
      "name": "certbot-www",
      "host": {
        "sourcePath": "/var/app/current/certbot/www"
      }
    }
  ],




  certbot:
    container_name: certbot
    image: certbot/certbot
    restart: unless-stopped
    volumes:
      #- certbot:/etc/letsencrypt
      #- letsencrypt:/var/www/certbot
      - ./data/certbot/conf:/etc/letsencrypt
      - ./data/certbot/www:/var/www/certbot
    entrypoint: "/bin/sh -c 'trap exit TERM; while :; do certbot renew; sleep 12h & wait $${!}; done;'"
