# daemon-to-apim-to-backend
THis Stephby-step describes how to setup a daemon app that sends requests via APIM to a REST API backend. 

The "architecture" for this tutorial case is something like the following

Automatic Daemon (With Azure Identity) -> APIM -> Backend REST API -> Operation 1
                                                                   -> Operation 2
                                                                   -> Operation n

Depending on the identity of the Daemon, the APIM will allow access to different parts of the backend API (different operations)

## Create the Daemon
The Daemon app will be a simple Python-thingy that authenticates towards Azure AD using the client credentials flow (https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow)

In order for the Daemon to get an identity, you need to register it in Azure AD. For background information about this, feel free to have a look at this page: https://docs.microsoft.com/en-us/azure/active-directory/develop/scenario-daemon-app-registration

First, search for "App Registrations" in the search bar, and select "App Registrations". The click in the plus-sign to the left to add a new Registration
