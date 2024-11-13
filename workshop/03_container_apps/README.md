# Set up Azure Container Apps
Our microservices will be deployed using Azure Container Apps.
All Container Apps reside inside a Container Apps Environment.
We will first create the Environment, during the creation of our first microservice.

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
10. Choose 'vnet-petstore' for the virtual network
11. Choose 'sn-ca' for the subnet
12. Set Virtual IP to Internal
13. Click 'Create'
14. Click 'Next: Container >'
15. Choose the ACR that you created
16. Choose 'petstorepetservice' for the image name and '0.1.0' for the image tag
17. Click 'Next: Ingress >'
18. Select 'Limited to VNet'
19. Set Target port to '8080'
20. Click 'Next'
21. Add your team tags
22. Click 'Create'

We have now created the Frontend service.


