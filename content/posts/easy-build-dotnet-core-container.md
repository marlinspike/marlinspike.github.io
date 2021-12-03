---
title: "Easily Build Dotnet Core Container"
date: 2019-03-26T14:30:38-05:00
draft: false
toc: false
images:
tags:
  - docker
  - azure
  - dotnet
---
Whether you're a Docker pro or just learning about it, you're doubtless come across scenarios for migrating demo apps to a Docker container running your development box. That's fine, but how about getting some more mileage and seeing it run in the cloud, as a PaaS app, where you've now taken advantage of Docker goodness to abstract away environment prep, and added Platform goodness by abstracting away all envrionment management too.

Lets dive right into it with a short and sweet recipe for all  your Dotnet Core apps!

The tools we're going to use:

- Docker - Create the Docker image. We're going to use multi-stage builds, so you'd need to be on Docker 17.05 or later 
- Azure Container Registry - Host the Docker image in Azure. You can use Docker Hub too, just change the appropriate URLs
- Azure Container Instance - Simple, easy, no-handlebars service to run any Docker container in Azure.

#### Step 1: Containerize the Donet Core App

At the root directory of your dotnet core app, create a text file called simply "Dockerfile". This is the file that's used to instruct Docker Build to create a custom image based upon various other layers we'll provide. Here's the Dockerfile:

````
FROM microsoft/dotnet:sdk AS build-env
WORKDIR /app

# Copy csproj and restore as distinct layers
COPY *.csproj ./
RUN dotnet restore

# Copy everything else and build
COPY . ./
RUN dotnet publish -c Release -o out

# Build runtime image
FROM microsoft/dotnet:aspnetcore-runtime
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "BasicWebApp.dll"]
````
 

 

#### Step 2: Create the image

Still at the root directory where your Dockerfile was created, run the following command to create your image. Replace <REGISTRY_NAME> with a name of your choosing. You'll need that name for the next step as well

`az acr build --registry <REGISTRY_NAME> --image basicwebapp:v2 .`

 

#### Step 3: Launch an Azure Container Instance based on the image

You'll need the following pieces of information for this step:

- Registry Name from the previous step
- Registry username
- Registry password
- DNS Label: Any name you'd like to give your container, so that you can navigate to it by the following URI - pattern: https://<dns-label>.azurecr.io
`az container create --name bwacontainer --resource-group Registry --image <REGISTRY_NAME>.azurecr.io/basicwebapp:v2 --registry-login-server <REGISTRY_NAME>.azurecr.io --registry-username <REGISTRY_USERNAME> -n bwacontainer --registry-password <REGISTRY_PASSWORD>  --dns-name-label <DNS_LABEL>`

 

#### Step 4: Launch the site

Navigate to the url: https://<dns_label>.azurecr.io
