**# Download Wazuh repos:**
git clone https://github.com/wazuh/wazuh-kubernetes.git -b v4.7.5 --depth=1
cd wazuh-kubernetes folder


**# Setup SSL certificates for wazuh indexer and dashboard:**
The below genereated certificates are imported via secretGenerator on the kustomization.yml file.

1. Generate self-signed certificates for the Wazuh indexer cluster using the script to the same directory
cd wazuh/certs/indexer_cluster
./generate_certs.sh .
Verify if the certificates are generated

2. Generate self-signed certificates for the Wazuh dashboard cluster using the script to the same directory
Navigate to /dashboard_http folder
./generate_certs.sh .
Verify if the certificates are generated


**# Setup storage class for AKS Cluster:**
kubectl get sc --- check if azue storage class for managed disk is present.
cd envs/local-env
vi storage_class_wazuh.yaml
kubectl get sc  --> check the newly created storage class.


**# Deploy Wazuh to AKS cluster:**
kubectl apply -k envs/local-env/


**# Deployment Verfication:**
kubectl get namespaces | grep wazuh
kubectl get services -n wazuh
kubectl get deployments -n wazuh
kubectl get statefulsets -n wazuh
kubectl get pods -n wazuh


**#Accessing Wazuh Dashboard:**
kubectl get services -o wide -n wazuh
Access the Wazuh dashboard : https://<lb_svc_ip>:443
Use the default credentials to login to Wazuh dashboard


**# Change the default passwords:**
1. Change password for Wazuh indexer users

kubectl exec -it wazuh-indexer-0 -n wazuh -- /bin/bash
export JAVA_HOME=/usr/share/wazuh-indexer/jdk
bash /usr/share/wazuh-indexer/plugins/opensearch-security/tools/hash.sh
Copy the Hash of the new password entered
Exit the bash shell
cd wazuh/indexer_stack/wazuh-indexer/indexer_conf
vi internal_users.yml
Replace the hash generated for both admin and kibana user. Save and exit the conf file.

2. Setting the new password:
Encode your new password in base64 format. void inserting a trailing newline character to maintain the hash value.
echo -n "NewPassword" | base64 ---- copy the new encoded password
Edit the indexer or dashbboard secrets configuration file as follows. Replace the value of the password field with your new encoded password.
move to wazuh folder
cd secrets
vi indexer-cred-secret.yaml
Replace the old password in this file with the new encoded password
vi dashboard-cred-secret.yaml
Replace the old password in this file with the new encoded password

3. Applying the changes:
cd
kubectl apply -k envs/local-env/

4. Start a bash shell in wazuh-indexer-0 once more
kubectl exec -it wazuh-indexer-0 -n wazuh -- /bin/bash

5. Set the following variables:
export INSTALLATION_DIR=/usr/share/wazuh-indexer
CACERT=$INSTALLATION_DIR/certs/root-ca.pem
KEY=$INSTALLATION_DIR/certs/admin-key.pem
CERT=$INSTALLATION_DIR/certs/admin.pem
export JAVA_HOME=/usr/share/wazuh-indexer/jdk

6. Run the securityadmin.sh to apply all the above changes from within the wazuh-indexer-0 container:
wait for the indexer to initialize properly before running the script.
bash /usr/share/wazuh-indexer/plugins/opensearch-security/tools/securityadmin.sh -cd /usr/share/wazuh-indexer/opensearch-security/ -nhnv -cacert  $CACERT -cert $CERT -key $KEY -p 9200 -icl -h $NODE_NAME

7. Login with the new credentials on the Wazuh dashboard
8. 

**# Change password for Wazuh user:**
The wazuh-wui user is the user to connect with the Wazuh API by default. Follow these steps to change the password.

1. Encode your new password in base64 format.
echo -n "NewPassword" | base64

2. cd wazuh/secrets
vi wazuh-api-cred-secret.yaml
Replace then new password in place of the old password

3. kubectl apply -k envs/local-env

4. Restart pods for Wazuh dashboard and Wazuh manager master.
