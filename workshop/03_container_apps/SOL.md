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
6. Choose region 'Sweden Central'
7. Click 'Create new' under the 'Container Apps Environment'
8. Name the environment 'ca-env-petstore'
9. Go to networking and choose 'Yes' under 'Use your own virtual network'
10. Choose the provided VNet & subnet for the container apps
11. Set Virtual IP to Internal
12. Click 'Create' -> We have now created an internal Container Apps Environment
13. Click 'Next: Container >'
14. Choose public registry: docker.io and image hello-world:latest
15. Click 'Next: Ingress >'
16. Select 'Limited to VNet'
17. Set Target port to '8080'
18. Click 'Next'
19. Add your team tags
20. Click 'Create'
 
Unfortunately due to permission issues we can not immediately create a container app with the image from our ACR.
After the Environment and the Contaienr App have been successfully created, navigate to the petstoreapp.
1. Go to Settings->Identity->User assigned->Add
2. Choose a managed identity with the name `id-hackathon-devlab-00<group_number>`
We have now added the managed identity to the Container App allowing it to access our ACR.

To actually deploy our petstoreapp:
1. Go to Application->Containers->Edit and deploy
2. Click on the petstoreapp image
3. Choose Azure Container Registry
4. Choose the ACR that you created
5. Choose 'petstoreapp' for the image name and '0.1.0' or whichever tag you chose when you uploaded the image
6. Choose Authentication Type: Managed Identity
7. Choose the managed identity you assigned above
8. Click Save->Create

We have now created the Frontend service.
Go to the created container app and click on the 'Application Url'.
You will see that the browser can not access the Petstore.

## Private DNS Zone
Four our browser to connect to the website we need to add a DNS record.
1. Go to the Azure Portal
2. Search for Private DNS Zone
3. Create
4. We need two peaces of information: domain and ip address
5. We can find the domain in the petstoreapp Container App
Example: If the application uri for our petstoreapp is https://petstoreapp.lemonfield-fc1fb0ba.swedencentral.azurecontainerapps.io, 
then the domain is `lemonfield-fc1fb0ba.swedencentral.azurecontainerapps.io`
6. The ip address can be found on the container app environment page at the top right under Static IP.
Example: 10.0.1.16
7. Add the domain as the name of the Private DNS Zone
8. Add tags and create it
9. Click on the left-hand side DNS Management->Recordsets
10. We will Add two records: one with the name `@` the other with the name `*`.
Both will have the same ip address from the above.
11. Once the two records are created navigate to Virtual Network Links and link the Zone with our VNet

Now our frontend petstoreapp will be accessible through the browser.
Browse around! Try to click on the links at the 'Shop' links at the bottom.
Does it work? Why not?
It doesn't work because the backend services that the frontend service calls are not yet created.

## Creating the Backend Services
The procedure to create the 3 backend services is very similar to creating the frontend service.
The difference is that we do **NOT** create a new Container Apps Environment every time.
Instead, we select the existing one.
For the backend services we also need to enable ingress.
However, they do not need to be accessible from the entire VNet.
Therefore, we make the Ingress Traffic 'Limited to Container Apps Environment'.
All backend services have the same target port of '8080'.
The procedure of first starting with the `hello-world:latest` image, assigning the managed identity, and 
switching the image to the one from the ACR is identical.

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
The procedure for how to test with jmeter can be found [here](https://www.geeksforgeeks.org/how-to-use-jmeter-for-performance-and-load-testing/).

By default, the scaling is set from 0 to 10.
When there is no traffic, the Container App will scale down to zero.
When there is a sufficient traffic, it will scale all the way up to 10.
We can use jmeter to make it scale all the way to 10.

You have probably noticed that the apps sometimes scale to zero. 
You do not get charged during the time the app is scaled to zero.
However, the major drawback is that the first time you try to access the app, you
need to wait for about half a minute for the app to be ready.