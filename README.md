# keycloak-example
Docker compose example on Keycloak.  
Pretty self explanatory really. Just run the damn `docker-compose.yml` file.  

In the `gatekeeper-example` folder theres an example on how to protect a web application with GateKeeper and roles from Keycloak.

```
version: '3'

volumes:
  keycloak-db:

services:
  keycloak:
    image: jboss/keycloak:latest
    container_name: keycloak
    ports:
      - 8080:8080
    environment:
      # Create a user after the container up and running with the following command
      # or uncomment below environment variables.
      # docker exec <container> keycloak/bin/add-user-keycloak.sh -u <username> -p <password>
      #KEYCLOAK_USER: 'admin'
      #KEYCLOAK_PASSWORD: 'admin'
      PROXY_ADDRESS_FORWARDING: 'true'
      DB_VENDOR: 'mariadb'
      DB_ADDR: 'mariadb'
      DB_PORT: '3306'
      DB_DATABASE: 'keycloak'
      DB_USER: 'keycloak'
      DB_PASSWORD: 'database-password-here'

  mariadb:
    image: mariadb:5
    container_name: keycloak-db
    restart: unless-stopped
    expose:
      - 3306
    volumes:
      - keycloak-db:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: 'root-password-here'
      MYSQL_DATABASE: 'keycloak'
      MYSQL_USER: 'keycloak'
      MYSQL_PASSWORD: 'database-password-here'
```
