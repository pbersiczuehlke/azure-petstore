# Intro
Welcome to this Azure Fundamentals Workshop.
During this workshop we will deploy a Pet store application.

## Task 0: Team formation
Your first task of the day is to introduce yourselves and to pick a name for your team.

## Application Architecture
Presentation on the projector, described here.
The application is architected in the form of microservices.
The frontend service called `petstoreapp` connects to three backend services called `petservice`, 
`productservice`, and `orderservice`.

TODO: provide the architecture overview, show a demo

## Container App Service & Kubernetes
Explanation of how it works under the hood on the projector.

TODO: Described here what is presented.

## Team Azure Structure 
One subscription, separated with resource groups.
We will be working through the Azure portal to interact with Azure, mostly.

## Agenda
1. Choose your team name and understand what we are deploying
2. Build & Run the pet store locally (optional)
3. Explore Azure Portal
4. Set up Azure access
5. Create Azure Container Registry
6. Push the docker images into the registry
7. Create Container Apps Environment
8. Create Container App for the frontend microservice
9. Connect to the frontend service
10. Create Container Apps for a backend microservices
11. Scaling up and down
12. Email sending with Logic App

## Acceptance Criteria
1. You have chosen a name for your team
2. You have an understanding of what we are trying to deploy
3. You know your resource group

## Acknowledgement 
This workshop is an adaptation of the Azure Petstore project.
The Azure Petstore is a demo application provided by Microsoft to showcase various Azure services and capabilities.
You can learn more about it and access the resources at [Azure Petstore GitHub Repository](https://github.com/chtrembl/azure-cloud/tree/main). 
