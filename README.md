# apiProtectionLabDRAFT
# Protecting APIs in Azure  with F5#

Welcome to API protection Lab. This time you'll work with GraphQL APIs running in an ACI (Azure Container Instance)

# Pre-requisites
- An azure account
- F5 Advanced WAF BIG-IP Trial License

# We GOAL: #

Protect vulnerable APIs running in Azure Container Service

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
3. Open the *Cloud Shell* pane by selecting on the toolbar icon directly to the right of the search textbox

![Alt text](data:image/png;base64,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 "title")


4. Select PowerShell (usually Bash is default)
5. Click  *Confirm* when asked to switch from Bash to Powershell 
6. Click the Upload button and upload : tmp_apiprot.json and wait for *upload complete message*
7. Type *ls* in the *Cloud Shell* to validate template file is there.

See what regions are available and choose the closest toy you by:

Get-AzLocation | select displayname,location


8. Now modify *$location* value by the region you selected ( example: $location = centralus)

 ## create an Azure Resource Group

$location = '<Azure region>'
New-AzSubscriptionDeployment `
  -Location $location `
  -Name <YOUR INITIALS HERE>apiprotlabrgdeploy `
  -TemplateFile $HOME/apiprotlab.json `
  -rgLocation $location `
  -rgName '<YOUR INITIALS HERE>apiprotlab'

 ## Your modified code should look like this  ##

$location = 'centralus'
`New-AzSubscriptionDeployment `
  -Location $location `
  -Name hhapiprotlabrgdeploy `
  -TemplateFile $HOME/tmp_apiprot.json `
  -rgLocation $location `
  -rgName 'hhapiprotlabDELETERG' `

  9. Wait few seconds and your confirmation. Should look like this:

   Id                      : /subscriptions/XXXXXXXXXX/providers/Microsoft.Resources/deployments/hhapiprotlabrgdeploy
DeploymentName          : hhapiprotlabrgdeploy
Location                : centralus
ProvisioningState       : *Succeeded*
Timestamp               : xx/xx/xx xx:xx:xx PM
Mode                    : Incremental
TemplateLink            :
Parameters              :
                          Name             Type                       Value
                          ===============  =========================  ==========
                          rgName           String                     hhapiprotlabDELETERG
                          rgLocation       String                     centralus

Outputs                 :
DeploymentDebugLogLevel :

Also you can grep  Get-AzResourceGroup  to find your newly RG created:  Get-AzResourceGroup | grep <RGname>

10. If your RG is listed you can go to the next step :)
  Extra: you may want to navigate thru Azure portal to find your Resource Group



  # Task 2 -  Deploy DVGA - GraphQL App in an Azure Container Instance
  # Task 3 -  Do some attacks
    try...
     what else is needed?
  # Task 4 - Deploy F5 Adv WAF to protect GraphQL endpoint
  # Task 5 - Try attacks again and see f5 event log
