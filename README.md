# Matrix Synapse, Elements Web, Coturn

## Matrix Synapse

1. Create a directory for the configuration files: `mkdir synapse`
2. Create configuration file for matrix synapse. Replace the `PATH_TO_SYNAPSE_CONFIG_FOLDER` and the `my.matrix.host`

```
docker run -it --rm \
    -v PATH_TO_SYNAPSE_CONFIG_FOLDER:/data \
    -e SYNAPSE_SERVER_NAME=my.matrix.host \
    -e SYNAPSE_REPORT_STATS=yes \
    matrixdotorg/synapse:latest generate
```

3. Configure Coturn server: set the following variables in the `homeserver.yaml`
   1. `turn_uris: ["turn:YOUR_DOMAIN:3478?transport=udp", "turn:YOUR_DOMAIN:3478?transport=tcp" ]`
   2. `turn_shared_secret: "TURN_SHARED_SECRET_FROM_COTURN_CONFIGURATION"`
   3. `turn_allow_guests: True`

### Postgres instead of Sqlite3

The default database is Sqlite3. For production use you should change to postgres. 

Example postgres usage via docker-compose:
```
postgres:
    image: postgres
    container_name: postgres
    restart: always
    volumes:
      - ../../data/postgresdata:/var/lib/postgresql/data
    networks: 
      - database_network
```

Setup for database can here found: [https://github.com/matrix-org/synapse/blob/develop/docs/postgres.md#set-up-database](https://github.com/matrix-org/synapse/blob/develop/docs/postgres.md#set-up-database)

### Openldap integration

Install openldap see [https://github.com/baireutherjonas/docker-openldap](https://github.com/baireutherjonas/docker-openldap).

Configure matrix synapse to allow login via openldap insert the following block in the `homeserver.yaml` file

```
password_providers:
 - module: "ldap_auth_provider.LdapAuthProvider"
   config:
     enabled: true
     mode: "search"
     uri: "ldap://openldap:389"
     start_tls: true
     base: "ou=users,dc=YOUR_DOMAIN_DC,dc=de"
     attributes:
        uid: "cn"
        mail: "mail"
        name: "gecos"
```

## Coturn

Replace the following placeholders in the `coturn/turnserver.conf`

1. Set the domain name `realm=domain.com`
2. Set the external-ip `external-ip=123.123.123.123`
3. Create a shared secret with the following command `openssl rand -hex 32` and set the secret in the config

## Elements Web

Replace the following placeholders in the  `elements_web/config.json`

1. Set the base_url `"base_url": "https://my-fancy-company.de"`
2. Set the server_name `"server_name": "My Fancy Company"`

## Nginx - Reverseproxy

Setup the nginx reverseproxy like [https://github.com/baireutherjonas/nginx-reverseproxy](https://github.com/baireutherjonas/nginx-reverseproxy)

Add the following new endpoints to the `nginx.conf`. The config is needed for the following ports 80/443 and 8448

```
location ~* ^(/_matrix|/_synapse|/_client) {
    proxy_pass http://synapse-stream;
    proxy_set_header X-Forwarded-For $remote_addr;
    client_max_body_size 200M;
}
```
