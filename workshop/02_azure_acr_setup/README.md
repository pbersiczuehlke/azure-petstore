# Set up Azure & Create ACR

We can use the [Azure Portal](https://portal.azure.com) or the Azure CLI (`az`) to interact with Azure.
However, not all functionality is available through the Portal.
To fully utilize Azure, we need to use a command line interface (CLI) tool.
The name of the CLI tool for Azure is `az`.
The first step to using `az` is to log in and choose a subscription.

## Task 2.1: Set Up Azure CLI

- **Objective**: Configure the Azure CLI (`az`) for interacting with Azure services
- **Steps**:
    1. Log in using `az login` using the provided service principal
    2. Configure the output format to `table` for ease of use.
    3. List available accounts using `az account list`.
    4. Check current subscription with `az account show`.
- **Links**:
    - [Sign into Azure with a service principal using the Azure CLI](https://learn.microsoft.com/en-us/cli/azure/authenticate-azure-cli-service-principal)
    - [Output formats for Azure CLI commands](https://learn.microsoft.com/en-us/cli/azure/format-output-azure-cli?tabs=bash)
    - [Manage Azure subscriptions with Azure CLI](https://learn.microsoft.com/en-us/cli/azure/manage-azure-subscriptions-azure-cli?tabs=bash)

## Using the provided VNet
Due to the security setup, we are not allowed to expose any services to the internet.
This is enforced via Policies.
Therefore, we would fail to create any resources that are directly exposed to the internet.
To still then be able to create resources, we will deploy into an already existing VNet.

The provided VNet is already prepopulated with subnets.
They follow the following naming scheme:

| Name                                  | Purpose                    |
|---------------------------------------|----------------------------|
| snet-hackathon-cae-devlab-00[1-6]     | Container Apps Environment |
| snet-hackathon-cr-devlab-00[1-6]      | Container Registry         |
| snet-hackathon-default-devlab-00[1-6] | Everything else            |

Each group has 3 subnets: one for the Container Apps Environment, one for the Container Registry and
one for everything else.

## Task 2.2: Create Azure Container Registry

- **Objective**: Create and set up an Azure Container Registry (ACR) for storing Docker images
- **Steps**:
    1. In the Azure Portal, create a new ACR
    2. Name it `team<team_number>petstore`
    3. Set the location to 'Sweden Central'
    4. Configure it to the appropriate subnet
- **Links**:
    - [Quickstart: Create a private container registry using the Azure portal](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-get-started-portal)
    - [Use private endpoints for Azure Container Registry](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-private-link)

## Task 2.3: Push Local Images to ACR

- **Objective**: Tag and push Docker images built locally to the ACR.
- **Steps**:
    1. Log in to the ACR using `az acr login`
    2. Tag local Docker images with the ACR URL
    3. Push the Petshop images to the ACR
- **Links**:
    - [Push and pull Docker images](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-get-started-docker-cli)
    - [Authenticate with Azure Container Registry](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-authentication)

## Task 2.4: Build Petstore Images Using ACR (Demonstrative, Optional)

- **Objective**: Build and upload Docker images directly using Azure's build capabilities.
- **Steps**:
    1. Use `az acr build` to build and push images for each Petstore service to the ACR.
- **Links**:
    - [Build container images in the cloud with ACR Tasks](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-tasks-overview)
    - [Quickstart: Build and push image with Azure CLI](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-tutorial-quick-task)

## Acceptance Criteria

1. The container registry is created with a private endpoint.
2. Images are built and uploaded to the registry.
