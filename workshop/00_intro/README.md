# Intro
Welcome to this Azure Fundamentals Workshop.
During this workshop we will deploy a Pet store application.

## Task 0: Team formation
Your first task of the day is to introduce yourselves and to pick a name for your team.

## Application Architecture
Presentation on the projector, described here.
The application is architected in the form of microservices.
TODO: provide the architecture overview
TODO: show a demo

## Container App Service & Kubernetes
Explanation of how it works under the hood on the projector.
TODO: Described here what is presented.

## Team Azure Structure 
One subscription, separated with resource groups.
We will be working through the Azure portal to interact with Azure, mostly.

## Agenda
1. Build & Run the pet store locally
2. Set up azure access
3. Create azure container registry
4. Push the docker images into the registry - `docker push` or `az acr build`
5. Create Container Apps Environment
6. Create Container App for the frontend microservice
7. Create Container Apps for a backend microservices
8. Peer the VNets - container apps environment with the hub vnet
9. Configure the DNS zone to access the frontend microservice from the corporate network
10. Connect to the frontend service
11. Set up App Gateway in front of the frontend service
12. Set up Entra ID for the application
13. Enable Entra ID integration and redeploy
14. Load testing & scaling (scaling to 0)
15. Email sending with LogicApp
16. AI Chat - ChatGPT frontend

## Acceptance Criteria
1. You have chosen a name for your team
2. You have an understanding of what we are trying to deploy
3. You know your resource group

## Acknowledgement 
This workshop is an adaptation of the Azure Petstore project.
The Azure Petstore is a demo application provided by Microsoft to showcase various Azure services and capabilities.
You can learn more about it and access the resources at [Azure Petstore GitHub Repository](https://github.com/chtrembl/azure-cloud/tree/main). 
