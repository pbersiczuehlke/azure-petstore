# Set up Azure Container Apps
Our microservices will be deployed using Azure Container Apps.
Azure Container Apps is a serverless PaaS built on top of Kubernetes.

## Creating the Frontend Service
All Container Apps reside inside a Container Apps Environment.
During the creation of our first microservice, we will create the Environment.

To create the frontend service:
1. Navigate to the Azure Portal
2. Search for 'Container Apps'
3. Click '+ Create' -> 'Container App'
4. Choose your resource group
5. Name the app 'petstoreapp'
6. Choose region 'Switzerland North'
7. Click 'Create new' under the 'Container Apps Environment'
8. Name the environment 'ca-env-petstore'
9. Go to networking and choose 'Yes' under 'Use your own virtual network'
10. Choose the provided VNet for the virtual network
11. Create a new 'sn-<teamname>-ca' subnet
12. Set Virtual IP to Internal
13. Click 'Create' -> We have now created an internal Container Apps Environment
14. Click 'Next: Container >'
15. Choose the ACR that you created
16. Choose 'petstoreapp' for the image name and '0.1.0' or '0.1.0-az' for the image tag
17. Click 'Next: Ingress >'
18. Select 'Limited to VNet'
19. Set Target port to '8080'
20. Click 'Next'
21. Add your team tags
22. Click 'Create'

We have now created the Frontend service.
Go to the created container app and click on the 'Application Url'.
Browse around! Try to click on the links at the 'Shop' links at the bottom.
Does it work? Why?
It doesn't work because the backend services that the frontend service calls are not yet created.

TODO: Can this be accessed from the corporate network?
TODO: How long will it take for the service to be available through DNS?


## Creating the Backend Services
The procedure to create the 3 backend services is very similar to creating the frontend service.
The difference is that we do **NOT** create a new Container Apps Environment every time.
Instead, we select the existing one.
For the backend services we also need to enable ingress.
However, they do not need to be accessible from the entire VNet.
Therefore, we make the Ingress Traffic 'Limited to Container Apps Environment'.
All backend services have the same target port of '8080'.

Name them as follows:
1. 'petservice', image name `petstorepetservice:0.1.0`
2. 'productservice', image name `petstoreproductservice:0.1.0`
3. 'orderservice', image name `petstoreorderservice:0.1.0`

If you now browse the frontend application, you will still notice that the 'Shop' links do not work.
Why is that?

The reason is that the frontend service needs to know how to access the backend services.
For this we will use 'Dapr', a service that lets us call other services by their name.

## Enabling Dapr on all services
To enable communication using Dapr, we need to turn it on for each service.

Steps:
1. Navigate to the Azure Portal to the Container App Environment that you created.
2. Open the 'petservice' Container App
3. On the left sidepanel, open 'Settings' -> 'Dapr' -> 'Enabled'
4. Set the name to 'petservice'
5. Set the app port to '8080'
6. Save

The same procedure needs to be applied to each service, each Container App, changing the name for each service.
If you now go to the frontend service, you will still notice that the connection to the backend services does not work.
The reason is that we still need to configure the frontend service to know where to find the backend services.

## Configuring the communication between microservices
When we enable Dapr, it deploys an agent in a sidecar configuration.
This means that each pod has now two containers. 
One is our app, the other is the Dapr sidecar.
When we want to communicate with another service, we don't directly talk to it.
Instead, we send our request through the sidecar container.

For instance: to call the petservice `pet/findByStatus` endpoint, we issue a call to:
```
http://localhost:3500/v1.0/invoke/petservice/method/pet/findByStatus
```
Dapr then makes sure to forward the call to the petservice.

Our microservices are configured using environment variables.
The frontend service can be configured using these three environment variables:
```bash
PETSTOREPETSERVICE_URL
PETSTOREPRODUCTSERVICE_URL
PETSTOREORDERSERVICE_URL
```
They have to be set such that the frontend service can reach the backend services.
Therefore, we need to set them to:
```bash
PETSTOREPETSERVICE_URL=http://localhost:3500/v1.0/invoke/petservice/method/
PETSTOREPRODUCTSERVICE_URL=http://localhost:3500/v1.0/invoke/productservice/method/
PETSTOREORDERSERVICE_URL=http://localhost:3500/v1.0/invoke/orderservice/method/
```

The `orderservice` also needs to be able to communicate with the `productservice`.
For this we need to set the environment variables on the `orderservice`:
```bash
PETSTOREPRODUCTSERVICE_URL=http://localhost:3500/v1.0/invoke/productservice/method/
```

Monitor the status of the revision and browse the petstore.
The petstore should now work, except for the final email in the end.

## Scaling & load testing with jmeter
We will be using jmeter to put load on the application so that we can observe scaling.