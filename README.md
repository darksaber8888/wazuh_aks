**Steps to Install Wazuh on AKS Cluster**
  Refer to the official link:
  https://documentation.wazuh.com/current/deployment-options/deploying-with-kubernetes/kubernetes-deployment.html

**1. Download Wazuh repos:**
     git clone https://github.com/wazuh/wazuh-kubernetes.git -b v4.7.5 --depth=1

**2. Setup SSL certificates for wazuh indexer and dashboard:**
  The below genereated certificates are imported via secretGenerator on the kustomization.yml file.
  a. Generate self-signed certificates for the Wazuh indexer cluster using the script to the same directory
  b. Generate self-signed certificates for the Wazuh dashboard cluster using the script to the same directory

**3. Setup storage class for AKS Cluster:**
  a. Check if azue storage class for managed disk is present.
  b. Create a Storage Class object using th Yaml manifest.
  c. Verify if Storage Class is created.

**4. Deploy Wazuh to AKS cluster:**
     Deploy all Kubernetes objects using manifests files using <kustomize>.
     
**5. Deployment Verfication:**
     Verify if the namespace, deployments, statefulesets, pods and services required to run wazuh are installed correctly.

**6. Accessing Wazuh Dashboard:**
     Access the Wazuh dashboard : https://<lb_svc_ip>:443
     Use the default credentials to login to Wazuh dashboard

**7. Change the default passwords:**
  a. Change password for Wazuh indexer users. Replace the hash generated for both admin and kibana user in the <conf> file.
  b. Set the new password. Encode your new password in base64 format. void inserting a trailing newline character to maintain the hash value.Replace the old password in the <indexer-cred-secret.yaml> file with         the new encoded password. Replace the old password in <dashboard-cred-secret.yaml> file with the new encoded password
  c. Applying the changes:
  d. Start a bash shell in <wazuh-indexer-0> once more
  e. Set the required variables in the <wazuh-indexer-0> pod.
  f. Run the "securityadmin.sh" to apply all the above changes from within the <wazuh-indexer-0> container. Wait for the indexer to initialize properly before running the bash script.
  g. Login with the new credentials on the Wazuh dashboard

**8. Change password for Wazuh user:**
  The wazuh-wui user is the user to connect with the Wazuh API by default. 
  a. Encode your new password in base64 format.
  b. Replace the new password in place of the old password in the <wazuh-api-cred-secret.yaml> file.  
  c. Apply the changes with the kustomization file. 
  d. Restart pods for Wazuh dashboard and Wazuh manager master.
