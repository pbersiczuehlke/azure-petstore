# Running the Pet store locally

## Prerequisites
To be able to run the Pet store locally, we need to have some tools installed.
The following have to be installed: `docker`.

## Building the images
The Pet store runs in a microservices architecture with 4 microservices: 3 are background services, and 1 is the frontend.

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

## Todo - turning features on/off with env vars

## Acceptance criteria:
1. Images are built
2. Pet shop is running on your local machine