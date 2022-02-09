# Access keyvault secret from AKS.

```
1. helm repo add csi-secrets-store-provider-azure https://raw.githubusercontent.com/Azure/secrets-store-csi-driver-provider-azure/master/charts
```

```
2. helm install csi csi-secrets-store-provider-azure/csi-secrets-store-provider-azure --namespace kube-system
```

> ``Important: It’s recommended to install the Azure Key Vault Provider for Secrets Store CSI Driver in the kube-system namespace using Helm.``

#### Why kube-system

* The driver and provider are installed as a DaemonSet with the ability to mount kubelet hostPath volumes and view pod service account tokens. It should be treated as privileged and regular cluster users should not have permissions to deploy or modify the driver.
* For AKS clusters with limited egress traffic, installing the driver and provider in kube-system is required to be able to establish connectivity to the kube-apiserver. Refer to #488 for more details.
* The driver pods need to run as root to mount the volume as tmpfs in the pod. Deploying the driver and provider in kube-system will prevent ASC from generating alert “Running containers as root user should be avoided”. Refer to #327 for more details.

To validate the driver is running as expected, run the following command:

```
kubectl get pods -l app=csi-secrets-store-provider-azure -n kube-system
```

#### Create User Managed Identity using portal

#### Create Storage Provider class

```
# This is a SecretProviderClass example using user-assigned identity to access Keyvault
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: azure-kvname-user-msi
spec:
  provider: azure
  parameters:
    usePodIdentity: "false"
    useVMManagedIdentity: "true"                                      # Set to true for using managed identity
    userAssignedIdentityID: "<client id of user assigned identity>"   # Set the clientID of the user-assigned managed identity to use
    keyvaultName: "kvname"
    cloudName: ""                                                     # [OPTIONAL for Azure] if not provided, azure environment will default to AzurePublicCloud
    objects:  |
      array:
        - |
          objectName: secret1
          objectType: secret                                          # object types: secret, key or cert
          objectVersion: ""                                           # [OPTIONAL] object versions, default to latest if empty
        - |
          objectName: key1
          objectType: key
          objectVersion: ""
    tenantId: "tid"                                                   # the tenant ID of the KeyVault
```

Ref:

1. https://azure.github.io/secrets-store-csi-driver-provider-azure/getting-started/installation/
2. https://secrets-store-csi-driver.sigs.k8s.io/#project-status
3. https://github.com/Azure/secrets-store-csi-driver-provider-azure/blob/master/examples/user-assigned-managed-identity/v1alpha1_secretproviderclass_user_assigned_identity.yaml

