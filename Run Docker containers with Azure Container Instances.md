#### 


#### Exercise - Run Azure Container Instances

1. Create a new resource group with the name learn-deploy-aci-rg

```shell script
az group create --name learn-deploy-aci-rg --location eastus
```

```shell script
DNS_NAME_LABEL=aci-demo-$RANDOM
```

2. start a container instance

```shell script
az container create \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer \
  --image microsoft/aci-helloworld \
  --ports 80 \
  --dns-name-label $DNS_NAME_LABEL \
  --location eastus
```

3. check its status

```shell script
az container show \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer \
  --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" \
  --out table
```

#### Exercise - Control restart behavior
     
1. start the container

```shell script
az container create \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer-restart-demo \
  --image microsoft/aci-wordcount:latest \
  --restart-policy OnFailure \
  --location eastus
```

2. Run az container show to check your container's status. It should be in "Terminated" state
   
```shell script
az container show \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer-restart-demo \
  --query containers[0].instanceView.currentState.state
```

3. View the container's logs to examine the output. 

```shell script
az container logs \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer-restart-demo
```

#### Exercise - Use data volumes

1. store a unique name in a Bash variable.

```shell script
STORAGE_ACCOUNT_NAME=mystorageaccount$RANDOM
```

2. create your storage account

```shell script
az storage account create \
  --resource-group learn-deploy-aci-rg \
  --name $STORAGE_ACCOUNT_NAME \
  --sku Standard_LRS \
  --location eastus
```

3. place the storage account connection string into an environment variable named AZURE_STORAGE_CONNECTION_STRING

```shell script
export AZURE_STORAGE_CONNECTION_STRING=$(az storage account show-connection-string \
  --resource-group learn-deploy-aci-rg \
  --name $STORAGE_ACCOUNT_NAME \
  --output tsv)
```

4. create a file share, named aci-share-demo, in the storage account.

```shell script
az storage share create --name aci-share-demo
```

#### Get storage credentials

1. get the storage account key.

```shell script
STORAGE_KEY=$(az storage account keys list \
  --resource-group learn-deploy-aci-rg \
  --account-name $STORAGE_ACCOUNT_NAME \
  --query "[0].value" \
  --output tsv)
``` 

#### Deploy a container and mount the file share

1. create a container that mounts /aci/logs/ to your file share.

```shell script
az container create \
  --resource-group learn-deploy-aci-rg \
  --name aci-demo-files \
  --image microsoft/aci-hellofiles \
  --location eastus \
  --ports 80 \
  --ip-address Public \
  --azure-file-volume-account-name $STORAGE_ACCOUNT_NAME \
  --azure-file-volume-account-key $STORAGE_KEY \
  --azure-file-volume-share-name aci-share-demo \
  --azure-file-volume-mount-path /aci/logs/
```

2. get your container's public IP address.

```shell script
az container show \
  --resource-group learn-deploy-aci-rg \
  --name aci-demo-files \
  --query ipAddress.ip \
  --output tsv
```

3. display the files that are contained in your file share

```shell script
az storage file list -s aci-share-demo -o table
```

4. download a file to your Cloud Shell session. Replace <filename> with one of the files that appeared in the previous step.

```shell script
az storage file download -s aci-share-demo -p <filename>
```

```shell script
cat <filename>
```

#### Exercise - Troubleshoot Azure Container Instances

1. create a basic container

```shell script
az container create \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer \
  --image microsoft/sample-aks-helloworld \
  --ports 80 \
  --ip-address Public \
  --location eastus
```

2. Get logs from your container instance

```shell script
az container logs \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer
```

3. Get container events

```shell script
az container attach \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer
```

4. Execute a command in your container

```shell script
az container exec \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer \
  --exec-command /bin/sh
```

5. Monitor CPU and memory usage on your container

```shell script
CONTAINER_ID=$(az container show \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer \
  --query id \
  --output tsv)
```

6. Run the az monitor metrics list command to retrieve CPU usage information.
   
```shell script
az monitor metrics list \
  --resource $CONTAINER_ID \
  --metric CPUUsage \
  --output table
```

7. Run this az monitor metrics list command to retrieve memory usage information.
   
```shell script
az monitor metrics list \
  --resource $CONTAINER_ID \
  --metric MemoryUsage \
  --output table
```

#### Clean up resources

```shell script
az group delete --name learn-deploy-aci-rg
```
