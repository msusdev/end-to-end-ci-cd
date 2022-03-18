---
title: 'Automatically build and test your code with GitHub Actions'
author: 'Part 2 of 3 in the [Build end-to-end CI/CD capabilities with GitHub Actions](https://github.com/msusdev/end-to-end-ci-cd) series'
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
* **Session 2:**
  * **↪️ Automatically build and test your code with GitHub Actions**
* Session 3:
  * Deploy your code to Azure with GitHub Actions

## Today's agenda

* Review GitHub Actions
* Automate build for a web app

::: notes

We’ll show you how to implement continuous integration (CI) for code maintained in GitHub repositories. You’ll learn to:

* Build a project in a workflow
* Create build assets for deployment
* Distribute builds as packages

:::

## Review: GitHub Actions

::: notes

1. Create a GitHub repository with a single README file

1. Use Visual Studio Code to create a simple node project:

    ```
    node_modules/
    package-lock.json
    ```

    ```json
    {
      "name": "simple",
      "main": "app.js"
    }
    ```

    ```bash
    npm install --save moment
    ```

    ```javascript
    console.log('Hello, world!');
    
    var moment = require('moment');
    var date = moment().format('LL');
    console.log(date);
    ```

    > **Note**: You can use GitHub Codespaces to perform this task quickly.

1. Create a **.github/workflows/build.yml** file.

    > **Note**: The name of this file is arbitrary. For the remainder of this demo, we will assume you named this file **build.yml**.

1. In the **build.yml** file, add and validate:

    ```yml
    name: Node Continuous Integration
    on: push
    jobs:
      build-node:
        name: Build Node
        runs-on: ubuntu-latest
        container: node:14
        steps:
          - run: node --version
            name: Check Node Version
          - run: npm --version
            name: Check NPM Version
          - uses: actions/checkout@v2
            name: Checkout Code
          - run: npm install
            name: Install NPM Packages
          - run: node app.js
            name: Run Application
    ```

:::

## Demo: Automate Build

::: notes

1. Create a GitHub repository using the [msusdev-examples/contoso-spaces-next-web-app](https://github.com/msusdev-examples/contoso-spaces-next-web-app) template.

1. Test the web app using the following commands:

    ```bash
    npm install
    npm run dev
    ```

    > **Note**: You can use GitHub Codespaces to perform this task quickly.

1. Terminate the running ``dev`` command.

1. Observe the ``scripts.build`` property of the **package.json** file.

1. Create a **.github/workflows/integration.yml** file.

    > **Note**: The name of this file is arbitrary. For the remainder of this demo, we will assume you named this file **integration.yml**.

1. In the **integration.yml** file, create a build job and validate:

    ```yml
    name: Build Next.js web application
    on: push
    jobs:
      build-project:
        name: Build Project
        runs-on: ubuntu-latest
        steps:
        - name: Checkout code
          uses: actions/checkout@v2
        - name: Install NPM dependencies
          run: npm install
        - name: Build project assets
          run: npm run build
    ```

1. Upload an artifact and validate:

    ```yml
    ...
    - name: Upload static site
      uses: actions/upload-artifact@v2
      with:
        name: static-site
        path: .next/
    ```

1. Create a release job and validate:

    ```yml
    ...
    release-project:
      name: Release Project
      runs-on: ubuntu-latest
      needs: build-project
    ```

1. Download an artifact in the release job and validate:

    ```yml
    ...
    steps:
    - name: Download site content
      uses: actions/download-artifact@v2
      with:
        name: static-site
    ```

1. View the contents of the artifact and validate:

    ```yml
    ...
    - name: View content
      run: ls -R
    ```

1. Compress the folder and validate:

    ```yml
    - name: Archive site content
      uses: thedoctor0/zip-release@master
      with:
        filename: site.zip
    ```

1. Create a GitHub release and validate:

    ```yml
    - name: Create GitHub release
      uses: actions/create-release@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        tag_name: ${{ github.run_number }}
        release_name: Release ${{ github.run_number }}
    ```

1. Upload the final asset to release and validate:

    ```yml
    - name: Create GitHub release
      id: create-new-release
      uses: actions/create-release@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        tag_name: ${{ github.run_number }}
        release_name: Release ${{ github.run_number }}
    - name: Upload release asset
      uses: actions/upload-release-asset@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        upload_url: ${{ steps.create-new-release.outputs.upload_url }}
        asset_path: ./site.zip
        asset_name: site-v${{ github.run_number }}.zip
        asset_content_type: application/zip
    ```

:::

## Reviewing today's session

* Review GitHub Actions
* Automate build for a web app

## Reference Links

* <https://github.com/actions/upload-artifact>
* <https://github.com/actions/download-artifact>
* <https://github.com/actions/create-release>
* <https://github.com/actions/upload-release-asset>

* <https://github.com/marketplace/actions/zip-release>

## Reference Links (continued)

* <https://docs.github.com/actions/reference/workflow-syntax-for-github-actions>
* <https://docs.github.com/actions/reference/context-and-expression-syntax-for-github-actions>

* <https://docs.github.com/actions/guides/publishing-docker-images>
* <https://github.com/docker/build-push-action/>

## Microsoft Learn

* <https://docs.microsoft.com/learn/paths/automate-workflow-github-actions/>

## Thank You! Questions?
