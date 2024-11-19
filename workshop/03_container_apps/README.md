# Set up Azure Container Apps

Our microservices will be deployed using Azure Container Apps, a serverless PaaS built on top of Kubernetes.

## Task 3.1: Creating the Frontend Service

- **Objective**: Set up the frontend service within a new Container Apps Environment.
- **Steps**:
    1. Create a Container App 'petstoreapp' and choose 'Sweden Central' as the region
    2. Create a new Container Apps Environment and select the provided VNet & subnet for your group and the Container Apps
    3. Deploy the `hello-world:latest` image instead of the one in the ACR 
    4. Create an ingress available on the VNet and set the port to 8080
    5. Set Virtual IP to Internal
    6. Assign a managed identity to the Container App
    7. Create a new Revision using the correct image from the ACR
    8. Open the petstoreapp in the browser following the link on the front page of the Container Apps

- **Links**:
    - [Azure Container Apps Documentation](https://learn.microsoft.com/en-us/azure/container-apps/)
    - [Quickstart: Deploy your first container app](https://learn.microsoft.com/en-us/azure/container-apps/quickstart-portal)
    - [Provide a virtual network to an internal Azure Container Apps environment](https://learn.microsoft.com/en-us/azure/container-apps/vnet-custom-internal?tabs=bash&pivots=azure-portal)

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

## Task 3.2: Creating the Backend Services

- **Objective**: Create backend services within the existing Container Apps Environment.
- **Steps**:
    1. Use the same environment to create backend services.
    2. Enable ingress limited to the Container Apps Environment (port 8080)
    3. Name services 'petservice', 'productservice', and 'orderservice' with their respective images.

The procedure to create the 3 backend services is very similar to creating the frontend service.
The difference is that we do **NOT** create a new Container Apps Environment every time.
Instead, we select the existing one.
For the backend services we also need to enable ingress.
However, they do not need to be accessible from the entire VNet.
Therefore, we make the Ingress Traffic 'Limited to Container Apps Environment'.
All backend services have the same target port of '8080'.
The procedure of first starting with the `hello-world:latest` image, assigning the managed identity, and
switching the image to the one from the ACR is identical.

## Task 3.3: Enable Dapr for Microservices Communication

- **Objective**: Enable Dapr for inter-service communication.
- **Steps**:
    1. Navigate to each service's settings in Azure Portal and enable Dapr.
    2. Configure the app port to '8080' and save.
    3. Figure out on which port is the Dapr sidecar listening on
    4. Give the services names: 'petstoreapp', 'petservice', 'productservice', and 'orderservice' 
   
- **Links**:
    - [Dapr Overview](https://dapr.io/)
    - [Enable Dapr in Azure Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/dapr-overview)

## Task 3.4: Configuring the communication between microservices

- **Objective**: Use Dapr to configure communication paths. Microservices must know where they 
can find other microservices. For this reason they are configured with environment variables with URLS to other
microservices. If Dapr sidecar is running, it can be reached with `http://localhost:<dapr_port>` where `dapr_port`
is from the previous exercise.
- **Configuration**:
    - Set `petstoreapp` environment variables:
    ```bash
    PETSTOREPETSERVICE_URL=http://localhost:<dapr_port>/<invoke_path_for_petservice>
    PETSTOREPRODUCTSERVICE_URL=http://localhost:<dapr_port>/<invoke_path_for_productservice>
    PETSTOREORDERSERVICE_URL=http://localhost:<dapr_port>/<invoke_path_for_orderservice>
    ```
    - Set `orderservice` environment variable for `productservice`:
    ```bash
    PETSTOREORDERSERVICE_URL=http://localhost:<dapr_port>/<invoke_path_for_orderservice>
    ```
     
- **Links**:
    - [How-To: Invoke services using HTTP](https://docs.dapr.io/developing-applications/building-blocks/service-invocation/howto-invoke-discover-services/#additional-url-formats)
    - [Manage environment variables on Azure Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/environment-variables?tabs=portal)

## Task 3.5: Scaling & load testing with jmeter

- **Objective**: Test application scaling with jmeter
- **Steps**:
    1. Use jmeter to apply load and observe scaling behavior

- **Links**:
    - [How-to Jmeter](https://www.geeksforgeeks.org/how-to-use-jmeter-for-performance-and-load-testing/).
    - [Azure Container Apps Autoscaling](https://learn.microsoft.com/en-us/azure/container-apps/scale-app)
    - [Apache JMeter](https://jmeter.apache.org/)
