## This repo is to trigger azure service bus when sending messages to topics
It creates HPA to scale pods up and down according your configurations (you might need to delete your existing HPA)

# Make your changes to the yaml file, and run this command in order to create the resources:
kubectl apply -f < filename > --namespace < namespace >

- You can create resources and use the secret with [azure service bus connection string](https://keda.sh/docs/2.10/concepts/authentication/#defining-secrets-and-config-maps-on-scaledobject)
- More secure way is to create secret with azure key vault

# Creating secret with [azure key vault](https://keda.sh/docs/2.10/concepts/authentication/#azure-key-vault-secrets):
- Register an app with AAD App registrations (you will use it with TriggerAuthentication)
- Create an access policy for the registered app with: "Get, List, Recover" secret permissions
- Service Bus Shared Access Policy needs to be of type Manage. You can refer to [this article](https://keda.sh/docs/2.11/scalers/azure-service-bus/)

# Sending messages to service bus in order to check the HPA is working:
You can either use azure service bus explorer or [this code](https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-dotnet-how-to-use-topics-subscriptions?tabs=connection-string#add-code-to-send-messages-to-the-topic)

# Commands you can use:
- kubectl get hpa <hpa_name> --namespace <namespace>
- kubectl get deploy <deployment_name> --namespace <namespace>
- kubectl get pod <pod_name> --namespace <namespace>
- kubectl get secret <secret_name> --namespace <namespace>
- kubectl get TriggerAuthentication <TriggerAuthentication_name> --namespace <namespace>
- kubectl get ScaledObject <ScaledObject_name> --namespace <namespace>
- kubectl describe ScaledObject <ScaledObject_name> --namespace <namespace>
