# apiProtectionLabAlpha
# Protecting APIs in Azure  with F5#

Welcome to API protection Lab. This time you'll work with GraphQL APIs running in an ACI (Azure Container Instance)

# Pre-requisites
- An azure account
- F5 Advanced WAF BIG-IP Trial License

# GOAL: 

Protect vulnerable GraphQL endpoint running in an Azure VM with F5 Advanced WAF

# Activities: #

1. Create Azure infrastructure using Azure Cloudshell
2. Create GraphQL application on a linux VM
3. Do some attacks
4. Deploy F5 Adv WAF from Azure Cloudshell & configure it declaratively
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
rg=hhRGlab
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
2.  ;;;;VALIDATE IMPACT OF THIS ONE;;;


```

curl -g \
-X POST \
-H "Content-Type: application/json" \
-d '{"query":"query {pastes {owner {paste {edges {node {owner {paste {edges {node {owner {paste {edges {node {owner {id}}}}}}}}}}}}}}}"}' \
http://<VM Public IP>:5000/graphql

```
Note the time it takes to answer this single request. This may lead to DoS increasing the number of requests.
the next request does a similar effect but tajes more time

#### Resource Intensive Query Attack
 3.  This query will be more computationally expensive on resourcethan the latter, consuming more time and causing impact on infrastructure.
  

```
 time curl -g \
 -X POST 
  -H "Content-Type: application/json" 
   -d '{"query":"query {\n  systemUpdate\n}","variables":[]}' http://23.101.113.30:5000/graphql
  

```
It took a little time huh?

```

{"data":{"systemUpdate":"no updates available"}}
real    0m50.288s
user    0m0.011s
sys     0m0.000s


```
There are lots of attacks you can do on graphQL endpoints that can compromise underlying infrastructure so yes, you need to protect them as well with WAF that understands this arcitectural style.



  # Task 4 - Deploy F5 Adv WAF from Azure Cloudshell & configure it declaratively

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

3. create your f5 WAF:

  ```

  az vm create -n f51waf01 -g $rg --image f5-networks:f5-big-ip-advanced-waf:f5-big-awf-plus-hourly-25mbps:16.1.100000 --admin-username azureuser --admin-password <YOU F5 PASSWORD>  

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

Please note VM private & public IP 

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

#### timestamp 202111290



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

It is on the same net of your vulnerable GraphQL Machine. We will use that IP as pool member in our AS3 declaration

#### timestamp 2021121400

5. Configure your F5 to publish your GraphQL endpoint using AS3. so let's go and install the RPM file ont the WAF.
On your cloudshell console execute:

```

wget https://github.com/F5Networks/f5-appsvcs-extension/releases/download/v3.33.0/f5-appsvcs-3.33.0-4.noarch.rpm

```

You'll get this as a result:

```
wget https://github.com/F5Networks/f5-appsvcs-extension/releases/download/v3.33.0/f5-appsvcs-3.33.0-4.noarch.rpm
--2021-12-15 04:44:48--  https://github.com/F5Networks/f5-appsvcs-extension/releases/download/v3.33.0/f5-appsvcs-3.33.0-4.noarch.rpm
Resolving github.com (github.com)... 140.82.113.4
Connecting to github.com (github.com)|140.82.113.4|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/130101810/977b7d13-1ad6-49b7-917b-fcf412594ecf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20211215%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20211215T044448Z&X-Amz-Expires=300&X-Amz-Signature=60f9744318a63dd48bbeded4d06870a1116f68fd479d60d655e5104c88bac717&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=130101810&response-content-disposition=attachment%3B%20filename%3Df5-appsvcs-3.33.0-4.noarch.rpm&response-content-type=application%2Foctet-stream [following]
--2021-12-15 04:44:48--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/130101810/977b7d13-1ad6-49b7-917b-fcf412594ecf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20211215%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20211215T044448Z&X-Amz-Expires=300&X-Amz-Signature=60f9744318a63dd48bbeded4d06870a1116f68fd479d60d655e5104c88bac717&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=130101810&response-content-disposition=attachment%3B%20filename%3Df5-appsvcs-3.33.0-4.noarch.rpm&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.111.133, 185.199.108.133, 185.199.109.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.111.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 21353286 (20M) [application/octet-stream]
Saving to: ‘f5-appsvcs-3.33.0-4.noarch.rpm’

f5-appsvcs-3.33.0-4.noarch.rpm      100%[=================================================================>]  20.36M  21.2MB/s    in 1.0s

2021-12-15 04:44:50 (21.2 MB/s) - ‘f5-appsvcs-3.33.0-4.noarch.rpm’ saved [21353286/21353286]


```

4. Do a pwd to know your current directoy

```

h_heredia@Azure:~$ pwd
/home/h_heredia
h_heredia@Azure:~$

```

6. We'l take into account this path to complete the next command to upload & install AS3 onto the bigip




```

  ./install-rpm.sh <YOU F5 WAF PUBLIC IP> azureuser:<YOU F5 PASSWORD> /home/h_heredia/f5-appsvcs-3.33.0-4.noarch.rpm

```
finishing...


```
./install-rpm.sh <YOU F5 WAF PUBLIC IP> azureuser:<YOU F5 PASSWORD> /home/h_heredia/f5-appsvcs-3.33.0-4.noarch.rpm
Uploading RPM to https://<F5 WAF PUBLIC IP>:8443/mgmt/shared/file-transfer/uploads/f5-appsvcs-3.33.0-4.noarch.rpm
Installing f5-appsvcs-3.33.0-4.noarch.rpm on <F5 WAF PUBLIC IP>
Waiting for /info endpoint to be available
Installed f5-appsvcs-3.33.0-4.noarch.rpm on <F5 WAF PUBLIC IP>

