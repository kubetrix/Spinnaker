# Spinnaker
Spinnaker Production Setup

# Prerequisite
    - Halyard
    - Linux console or Git-Bash
    - 
# Table of Contents
 1. Halyard Setup(#hal-setup)
 2. Spinnaker Base Setup with basic configuration understanding(Webhook Setup, Component Sizing)
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

##  Halyard Setup <a name="hal-setup"></a>

   ####	Downloading Halyard CLI
          curl -O https://raw.githubusercontent.com/spinnaker/halyard/master/install/debian/InstallHalyard.sh
   #### Install it
          sudo bash InstallHalyard.sh --version 1.33.0-20200325200017
      
| Type | Description |
|:----|:-----------|
|--version(optional)| Set Halyard Version so you know which version you are using |

###
