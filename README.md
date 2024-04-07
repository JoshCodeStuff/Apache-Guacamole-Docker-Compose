
Hello! Welcome to my *Apache Guacamole Docker Compose* repository! The purpose of this project is to add new, up to date install instructions for Apache Guacamole installation to the internet. The current offerings out there are either out of date, horribly formatted, overly clunky (like the Apache docs themselves), or just broken. It is my hope this walkthrough will be easy to understand and well formatted. Without further ado, let's get started! Please note: This was tested in an Ubuntu LXC container hosted by proxmox.

## 1: Install Docker and Docker Compose.

 You can find the instructions to do so here: https://docs.docker.com/engine/install/ 

If you are using Ubuntu, you can use the setup script (dockersetup.sh) in this repo. After running the script, run the following command:
    
    systemctl enable docker && systemctl start docker

## 2: Setting up directories

    mkdir ./init
    chmod -R +x ./init

    docker run --rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --postgresql > ./init/initdb.sql

## 3: Docker Compose File
Create a new Compose file: 

    nano docker-compose.yml

Paste in the following code and save:

    networks:
      guacnetwork_compose:
        driver: bridge
    
    services:
    
      guacd:
        container_name: guacd
        image: guacamole/guacd
        networks:
          guacnetwork_compose:
        restart: always
        volumes:
        - ./drive:/drive:rw
        - ./record:/record:rw
    
      postgres:
        container_name: postgres
        environment:
          PGDATA: /var/lib/postgresql/data/guacamole
          POSTGRES_DB: guacamole_db
          POSTGRES_PASSWORD: 'ChangeMe'
          POSTGRES_USER: guacamole_user
        image: postgres:15.2-alpine
        networks:
          guacnetwork_compose:
        restart: always
        volumes:
        - ./init:/docker-entrypoint-initdb.d:z
        - ./data:/var/lib/postgresql/data:Z
    
      guacamole:
        container_name: guacamole
        depends_on:
        - guacd
        - postgres
        environment:
          GUACD_HOSTNAME: guacd
          POSTGRES_DATABASE: guacamole_db
          POSTGRES_HOSTNAME: postgres
          POSTGRES_PASSWORD: 'ChangeMe'
          POSTGRES_USER: guacamole_user
          TOTP_ENABLED: 'true'
          WEBAPP_CONTEXT: ROOT #Changes URL from <IP>/guacamole/ to just <IP>
        image: guacamole/guacamole
        links:
        - guacd
        networks:
          guacnetwork_compose:
        ports:
        - 80:8080/tcp
        restart: always


## 4: Start the containers!

    docker compose up -d
    
## 5: Final Steps

Navigate to the IP Address of the install. (for example) 10.0.0.200
