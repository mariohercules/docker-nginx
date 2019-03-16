# docker-nginx
Docker NGINX load balancer with WordPress and MySQL

## Scheme 

![Screenshot](Diagram.png)

## How to use

* create a file `docker-composer.yaml` and put the contents below:

```yaml
version: '3'
services:
  nginx:
    image: nginx:latest
    restart: always
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/log:/var/log/nginx      
    ports:
      - 80:80
      - 443:443
    depends_on:
      - wordpress1
      - wordpress2
      - wordpress3       
  db:
    image: mysql:5.5.60
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_ROOT_PASSWORD: w0rdpress
      MYSQL_USER: root
      
  wordpress1:
    #container_name: wordpress1
    image: wordpress:latest
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: w0rdpress
    volumes: 
      - ./wordpress:/var/www/html
    depends_on:
      - db
  wordpress2:
    #container_name: wordpress2
    image: wordpress:latest
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: w0rdpress
    volumes: 
      - ./wordpress:/var/www/html
    depends_on:
      - db  
  wordpress3:
    #container_name: wordpress2
    image: wordpress:latest
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: w0rdpress
    volumes: 
      - ./wordpress:/var/www/html
    depends_on:
      - db                
```

* create a file `nginx.conf` and put the contents below:

```conf
events { worker_connections 2048; }

http { 
  upstream localhost { # configura um pool de endereço de servidores  
      server docker_wordpress1_1;
      server docker_wordpress2_1;
      server docker_wordpress3_1;      
  }
  
  server { # configura esse servidor
      listen 80 default_server; # escutando por conexões na porta 80
      listen [::]:80 default_server;

      root /usr/share/nginx/html;
      index index.php;

      location / { # repassa todos os requests para um dos endereços do upstream
        proxy_pass  http://localhost;  # esse endereço aponta para o upstream wordpress
        add_header X-Upstream $upstream_addr; # adiciona o header Host com o valor de um dos endereços configurados no upstream
      }
  }
}

```

```bash
$ docker-compose up -d
```

* Open `http://localhost`
* Install e configure WordPress 
