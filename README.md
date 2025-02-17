# Azure App Service with Terraform, deployment slots and app settings

This repository contains a demo on how to use Azure Services deployment slots with Terraform 

The demo consists in :
- a few Terraform files to provision an App Service and a deployment slot
- a simple ASP.NET web app
- GitHub Action workflows to:
  - provision the Azure resources
  - deploy the web app: the _blue_ version on the _preproduction_ slot, and the _green_ version on the _staging_ slot
  - swap the _staging_ slot with the _preproduction_ one (as many time of you want)
  - destroy the Azure resources once you have finished to save costs

## Getting started

To run this demo there is nothing to install on you machine as everything is running in the cloud. You need to configure a few things in GitHub and in Terraform Cloud.  

### Create a Terraform Cloud workspace
Using Terraform Cloud using the _Local_ execution mode, so that only the state is stored in Terraform Cloud, and the commands are ran in GitHub Actions runners.  
In your Terraform Cloud organization (you can create one for free), create a new workspace using the _Local_ execution mode, and save your organization and workspace name for later.  

### Create a service principal in Azure
To grant access to your Azure subscription to the GitHub Action runners, you need to create a service principal with the _contributor_ role to your subscription.  
Everything is well explained [here](https://learn.microsoft.com/en-us/azure/developer/github/connect-from-azure?tabs=azure-cli%2Clinux#use-the-azure-login-action-with-openid-connect), follow the instructions until you have added federated credentials and save your tenantId, subscriptionId and clientId for later.

### Set a bunch of secrets

Set a few secrets. You should already have the `AZURE_CREDENTIALS` secret set from the previous step.  
Add these secrets to make Terraform CLI connect to Azure:
- `AZURE_CLIENT_ID` with your service principal's client id
- `AZURE_SUBCRIPTION_ID` with your subscription id
- `AZURE_TENANT_ID` with your tenant id

Then to grant to the runner access to your Terraform Cloud workspace you need these secrets:
- `TF_API_TOKEN` with an API token you can create from the GUI
- `TF_CLOUD_ORGANIZATION` with your Terraform Cloud organization name
- `TF_WORKSPACE` with the name of your workspace

Lastly, some of the workflows are creating secrets in GitHub, you'll need to create a [Personal Access Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) for that with the following permissions:
- Access to your repo
- Read and write access to secrets (this will also select the read access to metadata)

Once the PAT generated, save its value in the `GH_PAT` secret, and you're all set !

## Run the demo

Once everything has been set-up you can run the `01 - Initial deployment` workflow from the Actions tab in GitHub. This will create the resources in Azure, deploy a _blue_ version of the code in the preproduction slot, and a _green_ version of the code in staging.  

Go to the Azure portal to see the resources in the `rg-aps-slots-demo` resource group. You can browse both versions from there.

To make a swap, run `02 - Swap App Service slots using Azure CLI` workflow.  
Once the swap has been done, refresh the app in your browser, you should see the _green_ version in preproduction and the _blue_ one in staging.

Eventually you can perform another swap if you want to simulate a rollback.  

Lastly, do not forget to run the `03 - Destroy Azure resources with Terraform` once you have finished to save costs as the App Service runs in a Standard plan, which is not free of charge.