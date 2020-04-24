### Build RStudio Server by using ACR and ACI

#### Exercise - Deploy Azure Container Registry
1. Create a new resource group with the name learn-deploy-acr-rg

```shell script
az group create --name learn-deploy-acr-rg --location <choose-a-location>
```

2. Before running the command, replace <registry-name> with the unique name you want to give your new container registry. 

```shell script
ACR_NAME=<registry-name>
```

3. create our new container registry

```shell script
az acr create --resource-group learn-deploy-acr-rg --name $ACR_NAME --sku Premium
```
#### Exercise - Build container images with Azure Container Registry Tasks

1.  create a new file named Dockerfile. You can use any text editor to edit the file. 

```shell script
FROM rocker/rstudio-stable:devel
EXPOSE  8787
```

2. build the container image from the Dockerfile.

```shell script
az acr build --registry $ACR_NAME --image rstudio:v1 .
```

3. verify that the image has been created and stored in the registry.

```shell script
az acr repository list --name $ACR_NAME --output table
```

#### Exercise - Deploy images from Azure Container Registry

1. enable the admin account on your registry.

```shell script
az acr update -n $ACR_NAME --admin-enabled true
```

2. retrieve the username and password for the admin account you enabled in the preceding step.

```shell script
az acr credential show --name $ACR_NAME
```

3. deploy a container instance. Replace <username>,<password> in the following command with your registry's admin username and password. Replace <location> with the location value returned when you created the container registry earlier.

```shell script
az container create \
    --resource-group learn-deploy-acr-rg \
    --name acr-tasks \
    --image $ACR_NAME.azurecr.io/helloacrtasks:v1 \
    --registry-login-server $ACR_NAME.azurecr.io \
    --ip-address Public \
    --location <location> \
    --registry-username [username] \
    --registry-password [password] \
    --environment-variables PASSWORD=pavan \
    --ports 8787
```

4. Get the IP address of the Azure container instance

```shell script
az container show --resource-group  learn-deploy-acr-rg --name acr-tasks --query ipAddress.ip --output table
```

#### Open RStudio in Browser

http://localhost:8787

#### Clean up resources

1. Delete resource group

```shell script
az group delete --name learn-deploy-acr-rg
```