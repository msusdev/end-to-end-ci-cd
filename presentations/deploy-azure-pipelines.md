---
title: 'Deploy your code to Azure with Azure Pipelines'
author: 'Part 3 of 3 in the [Build end-to-end CI/CD capabilities with Azure Pipelines](https://github.com/msusdev/end-to-end-ci-cd) series'
---

## About the presenter

Presenter Name

☁️ *Presenter Title*

> For questions or help with this series: <msusdev@microsoft.com>

All demos and source code available on **GitHub**:

> [github.com/msusdev/end-to-end-ci-cd](https://github.com/msusdev/end-to-end-ci-cd)

## Series road map

* ~~Session 1:~~
  * ~~Introduction to Azure Pipelines~~
* ~~Session 2:~~
  * ~~Automatically build and test your code with Azure Pipelines~~
* **Session 3:**
  * **↪️ Deploy your code to Azure with Azure Pipelines**

## Today's agenda

* Test our web app as a Docker container
* Create Azure Container Registry resource
* Push to Azure Container Registry using Azure Pipelines
* Push Azure Container Registry image to:
  * Azure Container Apps
  * Azure Web Apps

::: notes

Learn how to use Azure Pipelines and workflows to implement a continuous delivery (CD) solution that deploys a web application to Microsoft Azure. We’ll also show how to automate the creation and teardown of the deployment environments using a workflow.

* Conditionally trigger continuous delivery from a workflow
* Deploy code to Microsoft Azure using a workflow
* Store secret credentials in Azure DevOps

:::

## Review: Azure Pipelines

::: notes

1. Start with an empty Azure DevOps project

1. Create a GitHub repository using the [msusdev-examples/contoso-spaces-dotnet-console-app](https://github.com/msusdev-examples/contoso-spaces-dotnet-console-app) template.

1. Create a new pipeline YAML file in your project.

    > **Note**: The filename of the pipeline is arbitrary. For the remainder of the demos, we will assume you named the pipeline **integration.yml**.

1. Publish the console application and NuGet package and validate:

    ```yml
    pool:
      vmImage: ubuntu-latest    
    stages:
    - stage: ci
      displayName: Continuous Integration
      jobs:
      - job: build
        displayName: Build Project Assets
        container: mcr.microsoft.com/dotnet/sdk:6.0
        steps:
        - script: |
            dotnet publish \
            --configuration Release \
            --output out \
            --self-contained \
            --runtime win-x64 \
            -p:PublishSingleFile=true \
          displayName: Publish .NET console project
        - publish: out/Contoso.Spaces.Console.exe
          displayName: Upload console app artifact
          artifact: console-app
        - script: |
            dotnet pack \
            --output pkg \
            -p:Version=1.0.0
          displayName: Package dotnet tool
        - publish: pkg/Contoso.Spaces.Console.1.0.0.nupkg
          displayName: Upload package artifact
          artifact: nuget-package
    ```

1. Commit your changes to the local repository and push the changes to GitHub.

1. Return to the new Azure DevOps project.

1. Create, save, and then run a new pipeline with the following settings:

    * **Source**: GitHub
    * **Repository**: *Select GitHub repository you created earlier*
    * **Template**: Existing Azure Pipelines YAML file
    * **Branch**: *main*
    * **Path**: */integration.yml*

1. Observe the job output logs.

:::

## Today's demo: Build and push a Docker container image

* Easy to do in Azure Pipelines
* Push to one or more container registries
* Can be done in one or two tasks

## Demo: Build and push container image

1. Create a GitHub repository using the [msusdev-examples/contoso-spaces-dotnet-web-app](https://github.com/msusdev-examples/contoso-spaces-dotnet-web-app) template.

1. Create a Dockerfile for the web application:

    ```dockerfile
    FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
    
    WORKDIR /app
    
    COPY . /app
    
    RUN dotnet publish --configuration Release --output out
    
    FROM nginx:alpine
    
    COPY --from=build /app/out/wwwroot /usr/share/nginx/html
    COPY nginx.conf /etc/nginx/nginx.conf
    
    EXPOSE 80
    ```

1. Test the dockerfile locally:

    ```bash
    docker build --tag dotnetwebapp .
    docker run --detach --publish 5000:80 dotnetwebapp
    ```

1. In the Azure portal, create a new **Azure Container Registry** resource.

1. Enable the **Admin user** for the ACR resource.

1. Return to the Azure DevOps project.

1. Create, save, and then run a new pipeline with the following settings:

    * **Source**: GitHub
    * **Repository**: *Select GitHub repository you created earlier*
    * **Template**: *Docker* | *Build a Docker image*
    * **Dockerfile**: ``$(Build.SourcesDirectory)/Dockerfile``

1. Rename the YAML file to **integration.yml**.

    > **Note**: The filename of the pipeline is arbitrary. For the remainder of the demos, we will assume you named the pipeline **integration.yml**.

1. In the **integration.yml** file, modify the stage/job configuration:

    ```yml
    pool:
      vmImage: ubuntu-latest    
    stages:
    - stage: cd
      displayName: Continuous Deployment
      jobs:
      - job: docker_build_push
        displayName: Build and Push to Azure Container Registry
        steps:
    ```

1. Add a Docker build step and validate:

    ```yml
    - task: Docker@2
      inputs:
        command: 'build'
        Dockerfile: '$(Build.SourcesDirectory)/Dockerfile'
        tags: |
          $(Build.BuildId)
          latest
    ```

1. Update the name of the Docker container image and validate:

    ```yml
    - task: Docker@2
      inputs:
        repository: 'dotnetwebapp'
        command: 'build'
        Dockerfile: '$(Build.SourcesDirectory)/Dockerfile'
        tags: |
          $(Build.BuildId)
          latest
    ```

1. Create a new service connection with the following settings:

    * **Registry type**: *Others*
    * **Docker Registry**: *https://\<name-of-acr-resource\>.azurecr.io/v1/*
    * **Docker ID**: *\<name-of-acr-resource\>*
    * **Docker Password**: Copy from the admin user configuration page
    * **Service conenction name**: Use a unique name

1. Update the name of the Docker container image again and validate:

    ```yml
    - task: Docker@2
      inputs:
        containerRegistry: '<name-of-acr-service-connection>'
        repository: 'dotnetwebapp'
        command: 'build'
        Dockerfile: '$(Build.SourcesDirectory)/Dockerfile'
        tags: |
          $(Build.BuildId)
          latest
    ```

1. Grant permission for the pipeline to access the service connection

1. Add a Docker push step and validate:

    ```yml
    - task: Docker@2
      displayName: Push to Azure Container Registry
      inputs:
        containerRegistry: '<name-of-acr-service-connection>'
        repository: 'dotnetwebapp'
        command: 'push'
        tags: |
          $(Build.BuildId)
          latest
    ```

1. In the Azure Portal, check the repositories for the ACR resource.

1. Login to Azure CLI, and then login to ACR on your local machine:

    ```bash
    az login
    az acr login --name <name-of-acr-resource>
    ```

1. Pull your container image from ACR locally to test:

    ```bash
    docker pull <name-of-acr-resource>.azurecr.io/dotnetwebapp:latest
    docker run --detach --publish 4000:80 <name-of-acr-resource>.azurecr.io/dotnetwebapp:latest
    ```

1. Deploy to Azure Web Apps

1. Deploy to Azure Container Apps

:::

## Reviewing today's session

* Test our web app as a Docker container
* Create Azure Container Registry resource
* Push to Azure Container Registry using Azure Pipelines
* Push Azure Container Registry image to:
  * Azure Container Apps
  * Azure Web Apps

## Reference Links

* <https://docs.microsoft.com/azure/devops/pipelines/ecosystems/containers/build-image>
* <https://docs.microsoft.com/azure/devops/pipelines/ecosystems/containers/push-image>
* <https://docs.microsoft.com/azure/app-service/tutorial-custom-container>
* <https://docs.microsoft.com/azure/container-apps/get-started-existing-container-image>

## Microsoft Learn

* <https://docs.microsoft.com/learn/paths/build-applications-with-azure-devops/>

## Thank You! Questions?
