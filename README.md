## Table of Contents:
* [Requirements](https://github.com/Cyfrowi-Publikanci/pub-lab#requirements)
* [Local development](https://github.com/Cyfrowi-Publikanci/pub-lab#local-development)
* [Access service](https://github.com/Cyfrowi-Publikanci/pub-lab#access-service)
* [Envoy](https://github.com/Cyfrowi-Publikanci/pub-lab#envoy)
* [Databases](https://github.com/Cyfrowi-Publikanci/pub-lab#databases)

<br/>

## Requirements:
* Docker
* docker compose

### Clone repository:

```bash
git clone https://github.com/Cyfrowi-Publikanci/pub-lab
```

### Clone services repositories:

```bash
cd pub-lab
# and
git clone https://github.com/Cyfrowi-Publikanci/publications-service
# and
git clone https://github.com/Cyfrowi-Publikanci/authentication-service
# and
git clone https://github.com/Cyfrowi-Publikanci/admin-service
```

### Expected catalog tree:

```bash
 ~/Projects/ZPWO/pub-lab tree -a -L 1
.
|-- .git
|-- .gitignore
|-- Dockerfile
|-- admin-service
|-- authentication-service
|-- docker-compose.yaml
|-- envoy.yaml
|-- front-service
`-- publications-service

5 directories, 4 files

```

### Create required env files:

```bash
cd publications-service
# and
cp .env.example .env.compose
```

```bash
cd authentication-service
# and
cp .env.example .env.compose
```

```bash
cd admin-service
# and
cp .env.example .env.compose
```

In .env file set SERVICE_HOSTNAME=localhost.<br/>
In file .env and .env.compose add missing values.

<br/>

## Local development:

### Running the app:

```bash
sudo docker-compose up
# or
docker-compose up
```

### Expected result:
```bash
 ~/Projects sudo docker ps
[sudo] password for kamil: 
CONTAINER ID   IMAGE                            COMMAND                  CREATED        STATUS          PORTS                                                                               NAMES
50896bbc5ffc   pub-lab_envoy                    "/docker-entrypoint.…"   2 hours ago    Up 12 seconds   0.0.0.0:8001->8001/tcp, 0.0.0.0:8080->8080/tcp, 0.0.0.0:8443->8443/tcp, 10000/tcp   pub-lab_envoy_1
08eeceff210d   pub-lab_publications-service     "docker-entrypoint.s…"   2 hours ago    Up 13 seconds   3003/tcp                                                                            publications-service
a59cd20fc771   pub-lab_authentication-service   "docker-entrypoint.s…"   19 hours ago   Up 13 seconds   3002/tcp                                                                            authentication-service
f3df8acdf1bc   pub-lab_admin-service            "docker-entrypoint.s…"   19 hours ago   Up 13 seconds   3001/tcp   

```

<br/>

## Access service:

Use [BloomRPC](https://github.com/uw-labs/bloomrpc) to call envoy localhost:8080.<br/>
Choose protofiles form apropriate services.

<br/>

## Envoy:

### Example of exposed endpoint:

```bash
- match:
    prefix: "/admin.UsersService"
  route:
    cluster: admin-service
```

### Example of exposed service:

```bash
 clusters:
    - name: admin-service
      connect_timeout: 0.25s
      type: STRICT_DNS
      lb_policy: ROUND_ROBIN
      http2_protocol_options: {}
      load_assignment:
        cluster_name: admin-service
        endpoints:
          - lb_endpoints:
            - endpoint:
                address:
                  socket_address:
                    address: admin-service
                    port_value: 3001
```

### Example of unprotected endpoint:

```bash
- match:
    prefix: "/authentication.AuthService/loginByEmail"
  route:
    cluster: authentication-service
  typed_per_filter_config:
    envoy.filters.http.ext_authz:
      '@type': type.googleapis.com/envoy.extensions.filters.http.ext_authz.v3.ExtAuthzPerRoute
      disabled: true
```

## Databases:

MongoDB Atlas:
* Setup free cluster https://www.youtube.com/watch?v=KKyag6t98g8
* Setup .env and .env.compose in each project: SERVICE_\<service_name>_DATABASE_URI=mongodb+srv://\<user>:\<password>@<project>.xvmnc.mongodb.net/\<db_name>?retryWrites=true&w=majority


Local replica set:
* Setup .env and .env.compose in each project: SERVICE_\<service_name>_DATABASE_URI=mongodb://mongo-publab:27001/db