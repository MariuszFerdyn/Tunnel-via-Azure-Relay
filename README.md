# Tunnel-via-Azure-Relay
This repository provides a robust solution for establishing fast and reliable tunnels between two machines over a public network using HTTPS on port 443. Leveraging Azure Relay, it enables seamless remote access similar to popular remote desktop applications like AnyDesk or TeamViewer. Users can connect securely without the need for complex VPN setups, making it ideal for accessing devices behind firewalls or NATs. The solution supports various protocols, ensuring flexibility for different use cases.
# Deploy a Hybrod connection
```
#!/bin/bash
##az login #--use-device-code

# Define variables
subscriptionId="your-subscription-id"
relytest="rely-test-proxy"
resourceGroupName="${relytest}-rg"
relay="${relytest}relay"
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
# Create Second VM
```
adminUsername="adminuser"
adminPassword="YourPassword123!"
vmSize="Standard_B2s"
az vm create --resource-group $resourceGroupName --name "${relytest}" --image Win2019Datacenter --public-ip-sku Standard --admin-username $adminUsername --admin-password $adminPassword --size $vmSize --location $location
```

# On both machines
Install [https://github.com/Azure/azure-relay-bridge/releases - download unzip](https://github.com/Azure/azure-relay-bridge/releases)

# On server machine with website
server.config:
```
RemoteForward :
   - RelayName: db
     Host: 127.0.0.1
     PortName: http
     HostPort: 80
     ConnectionString: Endpoint=sb://rely-test-proxyrelay.servicebus.windows.net/;SharedAccessKeyName=root;SharedAccessKey=xxx;EntityPath=db

LogLevel: INFO
```
## On server machine RDP example
```
RemoteForward :
   - RelayName: finalproxy
     Host: 127.0.0.1
     PortName: rdp
     HostPort: 3389
     ConnectionString: Endpoint=sb://rely-test-proxyrelay.servicebus.windows.net/;SharedAccessKeyName=root;SharedAccessKey=xxx;EntityPath=db

LogLevel: INFO
```
# Start the server
Execute:
```
azbridge -f .\server.config
```
# On client machine 
client.config:
```
LocalForward :
  - BindAddress: 10.0.0.4
    BindPort: 8080
    PortName: http
    RelayName: db
    ConnectionString: Endpoint=sb://rely-test-proxyrelay.servicebus.windows.net/;SharedAccessKeyName=root;SharedAccessKey=xxx;EntityPath=db

LogLevel: INFO
```
## For RDP file example
cleint.config:
```
LocalForward :
  - BindAddress: 127.0.0.1
    BindPort: 8181
    PortName: rdp
    RelayName: finalproxy
    ConnectionString: Endpoint=sb://finalproxy.servicebus.windows.net/;SharedAccessKeyName=finalproxy;SharedAccessKey=xxx=;EntityPath=finalproxy

LogLevel: INFO
```
## For RDP command example
```
azbridge -L 127.0.0.1:8181/rdp:finalproxy -x "Endpoint=sb://finalproxy.servicebus.windows.net/;SharedAccessKeyName=finalproxy;SharedAccessKey=xxx=;EntityPath=finalproxy"
```
## For RDP docker example
```
docker run --name azbridge-proxy -p 8181:8181 azbridge:0.15 -L 0.0.0.0:8181/rdp:finalproxy -x "Endpoint=sb://finalproxy.servicebus.windows.net/;SharedAccessKeyName=finalproxy;SharedAccessKey=xxx;EntityPath=finalproxy"
```
# Start the client
Execute:
```
azbridge -f .\client.config
```
execute
```
start http://10.0.0.4:8080/
```
The site should be loaded
