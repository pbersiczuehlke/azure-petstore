# Building and Running Locally (Optional)

This exercise is optional.
However, it will provide a deeper understanding of containerized applications.
In order to build and run locally, you need to have Docker installed on your machine.

The code for the four services that our Petshop consists of is located at:
`petstore/petstoreapp`, `petstore/petstorepetservice`, `petstore/petstoreproductservice`, `petstore/petstoreorderservcie`.

Each of those directories contains a Dockerfile that builds that microservice.

## Task 1:
Build the Docker images.
You should have 4 images by the end:

```
petstorepetservice:0.1.0
petstoreproductservice:0.1.0
petstoreorderservice:0.1.1
petstoreapp:0.1.0
```

You are free to name them and tag them as you see fit; these are just the suggestions.

Links:
* [Building Docker Images](https://docs.docker.com/get-started/docker-concepts/building-images/build-tag-and-publish-an-image/)
* [A Beginner's Guide](https://stackify.com/docker-build-a-beginners-guide-to-building-docker-images/)

## Task 2:
Run the Petstore locally.
It should be noted that the frontend `petstoreapp` service needs to be set up to connect to the backend services.
It is recommended to create a new bridge network in Docker with which all the services will be connected.

Every service allows us to set the port number under which it will be exposed:
* For the `petservice`: `PETSTOREPETSERVICE_SERVER_PORT`
* For the `productservice`: `PETSTOREPRODUCTSERVICE_SERVER_PORT`
* For the `orderservice`: `PETSTOREORDERSERVICE_SERVER_PORT`
* For the frontend `petstoreapp`: `PETSTOREAPP_SERVER_PORT`

By default, all these ports are 8080.

To connect between services, we need to specify the URLs to other services.
For the `petstoreapp`, we need to specify:

```
PETSTOREPETSERVICE_URL
PETSTOREPRODUCTSERVICE_URL
PETSTOREORDERSERVICE_URL
```

And for the `orderservice`, we need to specify:

```
PETSTOREPRODUCTSERVICE_URL
```

If the `orderservice` has an IP address of `10.0.0.1` in the bridge network, then the URL for the order service in the `petstoreapp` should be set:

```
PETSTOREORDERSERVICE_URL=http://10.0.0.1:8080
```

where the port `8080` depends on the `PETSTOREORDERSERVICE_SERVER_PORT` setting.

Note: Please add `--ulimit nofile=65536:65536` to the `docker run` command to avoid issues with Java.

Links:
* [Passing Environment Variables to Docker Containers](https://www.baeldung.com/ops/docker-container-environment-variables)
* [Create Bridge Network](https://docs.docker.com/reference/cli/docker/network/create/#description)
* [Container Networking](https://docs.docker.com/engine/containers/run/#container-networking)

## Acceptance Criteria:
1. Images are built.
2. Petshop is running on your local machine.
