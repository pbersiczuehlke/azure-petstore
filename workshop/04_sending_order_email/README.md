# Sending Order Email
When our order is placed, we wish to email the store worker about the order, so that they can start preparing it.
For this purpose we will use a Logic App.
Azure Logic Apps is a cloud platform where you can create and run automated workflows with little to no code.
They belong to the so-called low-code category.

## Communication Service
In order to send emails, we need to have an email account.
Luckily, Azure Communication Service provides us with one without having to provide our own domain.
We will use this free email domain to send the email from the Logic App.

First, we need to create the Communication Service:
1. Navigate to the Portal and type Communication Services
2. Choose your resource group
3. Name it `<teamname>cs`
4. Choose `United States` for data location **Note: Free email domain is only available with the US data location**
5. Create it

To set up a free domain:
1. Open the created Communication Service -> Settings -> Try Email
2. Set up a free azure subdomain
3. Email Service -> Add an email service
4. Set Name to `<teamname>es`
5. Keep other settings & Create
6. Use the try email form to email your personal corporate email, check the spam folder

To access this Communication Service from the Logic Apps, we will use Key which can be found under Settings.

## Service Bus
In order for the Logic App to start, it needs to be triggered.
We will configure our `orderservice` to push a message to the message broker service called Azure Service Bus.
Message brokers are services that facilitate passing messages between different applications or between the 
components of the same applications acting as middlemen.
In our case we will use the Service Bus to facilitate the communication between our web application and the Logic App.

To create a Service Bus:
1. Navigate to the Portal and type Service Bus
2. Click +Create
3. Choose your resource group
4. Namespace name: `<teamname>petstore`
5. Location: Switzerland North
6. Pricing tier: Premium
7. Click Next, Next
8. Go to Networking -> Private Access
9. Click + Private Endpoint
10. Choose your resource group
11. Name `pe-sb`
12. Choose the provided VNet
13. Choose the `sn-<teamname>` subnet
14. Create it

Once the Service Bus is created:
1. Press + Topic
2. Name it `orders`
3. Create it
4. On the left-hand side: Select Topics
5. Click on `orders`
6. On the left-hand side: Select Subscriptions
7. Click + Subscription
8. Name it `email`
9. Set Max delivery count to 1
10. Create it

We now have a topic into which we will push our messages and a subscriber that will be used by our Logic App.

To access the topic from the Logic App, we need to generate some keys.
1. Go to the `orders` Topic
2. Go to Shared access policies
3. Click + Add
4. Name it `logicapp`
5. Give it Listen claim

Create another one in the same way, but call it `orderservice` and give it the Send claim.

## Logic App
The job of the Logic App is to take a message from the service bus and use it to send an email.

To create a logic app:
1. Navigate to the Azure Portal -> Search Logic Apps
2. Click +Add
3. Choose Consumption  TODO: VNet communication?
4. Name `orderemail`
5. Region: Switzerland North
6. Create

To create the most basic Logic App that simply sends 'New Order Received' to your email:
1. On the left-hand side select Logic app designer
2. Click 'Add a trigger' in the middle of the screen
3. Search for Service Bus -> When a message is received in a topic subscription 
4. You will create a connection using the Keys provided on the service bus `logicapp` Shared Access Policy
5. Open the `logicapp` Policy and copy the primary connection string
6. Paste it in the Connection String field on the Logic App
7. Name the connection `email`
8. Set Topic name to `orders`
9. Set Subscription to `email`
10. Click on the + icon under the service bus
11. Search for Azure Communication Email -> Send Message
12. Connection name: my-connection
13. Copy and paste the primary connection string from the Communication Service above
14. Set the to email address to your personal corporate email
15. Subject: New Order 
16. Body: 'New Order Received'
17. Click away from the right-hand screen to the middle of the page and click Save in the upper left corner

To check if the logic app works:
1. Go to the Service Bus -> Networking and enable Public Access (PROBLEM!)
2. Try to trigger the app.
3. Go to the `orders` topic & click Send Message.
4. Type in any message any press send
5. An email should arrive in your inbox, check the spam folder. It might take 3-5 minutes.

## Connecting orderservice with the Service Bus
In order to achieve this, we need to comment to make some configuration changes to the `orderservice`.
Open the `petstoreorderservice/src/main/resources/application.yml`.
Comment lines 17,18. Uncomment lines 19-23.
Rebuild the application and give it a new tag: `0.2.0-email`, push this to the ACR.
We can now go to the Container Apps and create a new revision.
1. Go to the `orderservice` app service
2. Click Containers on the left-hand side
3. Click Edit and redeploy
4. Select the orderservice and click Edit
5. Change the image tag to `0.2.0-email`
6. Go to the environment variables and set two new variables: `PETSTOREORDERSERVICE_EMAIL_TOPIC_CONNECTION_STRING`, and `PETSTOREORDERSERVICE_SUBSCRIPTION_ID`.
The new env vars need to be set with the connection string for the topic that we created previously, the `orderservice` Shared Access Policy.
The value for the subscription id can be found in the Service Bus Namespace itself (the front page of the Service Bus resource).
7. Click Save and Create

An email should arrive now every time someone completes an order.