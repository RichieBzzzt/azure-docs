---
author: larryfr
ms.service: machine-learning
ms.topic: include
ms.date: 05/10/2022
ms.author: larryfr
---

### Online endpoint creation fails with a V1LegacyMode == true message

The Azure Machine Learning workspace can be configured for `v1_legacy_mode`, which disables v2 APIs. Managed online endpoints are a feature of the v2 API platform, and won't work if `v1_legacy_mode` is enabled for the workspace. 

> [!IMPORTANT]
> Check with your network security team before disabling `v1_legacy_mode`. It may have been enabled by your network security team for a reason.

For information on how to disable `v1_legacy_mode`, see [Network isolation with v2](/azure/machine-learning/how-to-configure-network-isolation-with-v2).

### Online endpoint creation with key-based authentication fails

Use the following command to list the network rules of the Azure Key Vault for your workspace. Replace `<keyvault-name>` with the name of your key vault:

```azurecli
az keyvault network-rule list -n <keyvault-name>
```

The response for this command is similar to the following JSON document:

```json
{
    "bypass": "AzureServices",
    "defaultAction": "Deny",
    "ipRules": [],
    "virtualNetworkRules": []
}
```

If the value of `bypass` isn't `AzureServices`, use the guidance in the [Configure key vault network settings](../articles/key-vault/general/how-to-azure-key-vault-network-security.md?tabs=azure-cli) to set it to `AzureServices`.

### Online deployments fail with an image download error

1. Check if the `egress-public-network-access` flag is __disabled__ for the deployment. If this flag is enabled, and the visibility of the container registry is private, then this failure is expected.
1. Use the following command to check the status of the private endpoint connection. Replace `<registry-name>` with the name of the Azure Container Registry for your workspace:

    ```azurecli
    az acr private-endpoint-connection list -r <registry-name> --query "[?privateLinkServiceConnectionState.description=='Egress for Microsoft.MachineLearningServices/workspaces/onlineEndpoints'].{Name:name, status:privateLinkServiceConnectionState.status}"
    ```

    In the response document, verify that the `status` field is set to `Approved`. If it isn't approved, use the following command to approve it. Replace `<private-endpoint-name>` with the name returned from the previous command:

    ```azurecli
    az network private-endpoint-connection approve -n <private-endpoint-name>
    ```

### Scoring endpoint can't be resolved

1. Verify that the client issuing the scoring request is a virtual network that can access the Azure Machine Learning workspace.
1. Use the `nslookup` command on the endpoint hostname to retrieve the IP address information:

    ```bash
    nslookup endpointname.westcentralus.inference.ml.azure.com
    ```

    The response contains an __address__. This address should be in the range provided by the virtual network.
1. If the host name isn't resolved by the `nslookup` command, check if an A record exists in the private DNS zone for the virtual network. To check the records, use the following command:

    ```azurecli
    az network private-dns record-set list -z privatelink.api.azureml.ms -o tsv --query [].name
    ```

    The results should contain an entry that is similar to `*.<GUID>.inference.<region>`.
1. If no inference value is returned, delete the private endpoint for the workspace and then recreate it. For more information, see [How to configure a private endpoint](/azure/container-registry/container-registry-private-link). 

### Online deployments can't be scored

1. Use the following command to see if the deployment was successfully deployed:

    ```azurecli
    az ml online-deployment show -e <endpointname> -n <deploymentname> --query '{name:name,state:provisioning_state}' 
    ```

    If the deployment completed successfully, the value of `state` will be `Succeeded`.
1. If the deployment was successful, use the following command to check that traffic is assigned to the deployment. Replace `<endpointname>` with the name of your endpoint:

    ```azurecli
    az ml online-endpoint show -n <endpointname>  --query traffic
    ```

    > [!TIP]
    > This step isn't needed if you are using the `azureml-model-deployment` header in your request to target this deployment.

    The response from this command should list percentage of traffic assigned to deployments.
1. If the traffic assignments (or deployment header) are set correctly, use the following command to get the logs for the endpoint. Replace `<endpointname>` with the name of the endpoint, and `<deploymentname>` with the deployment:

    ```azurecli
    az ml online-deployment get-logs  -e <endpointname> -n <deploymentname> 
    ```

    Look through the logs to see if there's a problem running the scoring code when you submit a request to the deployment.
