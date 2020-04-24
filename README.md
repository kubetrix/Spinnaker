# Spinnaker
Spinnaker Production Setup

# Prerequisite
    - Halyard
    - Linux console or Git-Bash
    - 
# Table of Contents
 1. [Halyard Setup](#hal-setup)
 2. [Spinnaker Base Setup with basic concepts (Webhook Setup, Component Sizing)](#spin-setup)
 3. [Enable Kubernetes Provider(Kubernetes Cluster)](#spin-kube)
 4. Docker Registry Account Addition to Spinnaker
 5. Gitlab Account Addition to Spinnaker  
 6. Jenkins Account Addition to Spinnaker  
 7. Add Storage Account to Spinnaker for Pipeline configuration
 8. Enable Artifact Support
 9. Enable slack notification channel
 10. Enable Helm Support
 11. Enable Canary Deployment
 12. Enable Stats
 13. Enable External Azure Redis with data retention
 14. Add Security
      - Basic SSL Configuration
      - Oauth
      - SAML
 15. Deploy from existing configuration
 16. [Spinnaker Deployment Type](#spin-distributed)

##  Halyard Setup <a name="hal-setup"></a>

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
 
 ### Distribute Spinnaker deployment <a name="spin-distributed"></a>
   Changing deployment type to distributed i.e.Deploying Spinnaker with one server group per microservice, and a single shared Redis   

            hal config deploy edit --type distributed --account-name
   |Type | Distributed |
   |-----|-----|
   |--version| Set Deployment Type |
   |--account-name| Spinnaker account name|
