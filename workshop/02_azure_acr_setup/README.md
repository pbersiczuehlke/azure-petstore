# Set up Azure & Create ACR

We can use the [Azure Portal](https://portal.azure.com) or the Azure CLI (`az`) to interact with Azure.
The CLI provides full functionality, so if `az` isn't installed, you'll need to add it to your system.
Start by logging in and selecting a subscription.

## Task 2.1: Set Up Azure CLI

- **Objective**: Install and configure the Azure CLI (`az`) for interacting with Azure services.
- **Steps**:
    1. Install the Azure CLI if it's not already installed.
    2. Log in using `az login`.
    3. Configure the output format to `table` for ease of use.
- **Links**:
    - [Install the Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)
    - [Output formats for Azure CLI commands](https://learn.microsoft.com/en-us/cli/azure/format-output-azure-cli?tabs=bash)

## Task 2.2: Authenticate with Azure

- **Objective**: Authenticate and manage subscriptions using the Azure CLI.
- **Steps**:
    1. Use `az login` to authenticate with Azure.
    2. List available accounts using `az account list`.
    3. Set the active subscription with `az account set -s <subscription_id>`.
- **Links**:
    - [Interactive login](https://learn.microsoft.com/en-us/cli/azure/authenticate-azure-cli-interactively#interactive-login)
    - [Manage Azure subscriptions with Azure CLI](https://learn.microsoft.com/en-us/cli/azure/manage-azure-subscriptions-azure-cli?tabs=bash)

## Task 2.3: Use the Provided VNet

- **Objective**: Deploy resources within an existing Virtual Network (VNet) to comply with security policies.
- **Steps**:
    1. Navigate to 'Virtual networks' in the Azure Portal.
    2. Select the provided VNet.
    3. Create a new subnet for your team:
        - Change the name to `sn-<teamname>`.
        - Ensure the size is set to `/24`.
- **Links**:
    - [Virtual network and subnets in Azure](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview)
    - [Create a virtual network and subnet](https://learn.microsoft.com/en-us/azure/virtual-network/quick-create-portal)

## Task 2.4: Create Azure Container Registry

- **Objective**: Set up an Azure Container Registry (ACR) for storing Docker images.
- **Steps**:
    1. In the Azure Portal, create a new ACR.
    2. Name it `<teamname>petstore`.
    3. Set the location to 'Switzerland North'.
    4. Configure it to use the subnet `sn-<teamname>`.
- **Links**:
    - [Quickstart: Create a private container registry using the Azure portal](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-get-started-portal)
    - [Use private endpoints for Azure Container Registry](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-private-link)

## Task 2.5: Push Local Images to ACR (Optional)

- **Objective**: Tag and push Docker images built locally to the ACR.
- **Steps**:
    1. Log in to the ACR using `az acr login`.
    2. Tag local Docker images with the ACR URL.
    3. Push the images to the ACR.
- **Links**:
    - [Push and pull Docker images](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-get-started-docker-cli)
    - [Authenticate with Azure Container Registry](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-authentication)

## Task 2.6: Build Petstore Images Using ACR

- **Objective**: Build and upload Docker images directly using Azure's build capabilities.
- **Steps**:
    1. Use `az acr build` to build and push images for each Petstore service to the ACR.
- **Links**:
    - [Build container images in the cloud with ACR Tasks](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-tasks-overview)
    - [Quickstart: Build and push image with Azure CLI](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-tutorial-quick-task)

## Acceptance Criteria

1. The container registry is created with a private endpoint.
2. Images are built and uploaded to the registry.
