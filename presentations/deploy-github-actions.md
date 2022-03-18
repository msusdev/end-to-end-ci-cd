---
title: 'Deploy your code to Azure with GitHub Actions'
author: 'Part 3 of 3 in the [Build end-to-end CI/CD capabilities with GitHub Actions](https://github.com/msusdev/end-to-end-ci-cd) series'
---

## About the presenter

Presenter Name

☁️ *Presenter Title*

> For questions or help with this series: <msusdev@microsoft.com>

All demos and source code available on **GitHub**:

> [github.com/msusdev/end-to-end-ci-cd](https://github.com/msusdev/end-to-end-ci-cd)

## Series road map

* ~~Session 1:~~
  * ~~Introduction to GitHub Actions~~
* ~~Session 2:~~
  * ~~Automatically build and test your code with GitHub Actions~~
* **Session 3:**
  * **↪️ Deploy your code to Azure with GitHub Actions**

## Today's agenda

* Test our web app as a Docker container
* Push to Docker Hub using GitHub Actions
* Push Docker Hub image to:
  * Azure Container Apps
  * Azure Web Apps

::: notes

Learn how to use GitHub Actions and workflows to implement a continuous delivery (CD) solution that deploys a web application to Microsoft Azure. We’ll also show how to automate the creation and teardown of the deployment environments using a workflow.

* Conditionally trigger continuous delivery from a workflow
* Deploy code to Microsoft Azure using a workflow
* Store secret credentials in GitHub

:::

## Demo: Dockerize app

::: notes

1. Return to the GitHub project.

1. Create a Dockerfile for the web application:

    ```dockerfile
    FROM node:alpine

    WORKDIR /app
    
    COPY . /app
    
    RUN npm install
    RUN npm run build
    
    EXPOSE 80
    
    CMD npm run start
    ```

1. Test the dockerfile locally:

    ```bash
    docker build --tag nextwebapp .
    docker run --detach --publish 6000:80 nextwebapp
    ```

1. In the **integration.yml** file, add a container job and validate:

    ```yml
    build-container:
      name: Build container
      runs-on: ubuntu-latest
      steps:
        - name: Checkout code
          uses: actions/checkout@v2
    ```

1. Add the following secrets to GitHub Actions:

    * **DOCKER_HUB_USERNAME**
    * **DOCKER_HUB_ACCESS_TOKEN**

1. Login to Docker Hub and validate:

    ```yml
    - name: Login to Docker Hub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKER_HUB_USERNAME }}
        password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
    ```

1. Push your container image to Docker Hub and validate:

    ```yml
    - name: Build and push to Docker Hub
      uses: docker/build-push-action@v2
      with:
        context: .
        push: true
        tags: <organization>/<image>:latest, <organization>/<image>:${{ github.run_number }}
    ```

1. Pull your container image from Docker Hub locally to test:

    ```bash
    docker pull <organization>/<repository>
    docker run --detach --publish 5000:80 <organization>/<repository>:latest
    ```

1. In the **integration.yml** file, push your container image to GitHub Packages and validate:

    ```yml
    - name: Login to GitHub Container Registry
      uses: docker/login-action@v1
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    - name: Build and push to GitHub Container Registry
      uses: docker/build-push-action@v2
      with:
        context: .
        push: true
        tags: <organization>/<repository>/<image>:latest, <organization>/<repository>/<image>:${{ github.run_number }}
    ```

1. Pull your container image from GitHub Packages locally to test:

    ```bash
    docker login https://docker.pkg.github.com
    docker pull docker.pkg.github.com/<organization>/<repository>
    docker run --detach --publish 4000:80 docker.pkg.github.com/<organization>/<repository>
    ```

1. Deploy to Azure Web Apps

1. Deploy to Azure Container Apps

:::

## Reviewing today's session

* Test our web app as a Docker container
* Push to Docker Hub using GitHub Actions
* Push Docker Hub image to:
  * Azure Container Apps
  * Azure Web Apps

## Reference Links

* <>

## Microsoft Learn

* <https://docs.microsoft.com/learn/paths/automate-workflow-github-actions/>

## Thank You! Questions?
