---
title: 'Automatically build and test your code with Azure Pipelines'
author: 'Part 2 of 3 in the [Build end-to-end CI/CD capabilities with Azure Pipelines](https://github.com/msusdev/end-to-end-ci-cd) series'
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
* **Session 2:**
  * **↪️ Automatically build and test your code with Azure Pipelines**
* Session 3:
  * Deploy your code to Azure with Azure Pipelines

## Today's agenda

* Review Azure Pipelines
* Automate build for a console app
* Automate release to NuGet

::: notes

We’ll show you how to implement continuous integration (CI) for code maintained in GitHub repositories. You’ll learn to:

* Build a project in a workflow
* Create build assets for deployment
* Distribute builds as packages

:::

## Review Azure Pipelines

::: notes

1. Start with an empty Azure DevOps project

1. Create a GitHub repository with a single README file

1. Use Visual Studio Code to create a simple .NET project:

    ```bash
    dotnet new console
    dotnet add package Colorful.Console
    dotnet new gitignore
    ```

    ```csharp
    using static System.Drawing.Color;
    using Console = Colorful.Console;

    Console.WriteAscii("Hello, Audience!", Red);
    ```

    > **Note**: You can use GitHub Codespaces to perform this task quickly.

1. Create a new pipeline YAML file in your project.

    > **Note**: The filename of the pipeline is arbitrary. For the remainder of the demos, we will assume you named the pipeline **build.yml**.

1. Add the following content to the pipeline YAML file:

    ```yml
    pool:
      vmImage: ubuntu-20.04    
    stages:
    - stage: continuous_integration
      displayName: Continuous Integration
      jobs:
      - job: build
        displayName: Build
        container: mcr.microsoft.com/dotnet/sdk:6.0
        steps:
        - script: dotnet --version
          displayName: Check .NET SDK version
        - script: dotnet restore
          displayName: Restore NuGet packages
        - script: dotnet build
          displayName: Build .NET project
        - script: dotnet run
          displayName: Run .NET application
    ```

1. Commit your changes to the local repository and push the changes to GitHub.

1. Return to the new Azure DevOps project.

1. Create, save, and then run a new pipeline with the following settings:

    * **Source**: GitHub
    * **Repository**: *Select GitHub repository you created earlier*
    * **Template**: Existing Azure Pipelines YAML file
    * **Branch**: *main*
    * **Path**: */build.yml*

1. Observe the job output logs.

:::

## Today's demo: .NET single-file applications

* Attractive option for deploying console apps
* Easy to distribute on GitHub "releases"
* Can included debug files
* Can be compressed to reduce size
* Can have ahead-of-time (AOT) compilation enabled

## Configuring single-file publishing

### Command: ``dotnet publish``

* Release vs Debug configuration
* Self-contained vs framework-dependent deployment
* OS and architecture for executable

### Project file

* Single/multiple files (``PublishSingleFile``)
* AOT compilation (``PublishReadyToRun``)
* Compression (``EnableCompressionInSingleFile``)

::: notes

* Single-file apps must be OS and architecture-specific and target a specific runtime
* Compression comes with a performance cost
* AOT compilation can have significant performance benefits with a 2-3x size increase to the executable

:::

## Single-file command example

```bash
dotnet publish \
    --configuration Release --output pkg \
    --self-contained --runtime win-x64 \
    -p:EnableCompressionInSingleFile=true \
    -p:PublishSingleFile=true \
    -p:PublishReadyToRun=true \
    -p:DebugType=embedded
```

## Demo: Automate Build

::: notes

