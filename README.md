# apiProtectionLabDRAFT
# Protecting APIs in Azure  with F5#

Welcome to API protection Lab. This time you'll work with GraphQL APIs running in an ACI (Azure Container Instance)

# Pre-requisites
- An azure account
- F5 Advanced WAF BIG-IP Trial License

# GOAL: 

Protect vulnerable GraphQL APIs running in an Azure Container Instance with F5 Advanced WAF

# Activities: #

1. Create Azure infrastructure using cloudshell
2. Deploy GraphQL App - DVGA
3. Attack it
4. Deploy F5 Adv WAF from F5 github repo
5. Attack again an see how F5 protects the app

# Task 1 -  Create Azure resources - ONGOING(OCT04)

We'll be  creating Azure resources with ARM templates. 

1. Download project files from: https://github.com/hheitor/apiProtectionLab00/ by clicking *green button* code and then select *Download ZIP*
2. Open a sesssion to your Azure Portal
3. Open the *Cloud Shell* pane by selecting on the toolbar icon directly to the right of the search textboX
4. Select PowerShell (usually Bash is default)
5. Click  *Confirm* when asked to switch from Bash to Powershell 
6. Click the Upload button and upload : tmp_apiprot.json and wait for *upload complete message*
7. Type *ls* command in the *Cloud Shell* to validate template file is there.
8. See what regions are available and choose the closest toy you by:

                    Get-AzLocation | select displayname,location


9. Now modify *$location* value by the region you selected ( example: $location = centralus)

```

$location = '<Azure region>'
New-AzSubscriptionDeployment `
  -Location $location `
  -Name <YOUR INITIALS HERE>rgapiprotdeploy `
  -TemplateFile $HOME/<templatefilename>.json `
  -rgLocation $location `
  -rgName '<YOUR INITIALS HERE>apiprotlab'
```
Your modified code should look like this  

```
$location = 'centralus'
`New-AzSubscriptionDeployment `
  -Location $location `
  -Name labrgdeploy `
  -TemplateFile $HOME/tmp_apiprot.json `
  -rgLocation $location `
  -rgName 'myrg' `
```


  10. Wait few seconds and your confirmation. Should look like this:
```
   Id                      : /subscriptions/XXXXXXXXXX/providers/Microsoft.Resources/deployments/labrgdeploy
DeploymentName          : hhapiprotlabrgdeploy
Location                : centralus
ProvisioningState       : ###*Succeeded*###
Timestamp               : xx/xx/xx xx:xx:xx PM
Mode                    : Incremental
TemplateLink            :
Parameters              :
                          Name             Type                       Value
                          ===============  =========================  ==========
                          rgName           String                     myrg
                          rgLocation       String                     centralus

Outputs                 :
DeploymentDebugLogLevel :
```
11. Also you can grep  Get-AzResourceGroup  to find your newly RG created: 

                   Get-AzResourceGroup | grep <RGname>

12. If your RG is listed you can go to the next step :)


  Extra: you may want to navigate thru Azure portal to find your Resource Group



  # Task 2 -  Deploy DVGA - GraphQL App in an Azure Container Instance

  1. Let's create a contianer that runs our vulnerable GraphQL App. Got to your Azure Cloud Shell and run the following command:

```
  az container create -g <YOU RESOURCE GROUP> --name 
  <AZURE CONTAINER NAME DEFINED BY YOU> --image dolevf/dvga --ports 5000 --dns-name-label <A NAME TO ATTACH TO PREDFINED AZURE PUBLIC DNS NAME>  --environment-variables WEB_HOST='0.0.0.0'
```


  # Task 3 -  Do some attacks
    try...
     what else is needed?
  # Task 4 - Deploy F5 Adv WAF to protect GraphQL endpoint
  # Task 5 - Try attacks again and see f5 event log
