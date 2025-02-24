## SRE Projects - Deploy REST API & Its Dependent Services Using Helm Charts (Assignment 8)

### Problem Statement / Expectations  
We need to create Helm charts for our REST API and its dependent services. We can use the Kubernetes manifests from the previous milestone as a reference. Moving forward, these Helm charts will be used for deployments.  

## Requirements  
- Docker  
- Kind  
- Kubectl  
- Helm (for installing Vault and ESO)  
- Kubernetes cluster (refer to Assignment 6 for details) and complete the necessary steps to proceed directly with Vault configuration.  

## Installation / Quick Start  

### 1) Clone and Navigate:  
```bash
git clone https://github.com/venk404/SRE--RestAPI.git
cd "Assignment 8"
cd helm/charts 
```

### 2) Setup Helm Releases  
Before installing Vault using Helm, we will create two releases: one for the app, database, and External Secrets Operator charts, and another for the Vault charts. Vault requires specific configurations, such as enabling the password engine and setting up a password for ESO to consume. Due to this setup, we have two separate releases. Let's begin with the Vault release.  

## Vault Setup  

### 3) Install the Vault Chart:  
```bash
helm install vault ./vault/ -n vault --create-namespace
```

### 4) Initialize and Unseal Vault:  
```bash
# Wait for the Vault-0 pod to reach the ready state.

kubectl exec vault-0 -n vault -- vault operator init -key-shares=1 -key-threshold=1 -format=json > cluster-keys.json
VAULT_UNSEAL_KEY=$(jq -r '.unseal_keys_b64[0]' cluster-keys.json)
kubectl exec vault-0 -n vault -- vault operator unseal <unseal_token>
```

### 5) Configure Vault Secrets:  
```bash
# Login to Vault using the root token from cluster-keys.json

kubectl exec -it vault-0 -n vault -- /bin/sh
vault login
vault secrets enable -path=secrets kv-v2
vault kv put secrets/DBSECRETS POSTGRES_PASSWORD=<postgres_password> POSTGRES_DB=<postgres_db> POSTGRES_USER=<postgres_user>
exit
```

### 6) Update the Values File for Vault Authentication:  
Locate the `secret` key in the values file, replace the token with the Base64-encoded value from `cluster_keys.json`, and save the file. Follow the command below:  
```bash
echo -n <token> | base64
cd external-secrets
vi values.yaml
```

### 7) Deploy the External Secrets:  
```bash
cd ..
helm install external-secrets .\external-secrets\ -n external-secrets --create-namespace
kubectl get pods -owide -A
```
### 8) Deploy the Restapi and DB:  
```bash
cd ..
helm install restapidb . -n student-api --create-namespace
kubectl get pods -owide -A
```

### 9) Access the API:  
```
http://127.0.0.1:30007/docs
```

### Conclusion  
All expectations were met. In the next chapter, we will deploy everything automatically with ArgoCD using continuous deployment (CD) tools, enabling one-click deployment.
