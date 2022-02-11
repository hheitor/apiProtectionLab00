# apiProtectionLabBeta
# Protecting APIs in Azure  with F5#

Welcome to API protection Lab. This time you'll work with GraphQL endpoint running in Linux VM instance on Azure

# Pre-requisites
- An azure account

# GOAL: 

Protect vulnerable GraphQL endpoint running in an Azure VM with F5 Advanced WAF

# Activities: #

1. Create Azure infrastructure using Azure Cloudshell
2. Create GraphQL application on a linux VM
3. Do some attacks
4. Deploy F5 Adv WAF from Azure Cloudshell & configure it using GUI
5. Attack again an see how F5 protects the app
6. Delete your created resources

# Task 1 -  Create Azure resources using Azure CLI

This where your role as Cloud admin begins, so let's go!

1. Go to you Azure Portal and click on the shell icon
2. Make sure Bash is selected (default)
3. Validate what locations are available to your subscription by issuing:
```

az account list-locations --output  table


```
See Name column and select the location  that makes more sense to  you. In this example we'll take 'centralus'

4. Define shell variables for further use with your location and Reource Group Name. In this example we use: centralus & hhRG:

```

location=centralus
rg=<YOUR INITIALS>RGlab
vm=graphqlVM


```

** Be careful with spacing

*** vm var is for later use

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
Take note of the Public IP Address & Private IP Address for later USE

 
2. Connect to your VM using your ssh key file located at $HOME/.ssh

```

ssh -i id_rsa azureuser@<YOUR VM PUBLIC IP>



```
3. Once inside proceed to install graphQL app:

```
 sudo apt-get update 
 sudo  apt-get install -y docker.io 
 sudo  docker pull dolevf/dvga 
 sudo docker run -t -p 5000:5000 -e WEB_HOST=0.0.0.0 dolevf/dvga -d
 sudo docker ps

```
 
4. Docker ps command should show container runningn on port 5000:

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
      "etag": "W/\"<ETAG>"",
      "id": "/subscriptions/<NONE OF YOUR BUSINESS>/resourceGroups/hhRGlab/providers/Microsoft.Network/networkSecurityGroups/graphqlVMNSG/defaultSecurityRules/AllowVnetInBound",
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
     
   
      "id": "/subscriptions/<NONE OF YOUR BUSINESS>/resourceGroups/hhRGlab/providers/Microsoft.Network/networkSecurityGroups/graphqlVMNSG/defaultSecurityRules/DenyAllOutBound",
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
- Navigate Private Pastes, Import paste and other left-located tabs



7. Now you can start attacking this app :)


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

As you can see, your GraphQL endpoint works and also allows anyone to know internal networking. Nice point to start target recoinossance. Not cool.

#### Deep Recursion - DoS
2.  Now try to abuse GraphQL parser by making it busy. 


```

curl -g \
-X POST \
-H "Content-Type: application/json" \
-d '{"query":"query {pastes {owner {paste {edges {node {owner {paste {edges {node {owner {paste {edges {node {owner {id}}}}}}}}}}}}}}}"}' \
http://<VM Public IP>:5000/graphql

```
Note the time it takes to answer this single request. This may lead to DoS increasing the number of requests.



There are lots of attacks you can do on graphQL endpoints that can compromise underlying infrastructure so yes, you need to protect them as well with WAF that understands this arcitectural style.



  # Task 4 - Deploy F5 Adv WAF from Azure Cloudshell & configure it using GUI

1. Create an PAYG instance of 25Mb Advanced WAF by fisrt getting its URN:

 ```
az vm image list --publisher f5-networks --all 

```
  I know, we don't use the 'networks' thing anymore but we cool :)
  
  As you can see a lot of output is shown but paying a bit of attention you'll see URN also correspond to BIG-IP version so, look for the URN that contains awf + 25mbps+hourly + 16.1

  Version 16.x includes GraphQL filters to better protect this traffic.

2. Filter ouput using this command:

```
az vm image list --publisher f5-networks --all | grep 25m | grep 16\.1\.1 | grep awf

```
 your URN of interese shall be:

 ```
 "urn": "f5-networks:f5-big-ip-advanced-waf:f5-big-awf-plus-hourly-25mbps:16.1.100000"

```

3. Create your f5 WAF:

  ```

  az vm create -n f51waf01 -g $rg --image f5-networks:f5-big-ip-advanced-waf:f5-big-awf-plus-hourly-25mbps:16.1.100000 --admin-username azureuser --admin-password  <YOU F5 PASSWORD>  
  

  ```

Output shall be similar to this:

```
{
  "fqdns": "",
  "id": "/subscriptions/<BIG NUMBA HERE>/resourceGroups/hhRGlab/providers/Microsoft.Compute/virtualMachines/f51waf01",
  "location": "centralus",
  "macAddress": "<A MAC>",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.7",
  "publicIpAddress": "<PUBLIC IP>",
  "resourceGroup": "hhRGlab",
  "zones": ""
}

```

*Please note WAF VM private & public IP *

4. Test your WAF using predefined credentials:

```

ssh -l azureuser <YOUR F5 WAF IP> 

```

A prompt with your user and Active string is a good signal that everything went ok:

