# Learning Docker & Kubernetes Demo

This is a small example showing the use of Docker & Kubernetes. This sample was created for my use during lectures, so nothing fancy here :-).

## Building the API Docker image

Run the following command from the repository's root to build basic Docker images for the API.

Version 1.0 (using SQLite database, no volumes configured)

```bash
docker build -t sswietoniowski/todo-api:version1.0 -f backend/api/api-sqlite.dockerfile ./backend/api
```

Version 2.0 (using SQLite database, volumes configured)

```bash
docker build -t sswietoniowski/todo-api:version2.0 -f backend/api/api-sqlite-with-volumes.dockerfile ./backend/api
```

Version 3.0 (using MSSQL database in a separate container)

```bash
docker build -t sswietoniowski/todo-api:version3.0 -f backend/api/api-mssql.dockerfile ./backend/api
```

## Running the API Docker image (locally)

To run the API, run the following command from the root of the repository:

```bash
docker run -p 5000:5000 -p 5001:5001 -d sswietoniowski/todo-api:version1.0
docker run -p 5000:5000 -p 5001:5001 -d sswietoniowski/todo-api:version2.0
```

In the case of the MSSQL version, you need to run the MSSQL container first:

```bash
docker network create todo-app
docker volume create sqlvolume
docker run --network todo-app --network-alias mssql -p 2433:1433 -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=Password123!' -e 'MSSQL_PID=Developer' -v sqlvolume:/var/opt/mssql -d mcr.microsoft.com/mssql/server:2019-latest
```

To test the connection to the MSSQL server from the app, run the following command:

```bash
docker run --network todo-app -it --rm mcr.microsoft.com/mssql-tools sqlcmd -S mssql -U SA -P Password123!
$Env:DatabaseEngine = "Mssql"
$Env:ConnectionStrings:MssqlConnection = "Server=localhost,2433;Database=todos;User=sa;Password=Password123!;"
cd .\backend\api
dotnet run
```

Then, run the API container:

```bash
docker run -p 5000:5000 -p 5001:5001 --network todo-app -d sswietoniowski/todo-api:version3.0
```

## Pushing the API Docker image to Docker Hub

To push the API Docker image to Docker Hub, run the following command from the root of the repository:

```bash
docker push sswietoniowski/todo-api:version1.0
docker push sswietoniowski/todo-api:version2.0
docker push sswietoniowski/todo-api:version3.0
```

## Running the API Docker image (remotely)

To do so, execute the same commands as above, but do so from a [remote machine](https://labs.play-with-docker.com/).

## Running the API Docker image (locally) with Docker Compose

To run the API with Docker Compose, use the following command from the root of the repository:

```bash
docker-compose -f ./backend/api/docker-compose.yaml up -d
```

## Running the API in Kubernetes

To run the API in Kubernetes, use the following command from the root of the repository:

```bash
kubectl apply -f ./backend/api/db-deployment.kubernates.yaml
kubectl apply -f ./backend/api/db-service.kubernates.yaml
kubectl apply -f ./backend/api/api-deployment.kubernates.yaml
kubectl apply -f ./backend/api/api-service.kubernates.yaml
```

We can then show how easy it is to scale the API deployment to 3 replicas:

```bash
kubectl scale deployment todo-api --replicas=3
```

If the above won't work, try the following:

```bash
cd ./backend/api
kubectl create deployment todo-api --image sswietoniowski/todo-api:version1.0
kubectl get deployments
kubectl describe deployments/todo-api
kubectl get pods
kubectl expose deployment/todo-api --type=LoadBalancer --port 5000
kubectl get services
kubectl describe services/todo-api
kubectl delete service -l app=todo-api
kubectl delete deployment -l app=todo-api
kubectl get deployments
kubectl get pods
kubectl get services
```

You can try to do the same on a remote machine for free using [this](https://labs.play-with-k8s.com/) service :-).
