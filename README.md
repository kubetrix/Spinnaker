# Spinnaker
Spinnaker Production Setup

# Prerequisite
    - Halyard
    - Linux console or Git-Bash
    - 
# Table of Contents
 1. [Halyard Setup](#hal-setup)
 2. [Spinnaker Base Setup with basic concepts (Webhook Setup, Component Sizing)](#spin-setup)
 3. Add Kubernetes Provider(Kubernetes Cluster)
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
            
 ### Distribute Spinnaker deployment <a name="spin-distributed"></a>
   Changing deployment type to distributed i.e.Deploying Spinnaker with one server group per microservice, and a single shared Redis   

            hal config deploy edit --type distributed --account-name
   |Type | Distributed |
   |-----|-----|
   |--version| Set Deployment Type |
   |--account-name| Spinnaker account name|
