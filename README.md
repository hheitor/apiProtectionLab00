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

  https://www.google.com/search?q=f5+logo+png&rlz=1C5CHFA_enUS874US874&sxsrf=AOaemvJFvCnKxhj5yIt2Zsf2pxB99bhSsg:1633368169514&tbm=isch&source=iu&ictx=1&fir=2Gad-4Ci8Oc02M%252CN4Cb2UdhPz7VBM%252C_%253Bkpvpz0qeus-KBM%252CedKUSvQFdv1uYM%252C_%253Boj0nm6yeum-SVM%252CZ8RxtXmmo_LNJM%252C_%253BiV4fPhF6JP2i5M%252C5V5jrMxZN2B8yM%252C_%253BiMKniXqG-OesKM%252CN4Cb2UdhPz7VBM%252C_%253BlXkWFF_Oc8euZM%252CjlNmv8E1hb-tZM%252C_%253BGmyLnngcuctRJM%252Cc5ZAy8J55EkUbM%252C_%253B2xYzAcX4BswqIM%252CHdktLU61JjxZfM%252C_%253Br0CTDt64AM7jOM%252CZ8RxtXmmo_LNJM%252C_%253B3zKKurXHWeS4qM%252C2nxL0pyFOaQ1jM%252C_%253BZHFPzuzqsPB2pM%252C23f5OA_aKlIGxM%252C_%253BRd1PiKJVREPtQM%252CFrLZ8ONZUyF-OM%252C_%253B9NlGdoKxwQXwGM%252C1Iojnzyz3W4XyM%252C_&vet=1&usg=AI4_-kS9Hwu95zDKzaczWOtCfP6cbmrVwQ&sa=X&ved=2ahUKEwj1x5_MorHzAhW-kWoFHTxdCfgQ9QF6BAgOEAE#imgrc=2Gad-4Ci8Oc02M

  
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
