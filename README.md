# Odoo Ansible Docker Traefik Swarm
```
http://traefik.demo.marlonfalcon.com/dashboard/#/
http://swarm.demo.marlonfalcon.com/
```

# 1- Creamos el server y copiamos la llave
```
ssh-copy-id root@1.1.1.1
```

# 2- Creamos los DNS
```
A
cuba.marlonfalcon.com -> 1.1.1.1

CNAME
*.cuba.marlonfalcon.com -> cuba.marlonfalcon.com
```

# 3- Configuramos el host en la carpeta Deploy

# 4- Hacemos el deploy
```
cd deploy/ansible
ansible-playbook -i inventories/cuba.marlonfalcon.com -e domain_name=cuba.marlonfalcon.com playbook-master.yml
```

# 5- Instalamos el docker yml en el stack
```
version: '3.7'
services:
  app-odoo:
    image: odoo
    environment:
      HOST: db
      PORT: 5432
      USER: odoo
      PASSWORD: odoo
    volumes:
      - app-odoo_filestore:/var/lib/odoo/filestore
    depends_on:
      - db
    networks:
      - app-odoo_net
      - traefik
    deploy:
      replicas: 1
      update_config:
        parallelism: 1
        delay: 2s
      labels:
        swarmpit.service.deployment.autoredeploy: 'true'

        traefik.enable: 'true'
        traefik.http.services.app-odoo.loadbalancer.server.port: '8069'

        traefik.http.routers.app-odoo-http.service: app-odoo
        traefik.http.routers.app-odoo-http.rule: Host(`cuba.marlonfalcon.com`)
        traefik.http.routers.app-odoo-http.entrypoints: http
        traefik.http.routers.app-odoo-http.middlewares: app-odoo-https-redirect
        traefik.http.middlewares.app-odoo-https-redirect.redirectscheme.scheme: https

        traefik.http.routers.app-odoo-https.service: app-odoo
        traefik.http.routers.app-odoo-https.tls: 'true'
        traefik.http.routers.app-odoo-https.tls.certresolver: letsencrypt
        traefik.http.routers.app-odoo-https.rule: Host(`cuba.marlonfalcon.com`)
        traefik.http.routers.app-odoo-https.entrypoints: https

  db:
    image: postgres:13
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_PASSWORD=odoo
      - POSTGRES_USER=odoo
      - PGDATA=/var/lib/postgresql/data/pgdata
    networks:
      - app-odoo_net
    volumes:
      - app-odoo_data:/var/lib/postgresql/data/pgdata

volumes:
  app-odoo_data:
  app-odoo_filestore:

networks:
  app-odoo_net:
  traefik:
    external: true
```

