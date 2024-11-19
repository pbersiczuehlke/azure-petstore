# Set up Azure & Create ACR
We can interact with Azure in multiple ways.
The most user-friendly way is to use the [Azure Portal](https://portal.azure.com).
However, not all functionality is available through the Portal.
To fully utilize Azure, we need to use a command line interface (CLI) tool.
The name of the CLI tool for Azure is `az`.
The first step to using `az` is to log in and choose a subscription.

## Output formatting
By default, Azure CLI outputs in the json format. 
This is usually too much information. 
For ease of use, we will first configure it to output with the *table* format.
However, the *table* output does hide a lot of information.
In the event that we need to see the entire output, we can add `--output json` to the end of any `az` command.
```bash
# set the output format to table
az config set core.output=table

# this will output in the table format
az config get defaults

# this will output in the json format
az config get defaults --output json
```
In this case, the amount of information does not differ between the two formats: *json* and *table*, 
but in most cases there is significant difference.


## Authenticating with Azure
To use the `az` command to interact with Azure, we need to log in and select a subscription on which we work.
The service principal accounts are distributed to you with the Vault.
We authenticate with:
```bash
az login --service-principal --username <service_principal_username> --password <service_principal_password> --tenant <tenant>
```

We can connect to multiple accounts and subscriptions with the `az` command. 
To manage the which account we are using, we can use:
```bash
# list accounts
az account list

# show current account
az account show

# to change to a different subscription
# replace <subscription_id> with the subscription id from the `az account list` command
az account set -s <subscription_id>
```

## Using the provided VNet
Due to the security setup, we are not allowed to expose any services to the internet.
This is enforced via Policies.
Therefore, we would fail to create any resources that are directly exposed to the internet.
To still then be able to create resources, we will deploy into an already existing VNet.

The provided VNet is already prepopulated with subnets.
The follow the following naming scheme:

| Name                                  | Purpose                    |
|---------------------------------------|----------------------------|
| snet-hackathon-cae-devlab-00[1-6]     | Container Apps Environment |
| snet-hackathon-cr-devlab-00[1-6]      | Container Registry         |
| snet-hackathon-default-devlab-00[1-6] | Everything else            |

Each group has 3 subnets: one for the Container Apps Environment, one for the Container Registry and 
one for everything else.

## Creating the Azure Container Registry
Docker images need to be stored somewhere before they are used.
For this purpose Azure provides the Azure Container Registry (ACR).
We will use the Azure Portal to create the ACR.
1. Navigate to the Portal, type 'Container Registries' into the search bar.
2. Click '+ Create'
3. Choose a Resource Group that your team has been assigned
4. Give it a name: `team<team_number>petstore`
5. Location: 'Sweden Central'
6. Pricing plan: 'Premium'
7. Click: 'Next: Networking >'
8. Select: 'Private access'
9. Click: 'Create a private endpoint connection'
10. Name: 'pe-acr' explanation: pe = private endpoint, acr = azure container registry
11. Virtual Network: provided one
12. Subnet: 'snet-hackathon-cr-devlab-00<team_number>'
13. Click: 'OK'
14. Next, Next
15. Add a tag. Name team, value <your team name>
16. Next
17. Create
 
We have now created a Registry which can be accessed by the Container Apps.

**Note:** Throughout this workshop we will create many Azure resources.
To keep things organized you should tag each resource with your team name like in the above example.

## Pushing local images to the ACR
We can push the images that we have built locally in the previous challenge to the ACR.
To push an image to a registry with docker we need to name it such that it contains the name of the registry.
For example: to upload the image `hello-world:latest` that we have locally to a registry at `team1petstore.azurecr.io`, 
we would tag the image with `team1petstore.azurecr.io/hello-world:latest`.
We can then push this image with `docker push`.
Before we can push an image to an ACR, we first need to log in into it.
This will store credentials on our machine so that the docker command can pull and push the images from this registry.

Example:
```bash
# list the container registries
az acr list

# log in to the appropriate ACR
az acr login -n <teamname>petstore

# creates a new image with the name <teamname>petstore.azurecr.io/hello-world:latest
# it doesn't copy anything, just creates a new alias
docker tag hello-world:latest <teamname>.azurecr.io/hello-world:latest

# we can then push this image to the registry
docker push <teamname>petstore.azurecr.io/hello-world:latest
```

## Building Petstore Images With Azure Build Agents
In addition to pushing the images directly, we can also use the `az acr build` command.
This works by uploading the entire context to a build agent on Azure, building the image and pushing the built image into
the ACR.
To build the images with `az`:
```bash
# pet backend service
cd petstore/petstorepetservice
az acr build -t petstorepetservice:0.1.0-az -r <teamname>petstore .

# order backend service
cd petstore/petstoreorderservice
az acr build -t petstoreorderservice:0.1.0-az -r <teamname>petstore .

# product backend service
cd petstore/petstoreproductservice
az acr build -t petstoreproductservice:0.1.0-az -r <teamname>petstore .

# frontend app service
cd petstore/petstoreapp
az acr build -t petstoreapp:0.1.0-az -r <teamname>petstore .
```

**Note:** This will NOT work with the private repository because the build agents are public.
A custom agentpool that is in the same VNet as the ACR would need to be created for this to work.

## Acceptance criteria
1. Container registry is created with a private endpoint
2. Images are built & uploaded to the registry