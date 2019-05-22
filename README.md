### Deploying RSPeer via Single Node Docker Swarm



**Postgres Persistent Volume:**

The following path must exist on the host machine before deployment.

```
/var/lib/postgresql/data/rspeer
```

**Postgres Secrets:**

The following secrets must be created before deployment: **rspeer_postgress_pass**

The host must be apart of a docker swarm to create the secret.

```
docker swarm init
```

```
printf POSTGRESS_PASSWORD_HERE | docker secret create rspeer_postgress_pass -
```


**Build Nginx and Nginx Proxy**

A custom nginx and custom nginx proxy image is utilized for the RSPeer infastructure. 

These dockerfiles are located in the dockrfiles/nginx folder.

The reverse proxy is forked from https://github.com/jwilder/nginx-proxy.

The custom nginx image is the official image + some custom configuration for tuning.

Both these images need to be built, the nginx one must be built first since the nginx proxy relies on it.

```
cd dockerfiles/nginx
docker build -f Dockerfile-Nginx -t rspeer-nginx .
docker build -f Dockerfile-Dockergen -t rspeer-nginx-docker .
```



**Build RSPeer Api**

Go to RSPeer-Api solution and publish the API to a /publish folder and move the Dockerfile-Api in dockerfiles, to the directory that contains the /publish folder.

```
dotnet publish RSPeer.Api.csproj -c Release -o ./publish
```

Build the image with the tag: **rspeer-api**. The dockerfile will copy the output from the /publish folder, so make you its in the same directory of the Dockerfile.

```
docker build -f Dockerfile-Api -t rspeer-api .
```



**Build RSPeer Compiler**

Go to RSPeer-Compiler solution and publish the API to a /publish folder and move the Dockerfile-Api in dockerfiles, to the directory that contains the /publish folder.

```
dotnet publish RSPeer.Compiler.csproj -c Release -o ./publish
```

Build the image with the tag: **rspeer-compiler**. The dockerfile will copy the output from the /publish folder, so make you its in the same directory of the Dockerfile.

```
docker build -f Dockerfile-Compiler -t rspeer-compiler .
```



**Build RSPeer WS**

Move the Dockerfile-Ws in dockerfiles, to the root directory of the RSPeer-WS solution and build the image with the tag: **rspeer-ws**

```
docker build -f Dockerfile-Ws -t rspeer-ws .
```



**Deploying**

Once all the images are built, deployment should be as simple as running the **docker-compose.yml** file.

```
docker stack deploy -c docker-compose.yml --prune rspeer
```



**Force Updating**

Docker swarm will not update services if you update the image but do not change the version, but you can force it with the following commands.

<!--RSPeer WS-->

```
docker service update --image rspeer-ws:latest --force rspeer_instance-ws
```

<!--RSPeer Api-->

```
docker service update --image rspeer-api:latest --force rspeer_api
```

<!--RSPeer Compiler-->

```
docker service update --image rspeer-compiler:latest --force rspeer_compiler
```

<!--Nginx Reverse Proxy-->

```
docker service update --image rspeer-nginx-docker:latest --force rspeer_nginx-proxy
```

