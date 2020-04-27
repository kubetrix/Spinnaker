# Spinnaker
Spinnaker is an open-source continuous delivery platform for releasing software changes with high velocity and confidence. Through a powerful abstraction layer, Spinnaker provides compelling tooling that empowers developers to own their application code from commit to delivery. As the most mature and widely productionalized continuous delivery platform, Spinnaker can apply the expertise of Netflix, Google, Microsoft, and Amazon to your SDLC. User companies include Target, Salesforce, Airbnb, Cerner, Adobe, and JPMorgan Chase.

Manage your SDLC in Spinnaker using the GUI (graphical user interface), or config-as-code. View, manage, and construct application workflows involving one or all of these resources:

### Why Do I Need Spinnaker?
With Spinnaker, create a “paved road” for application delivery, with guardrails that ensure only valid infrastructure and configuration reach production. Free development teams from burdensome ops provisioning while automating reinforcement of business and regulatory requirements. Delivery automation strategies such as canary deployments provide the safety necessary to capture value from quick innovation, while protecting against business and end-user impact

This section describes how to install and set up Spinnaker so that it can be configured for use in production. 

# Table of Contents
 1. [Halyard Setup](#hal-setup)
 2. [Spinnaker Base Setup with basic concepts (Webhook Setup, Component Sizing)](#spin-setup)
 3. [Enable Kubernetes Provider(Kubernetes Cluster)](#spin-kube)
 4. [Docker Registry Account Addition to Spinnaker](#spin-docker-registry)
 5. [Gitlab Account Addition to Spinnaker for Pull and Push to Repo](#spin-gitlab)
 6. [Jenkins Account Addition to Spinnaker](#spin-jenkins)
 7. [Add Storage Account to Spinnaker for Pipeline configuration](#spin-storageaccount)
 9. [Enable slack notification channel](#spin-slack)
 10. [Enable Helm Support](#spin-helm)
 11. [Enable Canary Deployment](#spin-canary)
 12. [Enable Stats for Tool Usage metrics to share logs with spinnaker team](#spin-stats)
 13. [Enable External Azure Redis with data retention](#spin-redis)
 14. [Secure Spinnaker](#spin-ssl)
      - Basic SSL Configuration
      - Oauth
      - SAML
 15. Deploy from existing configuration
 16. [Spinnaker Deployment Type](#spin-distributed)

##  Halyard Setup <a name="hal-setup"></a>

Halyard is a command-line administration tool that manages the lifecycle of your Spinnaker deployment, including writing & validating your deployment’s configuration, deploying each of Spinnaker’s microservices, and updating the deployment.

All production-capable deployments of Spinnaker require Halyard in order to install, configure, and update Spinnaker.

   #### Download Halyard CLI
          curl -O https://raw.githubusercontent.com/spinnaker/halyard/master/install/debian/InstallHalyard.sh
   #### Install it
          sudo bash InstallHalyard.sh --version 1.33.0-20200325200017
      
| Type | Description |
|:----|:-----------|
|--version(optional)| Set Halyard Version so you know which version you are using |

### Spinnaker Setup with basic concepts <a name= "spin-setup"></a>
       
   - List Spinnaker verion and compatible halyard version          
   
            hal version list
   - Validate current halyard version(If not as per compatibility please update using [Halyard Setup](#hal-setup)) 
   - Set Spinnaker Version(I am using 1.17.10)
   
            hal config version edit --version 1.17.10
   - List current halyard config file and validate version(on-top)
            
            hal config list
    
   - Enable artifact support for storing all generated artifacts and use in different pipelines
        
            hal config features edit --artifacts true
 
 ### Enable Kubernetes Provider(Enable kubernetes cluster accounts) <a name="spin-kube"></a>
    
   - Prerequisite 
           
       - Create kubeconfig file for specific cluster with --admin and move it intto .hal folder(/home/<your-user>/.hal/)
       - For Example I am creating kubeconfig file with --admin
               
                az login(authenticate with user)
                az aks get-credentials -n <kubernetes-cluster-name> -g <cluster-resource-group-name> --admin
                
      **Note :-** <br/>
         1) Move already present config file(/home/<your-user>/.kube/config) at some location before above command<br/>
         2) Then execute above command, you will see newly generated file. <br/>
         3) Move file(config) to .hal folder(/home/<your-user>/.hal/). Move back your backed-up config file to same location.<br/>
        
   - Enable Kubernetes Provider support 
            
            hal config provider kubernetes enable
            
   - Add Kubernetes account(clusters account for deployment)
            
            kubeconfig_path="<above-copied-kubeconfig-file-path>"(kubeconfig_path="/home/<your-user>/cluster-admin-kubeconfig-file")
            hal config provider kubernetes account add <account-name> --provider-version <kubernetes-halyard-provider-version> \
            --kubeconfig-file "$kubeconfig_path" \
            --context $(kubectl config current-context --kubeconfig "$kubeconfig_path") \
            --omit-namespaces=<namespace-name-for-ignore>
        |Type|Description|
        |-----|------|
        |kubeconfig_path|Adding Path variable for passing the kubeconfig file for cluster|
        |--provider|Halyard Kubernetes provider version(Current- v2)|
        |--kubeconfig-file|Path for kubernetes file for pointing to cluster|
        |-- omit-namespaces|Exclude namespaces which you dont want to include in spinnaker deployment|
    
**Note :-** <br/>
      1) You can add multiple clusters for deployment from spinnaker (Target clusters) <br />
      2) Add cluster config at the end in which you want to do spinnaker deployment, As it will take last added cluster as a spinnaker deployment
      
   
   - Add docker-registry account for above created kubernetes account
        (Before executing this step please create docker registry account using (#spin-docker-registry))
        
            hal config provider kubernetes account edit <spinnaker-kubernetes-account> --add-docker-registry <docker-registry-account-name> --live-manifest-calls true
            
        |Parameter|Description|
        |---------|------------|
        |--add-docker-registry|Add docker registry created in Azure/AWS/GCP|
        |--live-manifest-calls|CloudDriver will validate execution with live data instead of cached data(default false)|
        
 
 ### Enable Docker Registry in Spinnaker <a name="spin-docker-registry"></a>
 
   - Enable Docker Registry
            
         hal config provider docker-registry enable
   - Add docker registry
         
         hal config provider docker-registry account add <account-name> --address <host-url > --username <username> --password INNC=<password> --email <email-id>
        
        |Parameter|Description|
        |----------|------------|
        |account-name| Docker Registry Name|
        |host-url| Docker Registry Host URL|
        |username|Docker Registry UserName|
        |password|Docker Registry Password/Token|
 
 ### Generate Gitlab account and Pull/Push some data <a name="spin-gitlab"></a>
   - Considering you already have secured gitlab repo with 1 project
   - Create Gitlab user PAT(Personal Access Token) for pulling the data from gitlab project
        
            - Go to Users(Right-Top) -> Setting -> Access Token(Left)
            - Add token details
         
                name: token-name
                expiry: set as per your need
                access: Add(api, read_user, read_repository)
         
            - Then Create Personal Access Token(Copy the token as it will not be visible after refresh)
            - This will give read access to repository
         
   - Add token to Spinnaker 
        
            hal config artifact gitlab enable
            
            hal config artifact gitlab account add <token-name>
            
            hal config artifact gitlab account edit <token-name> --token <copied-token>
      
      |Paramters|Description|
      |------|-------|
      |token-name| Created token for gitlab|
      |--token | Generated PAT Token|
      
      Note: You can use it same token for writing as well with write access, but better not to provider token directly for writing to generic configuration. Configure it in pipeline steps.
       
   - Create Pipeline using Spinnaker for fetching data
        - Create Spinnaker Application
        - Create Pipeline
        - Adding Configuration
        
                - Add Expected Artifacts
                    - Go to gitlab and open one of the file in gitlab.
                    - Copy RAW url(Open Raw from file top right)
                        https://gitlab******/<user>/<repo>/raw/master/some-file
                        
                    - Change RAW url to API URL for API based access for artifacts download
                        https://gitlab******/api/v4/projects/<user>%2F<repo>/repository/files/<some-file>
                    
                    - Now you can download <some-file> from gitlab to spinnaker
                    - Bake the spinnaker artifact and use further
                  
        - Create Pipeline for pushing data(Continuing with same pipeline)            
                
             Generate repo Webhook token for pushing 
                    
                        - Go to Gitlab -> Project -> Settings -> Integration
                        - Add API URL 
                                https://gitlab******/api/v4/projects/<user>%2F<repo>/repository/commits
                        - Paste Some random value for token and keep with you
                        - Select Push Events and Enable SSL
                        - Click on Add Webhook
                        
             Go To Spinnaker Pipeline
                        
                        - Add Webhook Pipeline
                            Provide Webhook URL(Above one): https://gitlab******/api/v4/projects/<user>%2F<repo>/repository/commits
                            method: POST
                            Payload:
                                    {
                                      "actions": [
                                        {
                                          "action": "update",
                                          "content": "${#fromBase64( #stage( 'Bake (Manifest)' )['outputs']['artifacts'][0]['reference']                                           ) }",
                                          "file_path": "file-location"
                                        }
                                      ],
                                      "branch": "master",
                                      "commit_message": "Spinnaker commit"
                                    }
            
                         'Bake (Manifest)' - Previous Pipeline Step where you generated artifacts
                                             
             Save and Execute Pipeline
                            
                
 ### Enable Jenkins/Jenkins to Spinnaker build <a name="spin-jenkins"></a>

This is required for viewning and managing Jenkins CI Master Configuration for Spinnaker
            
   - Enabling Jenkins and Adding Jenkins Master
                hal config ci jenkins enable
                hal config ci jenkins master add jenkins --address <jenkins-url> --username <username> --password <password> --csrf true
 
   |Parameters|Description|
   |----------|-----------|
   |--address|Jenkins URL|
   |--username|Jenkins Username|
   |--password|Jenkins Password|
   |--csrf|Negotiate Jenkins Token|

### Add Storage Account <a name="spin-storageaccount"></a>

For Storing all pipeline configuration we need to setup some storage account.

In this example I am setting azure(azs) storage account.
   - For Azure Storage Account Creation
        Storage Account - https://docs.microsoft.com/en-us/azure/storage/common/storage-account-create?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json&tabs=azure-portal
        Container Creation - https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-container-create
        
   - Enable Storage account 'azs' and add azs account details. 
        
         hal config storage edit --type azs
         
         hal config storage azs edit --storage-account-name <account-name> --storage-account-key <storage-account-key> --storage-container-name <storageaccount-container-name>
         
     |Parameters|Description|
     |----------|-----------|
     |--type|Setting Storage account type to Azure(azs)|
     |account-name|Azure Storage Account Name|
     |storage-account-key|Storage Account Key (Password)|
     |storageaccount-container-name|Storage account container name, which is created for storage|
     
 Note:- Never use Redis for Configuration Storage, not recommended by Spinnaker. 
   
### Add Slack Notification channel <a name="spin-slack"></a>

it's required for pipeline execution notification.(Good to setup for Pipeline Failure).

  - Enable Slack channel and add configuration
  
            hal config notification slack enable

            hal config notification slack edit --bot-name spinnaker-slack-bot --token ***********

    |Parameters|Description|
    |----------|------------|
    |--bot-name|Slack Bot Name|
    |--token|Add Slack token|
    
  Note:- Email based notification is enabled by default.

### Enable Helm Charts support <a name="spin-helm"></a>
 
  -  Enable Helm chart support
      
         hal config artifact helm enable

### Enable Canary deployment support <a name="spin-canary"></a>

  - Enable Canary deployment
    
        hal config canary enable

### Enable Stats for Tool Usage metrics to share logs with spinnaker team <a name="spin-stats"></a>

If enabled, Spinnaker collects data about how the tool is being used. This data is anonymized (before being sent across the internet) and aggregated

         hal config artifact helm enable
  
Note:- If you don't want to share any logs of execution with spinnaker, then please disable this.

### Enable External Redis <a name="spin-redis"></a>

- Make sure Spinnaker is already deployed, if not then please deploy without adding redis configuration because spinnaker will fail in intial deployment with external redis (Azure)

     - 	Add Redis config file i.e. redis.yml in “spinnaker/default/service-settings/redis.yml”
              
           Redis.yml
              
              overrideBaseUrl: redis://:password@host:port
              skipLifeCycleManagement: true
     
**Note:-**
1) Deploy spinnaker/Generate halyard config(hal config generate) after adding above configuration, donät apply all below mentioned    configuration together, External Redis will not work with all configuration together as url will overwrite.

2) After generation please check spinnaker.yml file present under "default/staging/spinnaker.yml". Search for redis and check the above mentioned configuration added there correctly.
    
- Start deployment and wait for few minutes you will see gate pod is failing but redis is up and running.
-	Now for removing gate pod failure issue, Please add below file.
  
    - Add gate-local.yml file at “spinnaker/ default/profiles/gate-local.yaml”

        Gate-local.yml
                
          redis:
            configuration:
              secure: true

- Do the deployment and validate, redis and now all pods are working fine as well
- 	Now set the retention period for pipeline execution logs. 
     
     https://www.spinnaker.io/setup/productionize/caching/configure-redis-usage/

Note:- It is required for logs cleanup as you never prefer to keep data for long and with azure redis data storage will be very costly as well.


### Secure Spinnaker <a name="spin-ssl"></a>

Spinnaker has multiple options for both authentication and authorization. Instead of reinventing yet-another-login system, Spinnaker hooks into a login system your organization probably already has, such as OAuth 2.0, SAML, or LDAP.

**In this tutorial we are implementing Oauth 2.0, SAML security with organization secured domain certificate**


 ### Distribute Spinnaker deployment <a name="spin-distributed"></a>
   Changing deployment type to distributed i.e.Deploying Spinnaker with one server group per microservice, and a single shared Redis   

            hal config deploy edit --type distributed --account-name
                        
   |Type | Distributed |
   |-----|-------------|
   |--version| Set Deployment Type |
   |--account-name| Spinnaker account name|
