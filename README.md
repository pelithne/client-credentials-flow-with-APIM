# daemon-to-apim-to-backend
This step-by-step describes how to setup a daemon app that sends requests via APIM to a REST API backend. 

The "architecture" for this tutorial case is something like the following

Automatic Daemon (With Azure Identity) -> APIM -> Backend REST API -> Operation 1
                                                                   -> Operation 2
                                                                   -> Operation n

Depending on the identity of the Daemon, the APIM will allow access to different parts of the backend API (different operations)

## Create a Resource Group

Start by log in in to the Azure portal, and make sure you are in your "Home", https://portal.azure.com/#home

Click on "Create a Resource"

<p align="left">
  <img width="40%"  src="./media/create-a-resource.png">
</p>

In the search field, search for "Resource Group" and select Resource Group from the search results. Then click the "Create" button to start the creation of the Resource Group (RG). Give the RG a nice name, and make sure its placed in a Region close to you.

<p align="left">
  <img width="60%"  src="./media/create-a-resource-group.png">
</p>

Then click review and create. Validation should pass, after which you can click on create.


## Create an API with APIM

Azure API Manager, is a platform that can hold API definitions. The APIs are not hosted in APIM, instead it points to backend APIs, which could be running on Azure, on-prem, in another cloud or anywhere else you have connectivity to.

Start by going to your resource group, if you are not already there. Click on "Create Resources" (or "Add") and search for APIM in the search field. Select API Management from the search results, then click create.

Give your APIM a globally unique name. This is needed because the name will be used to create a URL that needs to be a Fully Qualified Domain Name, FQDN. 

Make sure that the APIM is located in the right subscription and in the resource group you just created. 

Add an "Organization name" of your choice and an "Administrator email". 

**Make sure** to use the "Developer" pricing tier. The developer tier gives you full functionality but without a Service Level Agreement, and is much cheaper than the other alternatives.

<p align="left">
  <img width="50%"  src="./media/create-apim.png">
</p>

Now wait. It can take a while to create the APIM instance, up to 40 minutes at the time of writing (May 2020)

## Create Application Registrations

Both the API and the Daemon needs to be registered in Azure AD, so that we can use Oauth2 for authentication. We start with the API.

Search for "App registrations" and select App Registrations from the search results. Name the registration appropriately and leave the defaults and click "Register".

<p align="left">
  <img width="50%"  src="./media/api-app-registration.png">
</p>


In the left hand navigation pane, go to "Expose an API", then click on "Application ID URI - Set", and leave the default value, which should look similar to ````api://7f038808-5322-4125-8143-12d804a45c1b````. The alphanumeric string is the clientID. **Make a note of this** as it will be needed later.

Now, create another app registration for the daemon. Give it a name, and leave the defaults then click "Register".

Now, we need to create a secret for the daemon. In the left hand navigation pane, go to "Certificate & Secrets", then select "New Client Secret". Give it a name and choose an expiration time (I use 1 year).

Copy the secret and store it safely. You will not be able to see it again in the portal.    

Also, make a note of the clientID, which can be found in the "Overview" from the left hand navigation pane.

Use e.g. postman to try if you get a response from your token endpoint

<p align="left">
  <img width="80%"  src="./media/postman.png">
</p>

You should get a response similar to the (slightly redacted) output in the picture above.









## Create the Daemon
The Daemon app will be a simple Python-thingy that authenticates towards Azure AD using the client credentials flow (https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow)

In order for the Daemon to get an identity, you need to register it in Azure AD. For background information about this, feel free to have a look at this page: https://docs.microsoft.com/en-us/azure/active-directory/develop/scenario-daemon-app-registration

First, search for "App Registrations" in the search bar, and select "App Registrations". The click in the plus-sign to the left to add a new Registration

<p align="left">
  <img width="50%"  src="./media/new-registration.png">
</p>


Give the app a name (I called it daemon-app), select the single tenant account type and click on the "Register" button (There is no need for a Redict URI, since we will only be using the the client credentials flow which does not involve any call-back).

<p align="left">
  <img width="60%"  src="./media/new-registration2.png">
</p>

Next, use the left hand navigation pane to go to **API Permissions**, then select "Add a permission"

<p align="left">
  <img width="20%"  src="./media/permission.png">
</p>

