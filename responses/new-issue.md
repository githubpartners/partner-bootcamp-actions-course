## Getting our secrets

#### Azure secrets

<details><summary>How to configure Azure secrets</summary>

1. In your terminal, run:
   ```
   az login
   ```
1. When prompted enter your Azure credentials
1. This command will return an object similar to that depicted below. Copy the value of the `id:` field to a safe place. We will later store it as a repository secret named `AZURE_SUBSCRIPTION_ID`.
   ```
   [
   {
   "cloudName": "AzureCloud",
   "id": "f****a09-****-4d1c-98**-f**********c", # <-- Copy this id field
   "isDefault": true,
   "name": "the-name-of-the-subscription-you-created-earlier",
   "state": "Enabled",
   "tenantId": "********-a**c-44**-**25-62*******61",
   "user": {
       "name": "mdavis******@*********.com",
       "type": "user"
       }
   }
   ]
   ```
1. Next we will create the service principle, which is will allow us to manage resources in your subscription. Run this command in your terminal:

```
az ad sp create-for-rbac --name "GitHub-Actions" --role contributor \
                        --scopes /subscriptions/{subscription-id} \
                        --sdk-auth

# Replace {subscription-id} with the id you copied to use as AZURE_SUBSCRIPTION_ID.
```

> **The \ character works as a line break on Unix based systems. If you are on a Windows based system the \ character will cause this command to fail. Place this command on a single line if you are using Windows.**

1. The above command will respond with an object similar to the one shown below. Copy the entire objects contents to a safe place. We will later store it as a repository secret named `AZURE_CREDENTIALS`
   ```
   {
   "clientId": "<GUID>",
   "clientSecret": "<GUID>",
   "subscriptionId": "<GUID>",
   "tenantId": "<GUID>",
   (...)
   }
   ```
1. Time to add these secrets to the repository! [Navigate to this repository's secrets]({{repoUrl}}/settings/secrets) in the Settings tab.
1. Click **New secret**
1. Name your new secret **AZURE_SUBSCRIPTION_ID** and paste the value from the `id:` field from the result of step 2.
1. Click **Add secret**.
1. Repeat this process for **AZURE_CREDENTIALS** and paste the entire contents from the terminal command you ran in step 4.
</details>

#### Probot secrets

<details><summary>How to configure Probot secrets</summary>

Like Azure, our Probot app has some sensitive data that we need to configure as a secret.

1. Time to add these secrets to the repository! [Navigate to this repository's secrets]({{repoUrl}}/settings/secrets) in the Settings tab.
1. Click **New secret**
1. Name your new secret **PROBOT_APP_ID** and paste the value from the [app settings page](https://github.com/settings/apps) of the Probot app you created.
1. Click **Add secret**.
1. Repeat this process for **PROBOT_WEBHOOK_SECRET** using the webhook secret you generated when you first created the app.
1. Lastly, create a **PROBOT_APP_PRI_KEY** secret with the contents of the private key you downloaded after creating the application.
   > **Note: The private key needs to be stored with \n characters so that Azure can understand it. Notice the \n characters in this sample key**
   ```
   -----BEGIN RSA PRIVATE KEY-----\n
   ***********CAQEA6rpU1UXM51b2Xr7EaVaS2LxPeXN9u*******************
   ***********************************************2Bt9iyYeKgHV8vgAt
   YxmGbiDJSFPrUzadtnyi1l9CsKfwAtzU7sSF0YxdvvlB3h/V2BvZpN0YT0xfaPK7
   PdRVXoHMbkxQEngxHmM8MGROkeX3UD7r/ngV/rM6a2V8jt2VwlwGRTFAlGaOrcZc
   ZiXGYhr57kzrvkwz9+Rw5P6Ws8KAvnhnpFdxc04MRIMQHvZfUw+VPK0j1+4UGCyp
   ********i1RiSuj9bSHtlOVd*************wIDAQABAoIBAQCDmG9TGkzkLcb2
   GzF2dQji5cAQsQT9sDas3rdoSi7N57RXkuXAN+n+cDKbbJCqGnXKzhFJfzjamWG2
   GwX************X+aw28ioqHgkyN6v1wZewhXDucQcnu5SlFGubEYkhckqrS5ut
   nyzpp9***************************EjO/yQ15uBLGgSEkeSa+Ca9PXzjysnA
   Aoyv/Z7ulWp45cLgKJEh3j+T6KXaDmXi1x/0lv5cbg2xycilRLdznRki0VWI7KK5
   23u+HqsXjqd3E2PRujsZCYzHtS3RRy4NSIduryt9zj+jza/s5jWGB2T76paBipXR
   RF******************S5t+/MUcAHZW8PQ7VPfym1AgFCdq0i1GNrr+JFYV7i5T
   tiAx7NChqAY8b6qCslzgT02zEcPOU00OTSiHNo8qOGs8JFCFCFZdDWnM6ZHGmGuT
   HfKz5c6Q4fQgeKD8ujhcUPGhij4BPQelyxEBCwM1yAX427HnnyQw4WHhAoGBAOza
   onqq0Al5/txqbf********************************V2iPs5ynW1/AMg0mVZ
   ********KdUYt/SBf+WMbxXcbIJvD4YzLfLcclXdl0AiSbcO4DxwiDqS3T/dNzh3
   KcecGReTjZYN5KkRLpKqSfrx79Q+************************************
   nGOhbxo4JoLxTNWiTEuyHaDwJ5u+5UiRuWAuPsz7kJTqEwwF4Z4n+Hrdo3ji83D0
   vJq3+c1/n2e*******************************************871VXxcXLS
   GrUewQ4OkwZ0Tz6zsSK92mECgYEAl/piL1vTSYFiK***********************
   JQU7lD6LorRGHBtHIgyfAJDZOBijaC0X0IdhXgIpWXcW8mj17BEEZn6dkcQ5rd7B
   KQiSxM3+80QNEM0WzTFX+F2s***************************2wzLitMoByEES
   *************8CEsn0M9q4S6rgyKsr1pkzCZDFuJuvEoUJc6oQZN/yGgcDH2hCT
   YHPsJzLHeVd98NEdywC7xmylDyh4i0OznCOWUwSmCCqcPoGk4UDuttWQtY8tK6iT
   1A8lkEE3c33QJRMYAduUPXKzWZ/RjS8BVvsqPuYunAtpRxnfUrhbVA==
   \n-----END RSA PRIVATE KEY-----\n
   ```
   </details>

#### Final outcome

Once all secrets are added your repository should look like this
![image](https://user-images.githubusercontent.com/69262924/89960438-e5f81c80-dbf3-11ea-8594-ae6b78be4b67.png) and each secret should have the proper values

---

<h5 align="center">To continue this activity leave this issue open and navigate to the pull request titled "Creating Azure Resources".</h5>
