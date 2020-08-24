
In a continuous integration and development cycle that uses GitHub Actions, workflow files define what happens when your workflow is triggered. Think of the workflow file as a blueprint that establishes structure. The template not only defines how the workflow should run, but also establishes working parameters such as: 

    - the operation system to use on runners
    - programming languages supported
    - instructions to install dependencies

In this exercise, you will become familiar with the workflow files. As you construct your own file, you will also learn about different parts of the template and their purpose. 

By the end of the exercise you will have created a usable workflow file and defined your own workflow for your Probot app.

Let's get started!

## 1. Find the CD workflow file

Locate the `probot-cd.yml` file inside of the directory `.github/workflows`. See what's already there:

**Workflow name**: The following line of code sets the name of your workflow.  You don't need to add this, it's already there.

```
name: Test and deploy probot

```

**Define events that trigger your workflow**:

There are a number of event that take place inside of or to a repository. These events can be detected and then acted on from within your workflow.  In the case of the probot app, we want to have the system trigger the workflow when users push commits to the `main` or `master`branch.

These are also already included, so you don't need to add anything to these lines:

```
on:
  push:
    branches:
      - master
      - main

```

**Environment variables**: Notice the variable for the Azure Webapp name. This correlates to the values in Azure.

```
env:
  AZURE_WEBAPP_NAME: probot-add-collaborators
```

## 2. Define jobs

Now, it's time to start making some changes. This is where we will list out the instructions that we want to happen when this action is run.

When you define a job inside of your workflow, you will specify its name and the operating system that it runs on.

For the Probot app, define a job named `package-and-deploy-to-azure` that runs on Ubuntu.

```
jobs:
  package-and-deploy-to-azure:
    runs-on: ubuntu-latest
```

## 3. Assign steps to the job

Finally, let's define what should be done sequentially as the job  `package-and-deploy-to-azure` runs.

Notice the section `steps`:

```
jobs:
  package-and-deploy-to-azure:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository contents
        uses: actions/checkout@v2

      - name: Setup NodeJS
        uses: actions/setup-node@v1
        with:
          node-version: "12"

      - name: Install dependencies
        run: |
          npm install
          npm run lint
          npm run test

      - name: Login to Azure
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Package and Deploy to Azure
        if: success()
        uses: azure/webapps-deploy@v2
        with:
          app-name: ${{ env.AZURE_WEBAPP_NAME }}

      - name: Logout of Azure
        run: |
          az logout

```

<details>
  <summary>Want to check your work? Here is the complete `probot-cd.yml` file.</summary>
  <br>
  
  ```
  on:
  push:
    branches:
      - master
      - main

env:
  AZURE_WEBAPP_NAME: probot-add-collaborators

jobs:
  package-and-deploy-to-azure:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository contents
        uses: actions/checkout@v2

      - name: Setup NodeJS
        uses: actions/setup-node@v1
        with:
          node-version: "12"

      - name: Install dependencies
        run: |
          npm install
          npm run lint
          npm run test

      - name: Login to Azure
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Package and Deploy to Azure
        if: success()
        uses: azure/webapps-deploy@v2
        with:
          app-name: ${{ env.AZURE_WEBAPP_NAME }}

      - name: Logout of Azure
        run: |
          az logout
  ```

</details>