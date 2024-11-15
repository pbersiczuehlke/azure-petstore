# Set up Azure Container Apps

Our microservices will be deployed using Azure Container Apps, a serverless PaaS built on top of Kubernetes.

## Task 3.1: Creating the Frontend Service

- **Objective**: Set up the frontend service within a new Container Apps Environment.
- **Steps**:
    1. Create a Container App 'petstoreapp' and choose 'Switzerland North' as the region.
    2. Create a new Container Apps Environment and select the provided VNet.
    3. Create a subnet named 'sn-<teamname>-ca' with `/27` size.
    4. Set Virtual IP to Internal
    5. Configure the Container App to use your Azure Container Registry (ACR).

- **Links**:
    - [Azure Container Apps Documentation](https://learn.microsoft.com/en-us/azure/container-apps/)
    - [Quickstart: Deploy your first container app](https://learn.microsoft.com/en-us/azure/container-apps/quickstart-portal)
    - [Provide a virtual network to an internal Azure Container Apps environment](https://learn.microsoft.com/en-us/azure/container-apps/vnet-custom-internal?tabs=bash&pivots=azure-portal)

## Task 3.2: Creating the Backend Services

- **Objective**: Create backend services within the existing Container Apps Environment.
- **Steps**:
    1. Use the same environment to create backend services.
    2. Enable ingress limited to the Container Apps Environment.
    3. Name services 'petservice', 'productservice', and 'orderservice' with their respective images.

## Task 3.3: Enable Dapr for Microservices Communication

- **Objective**: Enable Dapr for inter-service communication.
- **Steps**:
    1. Navigate to each service's settings in Azure Portal and enable Dapr.
    2. Configure the app port to '8080' and save.
    3. Figure out on which port is the Dapr sidecar listening on

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

- **Objective**: Test application scaling with jmeter.
- **Steps**:
    1. Use jmeter to apply load and observe scaling behavior.

- **Links**:
    - [Azure Container Apps Autoscaling](https://learn.microsoft.com/en-us/azure/container-apps/scale-app)
    - [Apache JMeter](https://jmeter.apache.org/)
