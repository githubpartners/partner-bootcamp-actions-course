
In a continuous integration and development cycle that uses GitHub Actions, workflow files define what happens when your workflow is triggered. Think of the workflow file as a blueprint that establishes structure. The template not only defines how the workflow should run, but also establishes working parameters such as: 

    - the operation system to use on runners
    - programming languages supported
    - instructions to install dependencies

In this exercise, you will become familiar with the workflow files. As you construct your own file, you will also learn about different parts of the template and their purpose. 

By the end of the exercise you will have created a usable workflow file and defined your own workflow for your Probot app.

Let's get started!


## 1. Create a workflow file

  Create an empty file called `cd-workflow.yml` inside of the folder `.github/workflows`.

## 2. Name your workflow

The following line of code sets the name of your workflow.  This is the first line you will place in your file.

```
name: Test and deploy probot

```

## 3. Define events that trigger your workflow

There are a number of event that take place inside of or to a repository. These events can be detected and then acted on from within your workflow.  In the case of the probot app, we want to have the system trigger the workflow when users push commits to the `main` or `master`branch.

Here are the next lines that you'll use to set this up:

```
on:
  push:
    branches:
      - master
      - main

```

## 4. Specify any environment variables

Though this step may not always be needed, the parameter `env` defines any environment variables needed in the workflow.  

Use and environment variable to set the name of your Azure webapp name:

```
AZURE_WEBAPP_NAME: probot-add-collaborators
```

## 5. Define jobs

When you define a job inside of your workflow, you will specify its name and the operating system that it runs on.

For the Probot app, define a job named `package-and-deploy-to-azure` that runs on Ubuntu.

```
jobs:
  package-and-deploy-to-azure:
    runs-on: ubuntu-latest
```

## 6. Assign steps to the job

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