1. Create a GitHub repository using the [msusdev-examples/contoso-spaces-dotnet-console-app](https://github.com/msusdev-examples/contoso-spaces-dotnet-console-app) template.

1. Test the console app using the following command:

    ```bash
    dotnet run --
    ```

    > **Note**: You can use GitHub Codespaces to perform this task quickly.

1. Test the console app again using this variation on the previously ran command:

    ```bash
    dotnet run -- --seats 4
    ```

1. Test publishing the console app using a simplified version of the ``dotnet publish`` command:

    ```bash
    dotnet publish --configuration Release --output out -p:PublishSingleFile=true --self-contained --runtime win-x64
    ```

1. Observe the contents of the newly created **out** folder in your project.

1. Create a new pipeline YAML file in your project.

    > **Note**: The filename of the pipeline is arbitrary. For the remainder of the demos, we will assume you named the pipeline **integration.yml**.

1. Publish the console application as an EXE and validate:

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
            -p:EnableCompressionInSingleFile=true \
            -p:PublishSingleFile=true \
            -p:PublishReadyToRun=true \
            -p:DebugType=embedded
          displayName: Publish .NET console project
    ```

1. Create a build artifact for the console application and validate:

    ```yml
    ...
    - publish: out/Contoso.Spaces.Console.exe
      displayName: Upload console app artifact
      artifact: console-app
    ```

:::

## Pack command example

```bash
dotnet pack \
    --output pkg \
    -p:Version=1.0.0
```

## Another demo: NuGet publishing

* .NET console application can be distributed at **dotnet tools**
* Project is only required to be rebuilt using ``dotnet pack``
* Once packaged, the NuGet package (**.nupkg**) is published to <nuget.org>
* Versions are controlled using the **Version** project property

## Demo: Automate Package &amp; Release

::: notes

1. Test publishing the console app using the ``dotnet pack`` command:

    ```bash
    dotnet pack --output pkg -p:Version=1.0.0
    ```

1. Observe the contents of the newly created **pkg** folder in your project.

1. Package the dotnet tool for NuGet and validate:

    ```yml
    ...
    - script: |
        dotnet pack \
        --output pkg \
        -p:Version=1.0.0
      displayName: Package dotnet tool
    ```

1. Create a build artifact for the NuGet package and validate:

    ```yml
    ...
    - publish: pkg/Contoso.Spaces.Console.1.0.0.nupkg
      displayName: Upload package artifact
      artifact: nuget-package
    ```

1. Create a release job and validate:

    ```yml
    ...
    - job: release
      displayName: Release on NuGet
      dependsOn: build
      container: mcr.microsoft.com/dotnet/sdk:6.0
    ```

1. Download the NuGet package artifact and validate:

    ```yml
    ...
    steps:
    - download: current
      displayName: Download package artifact
      artifact: nuget-package
    ```

1. Navigate to the NuGet website and get a new API key for your account.

1. In your Azure Pipeline, create a new variable with the following settings:

    * **Name**: ``nuget-api-key``
    * **Is Secret**: yes
    * **Value**: API key from NuGet website

1. Push to NuGet and validate:

    ```yml
    ...
    - script: |
        dotnet nuget push \
        Contoso.Spaces.Console.1.0.0.nupkg \
        --skip-duplicate \
        --api-key $(nuget-api-key) \
        --source https://api.nuget.org/v3/index.json
      displayName: Push to NuGet.org
      workingDirectory: $(Pipeline.Workspace)/nuget-package
    ```

1. Navigate to <nuget.org> and view the newly created NuGet package.

1. Test the new NuGet dotnet tool's installation:

    ```bash
    dotnet tool install --global Contoso.Spaces.Console --version 1.0.0
    contosospaces
    contosospaces --seats 4
    dotnet tool uninstall --global Contoso.Spaces.Console
    ```

:::

## Reviewing today's session

* Review Azure Pipelines
* Automate build for a console app
* Automate release to NuGet

## Reference Links

* <https://docs.microsoft.com/azure/devops/pipelines/artifacts/pipeline-artifacts>
* <https://docs.microsoft.com/azure/devops/pipelines/tasks/utility/command-line>
* <https://docs.microsoft.com/dotnet/core/deploying/single-file/overview>
* <https://docs.microsoft.com/dotnet/core/tools/dotnet-publish>
* <https://docs.microsoft.com/dotnet/core/tools/dotnet-pack>
* <https://docs.microsoft.com/dotnet/core/tools/dotnet-nuget-push>

## Microsoft Learn

* <https://docs.microsoft.com/learn/paths/build-applications-with-azure-devops/>

## Thank You! Questions?
