# Running the Petstore locally (Optional)

## Prerequisites
The application is packaged as a set of Docker containers.
To build and run the application we need to have `docker` installed on the workstation.
There is no Docker on the provided VMs.
Therefore, to follow this guide you need to have `docker` installed on your machine.
If Docker is not already installed, you may skip this task.

## Building the images
The Petstore runs in a microservices architecture with 4 microservices: 1 frontend, and 3 background services.

Building the container images is done with:
```bash
# pet backend service
cd petstore/petstorepetservice
docker build -t petstorepetservice:0.1.0 .

# product backend service
cd petstore/petstoreproductservice
docker build -t petstoreproductservice:0.1.0 .

# order backend service
cd petstore/petstoreorderservice
docker build -t petstoreorderservice:0.1.0 .

# frontend app service
cd petstore/petstoreapp
docker build -t petstoreapp:0.1.0 .
```

## Running the pet store locally
```bash
# create a bridge network
docker network create petstorebridge

# start petservice
docker run --rm --net petstorebridge --name petstorepetservice -p 8081:8081 -e PETSTOREPETSERVICE_SERVER_PORT=8081 -d --ulimit nofile=65536:65536 petstorepetservice:0.1.0
# petservice is not available at localhost:8081

# start productservice
docker run --rm --net petstorebridge --name petstoreproductservice -p 8082:8082 -e PETSTOREPRODUCTSERVICE_SERVER_PORT=8082 -d --ulimit nofile=65536:65536 petstoreproductservice:0.1.0
# productservice is not available at localhost:8082

# get the IP of the required product service
PETSTORE_PRODUCT_SERVICE_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' petstoreproductservice)

# start orderservice
docker run --rm --net petstorebridge --name petstoreorderservice -p 8083:8083 \
  -e PETSTOREORDERSERVICE_SERVER_PORT=8083 \
  -e PETSTOREPRODUCTSERVICE_URL=http://$PETSTORE_PRODUCT_SERVICE_IP:8082 \
  -d --ulimit nofile=65536:65536 petstoreorderservice:0.1.0
# orderservice is not available at localhost:8083
```

To start the petstore app, we need to link it with the above microservices.
We will use `docker inspect` to extract the ips addresses of the microservices.
```bash
# start petstoreapp
NETWORK_NAME="petstorebridge"

# Get the IP addresses of the required services
PETSTORE_PET_SERVICE_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' petstorepetservice)
PETSTORE_PRODUCT_SERVICE_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' petstoreproductservice)
PETSTORE_ORDER_SERVICE_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' petstoreorderservice)

# Run the petstoreapp container with dynamically retrieved IPs
docker run --rm --net $NETWORK_NAME --name petstoreapp -p 8080:8080 \
  -e PETSTOREAPP_SERVER_PORT=8080 \
  -e PETSTOREPETSERVICE_URL=http://$PETSTORE_PET_SERVICE_IP:8081 \
  -e PETSTOREPRODUCTSERVICE_URL=http://$PETSTORE_PRODUCT_SERVICE_IP:8082 \
  -e PETSTOREORDERSERVICE_URL=http://$PETSTORE_ORDER_SERVICE_IP:8083 \
  --ulimit nofile=65536:65536 \
  -d petstoreapp:latest
# petstoreapp is now available at localhost:8080
```

## Explore the Petshop
Navigate your browser to `http://localhost:8080` and explore the Pet shop.

## Configuring a service
All services used in this workshop are built with Spring Boot.
As such they are configured using the `application.yml` file in the `src/resources` subdirectory of each service.
Some of the configurations pull their configuration from the environment variables.
Therefore, we can change the configuration of the service just by changing the environment variables.

## Acceptance criteria:
1. Images are built
2. Pet shop is running on your local machine