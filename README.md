# AWS-Scheduling-DB-Backups

## Project Overview

This Azure architecture uses containers to create an App Service plan, an HTTP service that hosts web apps, using Linux, webapps, Docker containers, and Azure Cloud Shell. Cloud Shell is used to create the Linux App Service plan, and the web app, which uses the container resource. This architecture can be used to efficiently create and deploy webs apps or update existing applications.

![diagram](/img/diagram.png)

### Resources Used:

- Azure Cloud Shell
- Azure App Service (Linux)
- Docker Container

## Procedure:

**Use Cloud Shell to Create a Linux App Service Plan**
- Right-click the Azure Portal tab in your browser and choose the Duplicate Tab option. In this new tab, open the Azure Cloud Shell using the icon in the top menu.
- Choose PowerShell when you are prompted. 
- With these options configured, click Create Storage.
- Copy the PowerShell commands below and paste into the Cloud Shell window:
```markup
$resourceGroup = az group list --query '[0].name' --output json
$appServicePlan = 'Linux-App-ServicePlan'
az appservice plan create -g $resourceGroup -n $appServicePlan --is-linux --number-of-workers 1 --sku B1
```

**Use Cloud Shell to Create a Web App Using a DockerHub Container Image**
Copy the PowerShell commands below and paste them into the Cloud Shell window:
```markup
$app = 'LinuxDockerApp' + (Get-Date).ticks

az webapp create --resource-group $resourceGroup --plan $appServicePlan --name $app --deployment-container-image-name microsoft/dotnet-samples:aspnetapp
```

We can verify this by navigating to our Azure Portal tab. Click All Resources and then click on our Docker App Service in the list of resources. In the Settings section, click Container settings and note the Image source is set to Docker Hub.

![one](/img/one.png) 

**Use Cloud Shell to Update a Web App Container Image from DockerHub to GitHub**
Copy the PowerShell command below and paste it into the Cloud Shell window:
```markup
az webapp config container set --resource-group $resourceGroup --name $app --docker-registry-server-url 'https://github.com/dotnet/dotnet-docker/tree/master/samples/aspnetapp'
```

We can verify that this command ran successfully by navigating back to the Azure Portal. Refresh the Container settings page. 

Our Image source now shows as Private Registry and the Server URL is the same URL we provided in our command.

![two](/img/two.png) 

We can verify that our site is running by clicking Overview in the App Service menu. Our endpoint is shown in the URL section on this page. Click on this URL to load our app and verify that it is running.

![three](/img/three.png) 
