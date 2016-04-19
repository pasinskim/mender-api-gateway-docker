# API Gateway

Dockerfile for API gateway image based on Tyk.io
`tykio/tyk-gateway:v2.0` image.

# Configuration

## Basic

Preconfigured `tyk.conf`:

- Tyk authorization `secret`: `f35cb63ee4d0405ad9f009e998532d21208d31b3`
- `listen_port`: 8080
- `redis`: looks for redis on `api-gateway-redis` host, on default
  6379 port

## Analytics

Analytics enabled, data is appended to `tyk-system-analytics` set in
redis.

## API definitions

The image contains an API definition for `deployments` service. The
API will be loaded automatically when tyk service starts up.

The definition file `deployments-0.0.1.json` was generated by calling
`tyk` like this:


```
tyk --import-swagger <path-to-api-json> \
--create-api \
--org-id='mender.io' \
--upstream-target='http://mender-deployments:9999/api/0.0.1/'
```

The definition file was manually updated to set `"listen_path":
"/deployments/"`. The API requests coming to
http://<tyk-host>:<listen-port>/deployments/ will be redirected to
http://mender-deployments:9999/api/0.0.1/

### Generating API definitions

The API definition file can also be generated using
`mendersoftware/api-gateway` Docker image like this:

```
docker run \
-v $PWD:/tmp \
-it \
mendersoftware/api-gateway \
    /opt/tyk-gateway/tyk \
    --import-swagger \
    /tmp/<api-file-name> \
    --create-api \
    --org-id 'mender.io' \
    --upstream-target 'http://deployments:9999/api/1.0.0' > tyk-api.json
```

The API file to be automatically loaded during startup, it must be
placed in `/opt/tyk-gateway/apps`.

Note that the API has to be Swagger JSON format. If the API is
described in YAML file, use the following one liner to generate a JSON
file:

`python -c 'import json;import yaml;json.dump(yaml.load(sys.stdin.read()),sys.stdout,indent=2)' < <spec-in-yaml> > out.json`

### Loading API at runtime

Once the tyk service is running the API can be added at runtime by
running the following curl command:

```
curl -H "X-Tyk-Authorization: <secret>"\
 -s\
 -H "Content-Type: application/json"\
 -X POST\
 -d '@<api-file.json>' \
 http://<gateway-host>:8080/tyk/apis/

```

The API definition should end up in `/opt/tyk-gateway/apps`.

### Reloading APIs

Once a new API has been added, tyk needs to be reloaded:

`curl -H "X-Tyk-Authorization: <secret>" http://<tyk-host>:<port>/tyk/reload`
