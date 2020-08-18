## Getting the Probot application to Azure

Microsoft Azure is packed full of cloud computing features. These features are an advantage and a fault when it comes to getting an environment fully configured and getting the codebase deployed to the proper resources.

Luckily, using GitHub Actions, we can greatly reduce this pain point and implement a friction free process that does all of these cumbersome tasks for us.

This pull request contains a workflow that will configure all the necessary resources and deploy the current version of your Probot application automatically for us! In order to get a good understanding of what this workflow is going, it is beneficial to examine this process from a manual perspective first.

**Expand one of the following sections to lean how to configure Azure either manually or using GitHub Actions**

<details><summary>Manually configuring Microsoft Azure</summary>

**Provisioning**

1. Configure a **subscription** (this is true even when using actions!)
2. From within your subscription, create a new resource group
   ![image](https://user-images.githubusercontent.com/38021615/89914895-8a557100-dbaa-11ea-8365-e1e69dccaf70.png)
3. Give your resource group a name **probot-applications** as well as set a the region to **Central US**.
   ![image](https://user-images.githubusercontent.com/38021615/89914986-a527e580-dbaa-11ea-8db4-b62273570cff.png)
4. Click **next** a few times until you eventually click **create**
5. You should now see your **probot-applicaitons resource group** located within your **subscription**
   ![image](https://user-images.githubusercontent.com/38021615/89915129-d4d6ed80-dbaa-11ea-8113-d99545ab3048.png)
6. Next, from within your **probot-applications resource group** create a new resource
   ![image](https://user-images.githubusercontent.com/38021615/89915621-6f373100-dbab-11ea-8b58-5658d6023fb8.png)
7. Select **App Service Plan** as the resource type from the wizard
   ![image](https://user-images.githubusercontent.com/38021615/89915811-ae658200-dbab-11ea-804b-4e432705136d.png)
8. Give the plan a name of **prod-probot-apps** and ensure the SKU and size are set to **Free F1** (this will ensure you we use free tier resources and you don't get charged)
   ![image](https://user-images.githubusercontent.com/38021615/89916192-3186d800-dbac-11ea-8598-347476bdc925.png)
9. Continue clicking **next** and eventually **create**
10. Back inside your **probot-applications resource group** add a new resource and select the **Web App** resource type from the wizard
   ![image](https://user-images.githubusercontent.com/38021615/89917327-870fb480-dbad-11ea-92ce-578f4e7e00bf.png)
11. Like before, name it **probot-add-collaborators**, change the **Runtime Stack** to `Node 12 LTS`, select **Linux** as the Operating System. Lastly verify the region and the SKU
   ![image](https://user-images.githubusercontent.com/38021615/89917584-cf2ed700-dbad-11ea-9cc7-7578342b58f1.png)
12. Click **next** a few times and eventually the **create** button
13. Now you can view the **probot-add-collaborators** deployment by clicking on the public link, doing show will result in a blank web page since we have not deployed any code to this resource
   ![image](https://user-images.githubusercontent.com/38021615/89918100-77dd3680-dbae-11ea-9953-68e501e4d3fb.png)


**Deployment**

1. From the terminal run the following command:
   ```
   az webapp deployment user set --user-name <username> --password <password>
   ```
1. When that command completes run the next one:
   ```
   az webapp deployment source config-local-git --name probot-add-collaborator --resource-group probot-applications
   ```
1. The previous command will present you with output similar to that below:
   ```
   {
       "url": "https://username@msdocs-node-cli.scm.azurewebsites.net/msdocs-node-cli.git"
   }
   ```
1. Add a new remote to Git named `azure`
   ```
   git remote add azure https://msdocs-node-cli.scm.azurewebsites.net/msdocs-node-cli.git
   ```
   > **Note: leave off your username from the url when adding the remote.**
1. Push to the codebase to Azure
   ```
   git push azure master
   ```
1. Enter any credentials you may need to and wait for the deployment to finish
</details>

<details><summary>Using GitHub Actions to configure Azure</summary>
In this pull request you will find a workflow named `config-azure.yml`. This workflow follows all of the above steps, however it leverages a few official Azure Actions as well as some raw `az cli` commands to get the job done.

Let's take a quick peek at the jobs in this workflow before it get's triggered and set's up your environment for you.

1. The first two jobs of this workflow are quite simple, they checkout the code from the repository into the Actions workspace and then log in to the Azure CLI using the service principle you saved as the **AZURE_CREDENTIALS** repository secret

   ```
     - name: Checkout repository
       uses: actions/checkout@v2

     - name: Azure login
       uses: azure/login@v1
       with:
         creds: ${{ secrets.AZURE_CREDENTIALS }}
   ```

1. The next series of actions run Azure CLI commands directly from the runner. Each GitHub Actions runner comes [packed with useful tools](https://docs.github.com/en/actions/reference/software-installed-on-github-hosted-runners), Azure CLI being one of them. These steps are the CLI equivalent to the manual clicking you would have done in the Azure portal

   ```
     - name: Create Azure resource group
       if: success()
       run: |
         az group create --location ${{env.AZURE_LOCATION}} --name ${{env.AZURE_RESOURCE_GROUP}} --subscription ${{secrets.AZURE_SUBSCRIPTION_ID}}

     - name: Create Azure app service plan
       if: success()
       run: |
         az appservice plan create --resource-group ${{env.AZURE_RESOURCE_GROUP}} --name ${{env.AZURE_APP_PLAN}} --is-linux --sku F1 --subscription ${{secrets.AZURE_SUBSCRIPTION_ID}}

     - name: Create webapp resource
       if: success()
       run: |
         az webapp create --resource-group ${{ env.AZURE_RESOURCE_GROUP }} --plan ${{ env.AZURE_APP_PLAN }} --name ${{ env.AZURE_WEBAPP_NAME }}  --runtime "NODE|12-lts" --subscription ${{secrets.AZURE_SUBSCRIPTION_ID}}

     - name: Configure probot app secrets
       if: success()
       run: |
         az webapp config appsettings set  --name ${{ env.AZURE_WEBAPP_NAME }} --resource-group ${{ env.AZURE_RESOURCE_GROUP }} --subscription ${{secrets.AZURE_SUBSCRIPTION_ID}} --settings APP_ID=${{secrets.PROBOT_APP_ID}} 
   ```

1. In the example in step 2 you should notice a range of environment variables and repository secrets being used to fill in the sensitive information for each command. You may have also noticed the conditional logic `if: success()` that will cause these steps to execute sequentially to ensure resources are created in the proper order.
   > **Note: Azure CLI is not the most secure method for this task. It returns objects that print sensitive information. As such, the object is being used in this capacity for demonstration purposes only. Take care when using Azure CLI.**
2. The final few steps handle getting the our codebase deployed to the newly created Azure resources! We first setup our NodeJS environment, then install our dependencies and finally deploy our packed to azure

   ```
     - name: use Node
       uses: actions/setup-node@v1
       with:
         node-version: '12'

     - name: install deps
       run: |
         npm install

     - name: Deploy application
       if: success()
       uses: azure/webapps-deploy@v2
       with:
         app-name: ${{ env.AZURE_WEBAPP_NAME }}
   ```

</details>

---

### Activity

Now, it's your turn to configure your Azure environment. You can proceed with either the manual configuration or use GitHub Actions.  

But first, you will need to add your private key manually to the Azure environment.

#### 1. Add your private key to you Azure application

The Probot app will need to make use of a private key. In a real world setting, we could automate this step by adding it to our workflow.  However, due to a known bug in Azure that occurs when passing the the private key data, we'll guide you through the manual entry of the private key in Azure. 

1. Copy your private key. 

  > **Note:** When adding the PRIVATE_KEY you need to add in the line break characters (\n) so that it looks like this:
  
   ```
      -----BEGIN PRIVATE KEY-----\n
      LONG_STRING_HERE\n
      -----END PRIVATE KEY-----
   ```

2. Add your private key to your **probot-add-collaborator app service** 
   - Go to **Settings > Configuration**
   - Add a new application setting
   - Paste in the private key
  
   ![image](https://user-images.githubusercontent.com/38021615/89952339-eab3d500-dbe1-11ea-9861-b18a053b3dc0.png)

3. Save the key
4. If prompted to Resave, click Continue
5. You are ready to deploy the codebase in this repo to your Azure resources


#### 2. Configure Azure

##### Manual configuration

- Follow the manual process shown above to for configuring Microsoft Azure and simply **close** this pull request when you are finished.

##### Configuration using GitHub Actions 
- Use this Actions workflow to configure Azure. To use this workflow:
  1. **Merge** this pull request 
  2. Go to the issue **Setting up your environment!** issue for further instructions.

In either case, once the deployment of code is finished you need to replace the [Probot Webhook URL](https://github.com/settings/apps) in the app settings to point to your azure deployment

![image](https://user-images.githubusercontent.com/38021615/89918100-77dd3680-dbae-11ea-9953-68e501e4d3fb.png)

![image](https://user-images.githubusercontent.com/69262924/89957400-26539c80-dbec-11ea-8c7c-74f9595f8531.png)

