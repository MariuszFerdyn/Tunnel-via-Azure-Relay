# Tunnel-via-Azure-Relay
This repository provides a robust solution for establishing fast and reliable tunnels between two machines over a public network using HTTPS on port 443. Leveraging Azure Relay, it enables seamless remote access similar to popular remote desktop applications like AnyDesk or TeamViewer. Users can connect securely without the need for complex VPN setups, making it ideal for accessing devices behind firewalls or NATs. The solution supports various protocols, ensuring flexibility for different use cases.
# Deploy a Hybrod connection
```
#!/bin/bash
##az login #--use-device-code

# Define variables
subscriptionId="your-subscription-id"
webAppName="hybrid-proxy"
resourceGroupName="${webAppName}-rg"
relay="${webAppName}relay"
location="your-desired-location"

# Set the active subscription
az account set --subscription $subscriptionId

# create resource group
az group create --name $resourceGroupName --location $location
# create relay namespace
az relay namespace create -g $resourceGroupName --name $relay --location $location
# create the hybrid connection endpoint 'db'
az relay hyco create -g $resourceGroupName --namespace-name $relay --name db
# grant the current user "owner" permission
# Create the secrets
az relay hyco authorization-rule create -g $resourceGroupName --hybrid-connection-name db --namespace-name $relay -n send --rights Send
az relay hyco authorization-rule create -g $resourceGroupName --hybrid-connection-name db --namespace-name $relay -n listen --rights Listen

az relay hyco authorization-rule keys list --hybrid-connection-name db --namespace-name $relay -g $resourceGroupName -n send
az relay hyco authorization-rule keys list --hybrid-connection-name db --namespace-name $relay -g $resourceGroupName -n listen
```
