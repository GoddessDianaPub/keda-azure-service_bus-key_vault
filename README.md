## This repo is to trigger azure service bus when sending messages to topics
It creates HPA to scale pods up and down according your configurations (you might need to delete your existing HPA)

# Install keda: https://keda.sh/docs/2.11/deploy/

- helm repo add kedacore https://kedacore.github.io/charts
- helm repo update
- helm install keda kedacore/keda --namespace keda --create-namespace

kubectl apply -f filename --namespace namespace

- You can create resources and use the secret with [azure service bus connection string](https://keda.sh/docs/2.10/concepts/authentication/#defining-secrets-and-config-maps-on-scaledobject)
- More secure way is to create secret with azure key vault

# Sending messages to service bus in order to check the HPA is working:
https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-dotnet-how-to-use-topics-subscriptions?tabs=connection-string#add-code-to-send-messages-to-the-topic


# Creating secrets with azure key vault: https://keda.sh/docs/2.10/concepts/authentication/#azure-key-vault-secrets
- Register an app with AAD App registrations (you will use it with TriggerAuthentication)
- I used "Keda-KeyVault-Access" registered application which will expire on 8/16/2025
- Create an access policy for the registered app with: "Get, List, Recover" secret permissions
- Service Bus Shared Access Policy needs to be of type Manage: https://keda.sh/docs/2.11/scalers/azure-service-bus/

# Workload identity
# Show the OIDC Issuer URL
az aks show -n myAKScluster -g myResourceGroup --query "oidcIssuerProfile.issuerUrl" -otsv

# Update an AKS cluster with OIDC Issuer
az aks update -g myResourceGroup -n myAKSCluster --enable-oidc-issuer

# Update an AKS cluster with workload-identity
az aks update -g myResourceGroup -n myAKSCluster --enable-workload-identity

# Check AKS Cluster's Identity Configuration
- az aks show --resource-group myResourceGroup --name myAKSCluster --query identity
- az aks show --resource-group myResourceGroup --name myAKSCluster --query identity.type
- az aks show -n myAKScluster -g myResourceGroup --query identityProfile.kubeletidentity.clientId -otsv


# Uninstall keda - you should delete the scaledobjects created in all of your namespaces before uninstalling keda
- kubectl delete $(kubectl get scaledobjects.keda.sh,scaledjobs.keda.sh -A -o jsonpath='{"-n "}{.items[*].metadata.namespace}{" "}{.items[*].kind}{"/"}{.items[*].metadata.name}{"\n"}')
- helm uninstall keda -n keda

- kubectl get scaledobjects.keda.sh,scaledjobs.keda.sh -A -o jsonpath='{"-n "}{.items[*].metadata.namespace}{" "}{.items[*].kind}{"/"}{.items[*].metadata.name}{"\n"}')


# In case of keda namespace is in terminating state, you can follow these steps:

1. kubectl get namespace keda -o json >tmp.json
2. Remove "kubernetes from the finalizers section in tmp.json:
- Before:
    "spec": {
        "finalizers": [
            "kubernetes"
        ]

- After:
    "spec": {
        "finalizers": []

3. Start the proxy: kubectl proxy (and opened a new cmd to continue:
4. Run this command: curl -k -H "Content-Type: application/json" -X PUT --data-binary @tmp.json http://127.0.0.1:8001/api/v1/namespaces/keda/finalize
5. kubectl get namespaces

https://www.ibm.com/docs/el/cloud-private/3.1.2?topic=console-namespace-is-stuck-in-terminating-state