```

The authenticity of host '<PUBLIC IP>' can't be established.
ECDSA key fingerprint is SHA256:<A FINGERPRINT YOU ALWAYS IGNORE>.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '<PUBLIC IP>' (ECDSA) to the list of known hosts.
Password: 
azureuser@(localhost)(cfg-sync Standalone)(Active)(/Common)(tmos)# 
azureuser@(localhost)(cfg-sync Standalone)(Active)(/Common)(tmos)# 

```


5. Validate f5 networking. At the TMSH prompt execute:

```

list /net self

```

you'll get your F5 WAF local IP:


```

net self self_1nic {
    address 10.0.0.7/24
    allow-service {
        default
    }
    traffic-group traffic-group-local-only
    vlan internal
}
azureuser@(localhost)(cfg-sync Standalone)(Active)(/Common)(tmos)# 


```

It is on the same net of your vulnerable GraphQL Machine.  


6  Open ports  8443 & 80 for f5

```

az vm open-port -g $rg -n f51waf01 --port 8443 priority 100
az vm open-port -g $rg -n f51waf01 --port 80 priority 101

```
Now your role as Cloud admin has finished. Let's put the SecOps cap!

### Configure your F5 to publish your GraphQL endpoint using GUI. 



1. Using a browser, Sign in using the credentials: azureuser/<YOUR WAF PASSWORD> at https://<WAFIP>:8443

**Note port 8443 since this a 1 NIC deployment

2. Make sure resource provsiniong has: ASM - Nominal,  MGMT - Medium. If that's not the case set them that way 
 
2. Go to Security > Application Security > Security Policies and Clik the + button.

3. On General Settings use:

  - Policy Name: mygqlpolicy1
  - Description: graphQl endpoint protection
  - Policy template: GraphQL Policy
  - Virtual Server: click on + Configure New Virtual Server
  - Virtual Server Name: vs_graphql_sec
  - HTTP Virtual Server Destination : 10.0.0.7
      - Service Port: HTTP
  - HTTP Pool Member: 
      - New Node: 10.0.0.5
      - Service Port: 5000
  - GraphQL Endpoints: Click on +  Create New GraphQL Endpoint in "Allowed URLs"
    - URL: HTTP POST /graphql
    - Click Create  
  
  - Logging Profiles: Log illegal requests
  - Enforcement Mode: Blocking
  - Signature Staging: Disabled

  Click Save

4. You've publish now your app with security. Let's see the objects that were created. Go to Local Traffic  ››  Virtual Servers : Virtual Server List an click vs_graphql virtual server

5. Go to Security > Policies  and you'll see Application Security Policy is enabled and Policy: mygqlpolicy1 is applied.
As you can see, we can extend protection capabilites like IP intenlligence, DoS protection & bot Protection to maximize security 
depending on your needs an threat  model.

6. Let's go back to your app policy to keep tunning it. Go to Security  ››  Application Security : Security Policies : Policies List and click mygqlpolicy1.

7. On GraphQL Endpoints click + Create New GraphQL Endpoint in "Allowed URLs"
  URL: HTTP POST /graphql 
  click create

8. Select your URL and click Enforce then click Ok

9. Click engine icon besides mygqlpolicy1 to go back to your policy

10. On Learning and Blocking click Learning and Blocking Settings

11. On attack signatures go to Signature Set category and click change.

12. On Select Policy Attack Signature Sets window enable:

  	OS Command Injection Signatures
    and click Change  

    and uncheck Enable Signature Staging **Important!!]

13. Click save then click Apply Policy
14. Go to Security  ››  Application Security : Content Profiles : GraphQL Profiles and click Default
15. On Maximum Structure Depth set a value of '10'. Click Update and then Apply policy

# Task 5 - Attack again an see how F5 protects the app

### Do attacks again, now to the graphQL app publish & protected by f5

1. Try Command execution again.

 You're pointing now to the graphql app published by f5, so use its 
 IP and port 80
```


curl -g \
-X POST \
-H "Content-Type: application/json" \
-d '{"query":"mutation {importPaste(host:\"localhost\", port:80, path:\"/ ; netstat -ar\", scheme:\"http\"){result}}"}' \
http://<VM Public IP>graphql

```

you-ll get this:

```
 
  "errors": [
    {
      "message": "The requested query was rejected. Please consult with your administrator. Your support ID is: 1213456921160318980"
    }
  ]

```

2. Go to Security  ››  Event Logs : Application : Requests and you''l see request details. you can see 'netstat' is detected as the string that triggered the violation.

3. Execute a Deep recursion attack now.

```

curl -g -X POST -H "Content-Type: application/json" -d '{"query":"query {pastes {owner {paste {edges {node {owner {paste {edges {node {owner {paste {edges {node {owner {id}}}}}}}}}}}}}}}"}' http://<WAFIP>/graphql

```

As per WAF content GraphQL content profile configuration,you are not allowed to go deeper than 10 levels so

```
"errors": [
    {
      "message": "The requested query was rejected. Please consult with your administrator. Your support ID is: 1213456921160318988"
    }
  ]

```
4. FINISHED!!!

As you can see, protecting graphQL endpoint swith F5 is fast & easy. 

# Task 6 - Delete your resources 

1. in your Azure cloud shell execute:

```
az group delete --name $rg
Are you sure you want to perform this operation? (y/n): y

```
it may take several minutes.

