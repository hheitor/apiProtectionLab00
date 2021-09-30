# apiProtectionLabDRAFT
# Protecting APIs in Azure  with F5#

Welcome to API protection Lab. This time you'll work with GraphQL APIs running in azure ACS

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

# Task 1 -  Create Azure resources

1. Download JSON files from: https://github.com/hheitor/apiprot/azuretemplates
2.  Open a sesssion to your Azure Portal
3. Open the *Cloud Shell* pane by selecting on the toolbar icon directly to the right of the search textbox
4. Select PowerShell
5. In the Upload section click the button and upload 

$location = '<Azure region>'
New-AzSubscriptionDeployment `
  -Location $location `
  -Name <YOUR INITIALS HERE>apiprotlabrgdeploy `
  -TemplateFile $HOME/apiprotlab.json `
  -rgLocation $location `
  -rgName '<YOUR INITIALS HERE>apiprotlab'
  
  # Task 2 -  Deploy DVGA - GraphQL App
  # Task 3 -  Do some attacks
    try...
     what else is needed?
  # Task 4 - Deploy F5 Adv WAF to protect GraphQL endpoint
  # Task 5 - Try attacks again and see f5 event log