```

AS3 is installed, now we can configure f5 declaratively :)

7. Before we proceed with config let's get a token first:

```
a curl -k -X POST https://<YOU F5 WAF PUBLIC IP>:8443/mgmt/shared/authn/login -H "Content-type: application/json"  -H "Host:40.86.90.125" -d '{
    "username":"azureuser",
    "password":"<YOU F5 PASSWORD>",
    "loginProviderName":"tmos"
}'

``` 
From the output you'll see a token value (after 'token:'):

```
{"username":"azureuser","loginReference":{"link":"https://localhost/mgmt/cm/system/authn/providers/tmos/1f44a60e-11a7-3c51-a49f-82983026b41b/login"},"loginProviderName":"tmos","token":{"token":"QYRBXDSCPZW2LGLGEKIB3E23L5","name":"QYRBXDSCPZW2LGLGEKIB3E23L5","userName":"azureuser","authProviderName":"tmos","user":{"link":"https://localhost/mgmt/shared/authz/users/azureuser"},"timeout":1200,"startTime":"2021-12-14T22:22:11.851-0800","address":"187.161.181.202","partition":"[All]","generation":1,"lastUpdateMicros":1639549331850940,"expirationMicros":1639550531851000,"kind":"shared:authz:tokens:authtokenitemstate","selfLink":"https://localhost/mgmt/shared/authz/tokens/QYRBXDSCPZW2LGLGEKIB3E23L5"},"generation":0,"lastUpdateMicros":0}heredia@FLD-ML-00054376:~$ 


```

Get it and use it for the next request as value of X-F5-Auth-Token header in order to valide AS3 installation & token validity: 

```

 curl -k https://<YOU F5 WAF PUBLIC IP>:8443/mgmt/shared/appsvcs/info  -H "Content-type: application/json" -H "Host:<YOU F5 WAF PUBLIC IP>" -H "X-F5-Auth-Token: <YOUR VALID TOKEN>"
 
```

You'll get your confirmation back as follows:

```

{"version":"3.33.0","release":"4","schemaCurrent":"3.33.0","schemaMinimum":"3.0.0"}

```

GREAT!!! Lets configure our WAF in a declarative way.

8. First we will send and AS3 declaration to publish thru f5 the graphQL app:




```

 curl -X POST -k https://40.86.90.125:8443/mgmt/shared/appsvcs/declare  -H "Content-type: application/json" -H "Host:<YOU F5 WAF PUBLIC IP>" -H "X-F5-Auth-Token: <YOUR VALID TOKEN>" \
  -d '{
     "class": "AS3",
     "action": "deploy",
     "persist": true,
     "declaration": {
         "class": "ADC",
         "schemaVersion": "3.0.0",
         "id": "2021121400",
         "label": "vs4graph",
         "remark": "VS to protect GraphQL endpoint",
         "WAF_01": {
             "class": "Tenant",
             "defaultRouteDomain": 0,
             "Application_1": {
                 "class": "Application",
                 "template": "http",
             "serviceMain": {
                 "class": "Service_HTTP",
                 "virtualAddresses": [
                     "10.0.0.7"
                 ],
                 "pool": "api_pool"
                 },
                 "api_pool": {
                     "class": "Pool",
                     "monitors": [
                         "http"
                     ],
                     "members": [
                         {
                             "servicePort": 5000,
                             "serverAddresses": [
                                 "10.0.0.5"
                             ]
                         }
                     ]
                 }
             }
         }
     }
 }' 
  


  ```
 9. F5 WAF will confirm back:


 ```

{"results":[{"code":200,"message":"success","lineCount":24,"host":"localhost","tenant":"WAF_01","runTime":3650}],"declaration":{"class":"ADC","schemaVersion":"3.0.0","id":"2021121400","label":"vs4graph","remark":"VS to protect GraphQL endpoint","WAF_01":{"class":"Tenant","defaultRouteDomain":0,"Application_1":{"class":"Application","template":"http","serviceMain":{"class":"Service_HTTP","virtualAddresses":["10.0.0.7"],"pool":"api_pool"},"api_pool":{"class":"Pool","monitors":["http"],"members":[{"servicePort":5000,"serverAddresses":["10.0.0.5"]}]}}},"updateMode":"selective","controls":{"archiveTimestamp":"2021-12-15T05:44:21.817Z"}}}h_heredia@Azure:~$


 ```





 10. Try accessing your API by curling to it pointing to F5 WAF public ip to port 80:


 ```
curl http://<YOU F5 WAF PUBLIC IP> | grep graphql
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  7955  100  7955    0     0  40794      0 --:--:-- --:--:-- --:--:-- 40794
          <a href="https://labs.bishopfox.com/tech-blog/design-considerations-for-secure-graphql-apis" target="_blank">BishopFox Labs - Design Considerations</a>
          <a href="https://leapgraph.com/graphql-api-security" target="_blank">Leap Graph - How to secure GraphQL APIs</a>


 ```

 We grep so output it is reduced :)


11. Now that we publish thru our WAF this graphQL app, let's protect it!

:::::::::;
::::::::::

 # Task 5 - Attack again an see how F5 protects the app