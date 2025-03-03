---
title: What are endpoints?
titleSuffix: Azure Machine Learning
description: Learn how Azure Machine Learning endpoints to simplify machine learning deployments.
services: machine-learning
ms.service: machine-learning
ms.subservice: mlops
ms.topic: conceptual
ms.author: seramasu
author: rsethur
ms.reviewer: larryfr
ms.custom: devplatv2, ignite-fall-2021, event-tier1-build-2022
ms.date: 05/24/2022
#Customer intent: As an MLOps administrator, I want to understand what a managed endpoint is and why I need it.
---

# What are Azure Machine Learning endpoints?

[!INCLUDE [dev v2](../../includes/machine-learning-dev-v2.md)]


Use Azure Machine Learning endpoints to streamline model deployments for both real-time and batch inference deployments. Endpoints provide a unified interface to invoke and manage model deployments across compute types.

In this article, you learn about:
> [!div class="checklist"]
> * Endpoints
> * Deployments
> * Managed online endpoints
> * Kubernetes online endpoints
> * Batch inference endpoints

## What are endpoints and deployments?

After you train a machine learning model, you need to deploy the model so that others can use it to do inferencing. In Azure Machine Learning, you can use **endpoints** and **deployments** to do so.

An **endpoint** is an HTTPS endpoint that clients can call to receive the inferencing (scoring) output of a trained model. It provides: 
- Authentication using "key & token" based auth 
- SSL termination 
- A stable scoring URI (endpoint-name.region.inference.ml.azure.com)


A **deployment** is a set of resources required for hosting the model that does the actual inferencing. 

A single endpoint can contain multiple deployments. Endpoints and deployments are independent Azure Resource Manager resources that appear in the Azure portal.

Azure Machine Learning uses the concept of endpoints and deployments to implement different types of endpoints: [online endpoints](#what-are-online-endpoints) and [batch endpoints](#what-are-batch-endpoints).

### Multiple developer interfaces

Create and manage batch and online endpoints with multiple developer tools:
- The Azure CLI
- Azure Resource Manager/REST API
- Azure Machine Learning studio web portal
- Azure portal (IT/Admin)
- Support for CI/CD MLOps pipelines using the Azure CLI interface & REST/ARM interfaces

## What are online endpoints?

**Online endpoints** are endpoints that are used for online (real-time) inferencing. Compared to **batch endpoints**, **online endpoints** contain **deployments** that are ready to receive data from clients and can send responses back in real time.

The following diagram shows an online endpoint that has two deployments, 'blue' and 'green'. The blue deployment uses VMs with a CPU SKU, and runs v1 of a model. The green deployment uses VMs with a GPU SKU, and uses v2 of the model. The endpoint is configured to route 90% of incoming traffic to the blue deployment, while green receives the remaining 10%.

:::image type="content" source="media/concept-endpoints/endpoint-concept.png" alt-text="Diagram showing an endpoint splitting traffic to two deployments.":::

### Online deployments requirements

To create an online endpoint, you need to specify the following elements:
- Model files (or specify a registered model in your workspace) 
- Scoring script - code needed to do scoring/inferencing
- Environment - a Docker image with Conda dependencies, or a dockerfile 
- Compute instance & scale settings 

Learn how to deploy online endpoints from the [CLI](how-to-deploy-managed-online-endpoints.md) and the [studio web portal](how-to-use-managed-online-endpoint-studio.md).

### Test and deploy locally for faster debugging

Deploy locally to test your endpoints without deploying to the cloud. Azure Machine Learning creates a local Docker image that mimics the Azure ML image. Azure Machine Learning will build and run deployments for you locally, and cache the image for rapid iterations.

### Native blue/green deployment 

Recall, that a single endpoint can have multiple deployments. The online endpoint can do load balancing to give any percentage of traffic to each deployment.

Traffic allocation can be used to do safe rollout blue/green deployments by balancing requests between different instances.

> [!TIP]
> A request can bypass the configured traffic load balancing by including an HTTP header of `azureml-model-deployment`. Set the header value to the name of the deployment you want the request to route to.

:::image type="content" source="media/concept-endpoints/traffic-allocation.png" alt-text="Screenshot showing slider interface to set traffic allocation between deployments.":::

:::image type="content" source="media/concept-endpoints/endpoint-concept.png" alt-text="Diagram showing an endpoint splitting traffic to two deployments.":::

Traffic to one deployment can also be mirrored (copied) to another deployment. Mirroring is useful when you want to test for things like response latency or error conditions without impacting live clients. For example, a blue/green deployment where 100% of the traffic is routed to blue and a 10% is mirrored to the green deployment. With mirroring, the results of the traffic to the green deployment aren't returned to the clients but metrics and logs are collected. Mirror traffic functionality is a __preview__ feature.

:::image type="content" source="media/concept-endpoints/endpoint-concept-mirror.png" alt-text="Diagram showing an endpoint mirroring traffic to a deployment.":::

Learn how to [safely rollout to online endpoints](how-to-safely-rollout-managed-endpoints.md).

### Application Insights integration

All online endpoints integrate with Application Insights to monitor SLAs and diagnose issues. 

However [managed online endpoints](#managed-online-endpoints-vs-kubernetes-online-endpoints) also include out-of-box integration with Azure Logs and Azure Metrics.

### Security

- Authentication: Key and Azure ML Tokens
- Managed identity: User assigned and system assigned
- SSL by default for endpoint invocation

### Autoscaling

Autoscale automatically runs the right amount of resources to handle the load on your application. Managed endpoints support autoscaling through integration with the [Azure monitor autoscale](../azure-monitor/autoscale/autoscale-overview.md) feature. You can configure metrics-based scaling (for instance, CPU utilization >70%), schedule-based scaling (for example, scaling rules for peak business hours), or a combination.

:::image type="content" source="media/concept-endpoints/concept-autoscale.png" alt-text="Screenshot showing that autoscale flexibly provides between min and max instances, depending on rules.":::

### Visual Studio Code debugging

Visual Studio Code enables you to interactively debug endpoints.

:::image type="content" source="media/concept-endpoints/visual-studio-code-full.png" alt-text="Screenshot of endpoint debugging in VSCode." lightbox="media/concept-endpoints/visual-studio-code-full.png" :::

### Private endpoint support (preview)

Optionally, you can secure communication with a managed online endpoint by using private endpoints. This functionality is currently in preview.

[!INCLUDE [preview disclaimer](../../includes/machine-learning-preview-generic-disclaimer.md)]

You can configure security for inbound scoring requests and outbound communications with the workspace and other services separately. Inbound communications use the private endpoint of the Azure Machine Learning workspace. Outbound communications use private endpoints created per deployment.

For more information, see [Secure online endpoints](how-to-secure-online-endpoint.md).

## Managed online endpoints vs Kubernetes online endpoints

There are two types of online endpoints: **managed online endpoints** and **Kubernetes online endpoints**. 

Managed online endpoints help to deploy your ML models in a turnkey manner. Managed online endpoints work with powerful CPU and GPU machines in Azure in a scalable, fully managed way. Managed online endpoints take care of serving, scaling, securing, and monitoring your models, freeing you from the overhead of setting up and managing the underlying infrastructure. The main example in this doc uses managed online endpoints for deployment. 

Kubernetes online endpoint allows you to deploy models and serve online endpoints at your fully configured and managed [Kubernetes cluster anywhere](./how-to-attach-kubernetes-anywhere.md),with CPUs or GPUs.

The following table highlights the key differences between managed online endpoints and Kubernetes online endpoints. 

|  | Managed online endpoints | Kubernetes online endpoints |
|-|-|-|
| **Recommended users** | Users who want a managed model deployment and enhanced MLOps experience | Users who prefer Kubernetes and can self-manage infrastructure requirements |
| **Infrastructure management** | Managed compute provisioning, scaling, host OS image updates, and security hardening | User responsibility |
| **Compute type** | Managed (AmlCompute) | Kubernetes cluster (Kubernetes) |
| **Out-of-box monitoring** | [Azure Monitoring](how-to-monitor-online-endpoints.md) <br> (includes key metrics like latency and throughput) | Supported |
| **Out-of-box logging** | [Azure Logs and Log Analytics at endpoint level](how-to-deploy-managed-online-endpoints.md#optional-integrate-with-log-analytics) | 	Unsupported |
| **Application Insights** | Supported | Supported |
| **Managed identity** | [Supported](how-to-access-resources-from-endpoints-managed-identities.md) | Supported |
| **Virtual Network (VNET)** | [Supported](how-to-secure-online-endpoint.md) (preview) | Supported |
| **View costs** | [Endpoint and deployment level](how-to-view-online-endpoints-costs.md) | Cluster level |
| **Mirrored traffic** | [Supported](how-to-safely-rollout-managed-endpoints.md#test-the-deployment-with-mirrored-traffic-preview) | Unsupported

### Managed online endpoints

Managed online endpoints can help streamline your deployment process. Managed online endpoints provide the following benefits over Kubernetes online endpoints:

- Managed infrastructure
    - Automatically provisions the compute and hosts the model (you just need to specify the VM type and scale settings) 
    - Automatically updates and patches the underlying host OS image
    - Automatic node recovery if there's a system failure

- Monitoring and logs
    - Monitor model availability, performance, and SLA using [native integration with Azure Monitor](how-to-monitor-online-endpoints.md).
    - Debug deployments using the logs and native integration with Azure Log Analytics.

    :::image type="content" source="media/concept-endpoints/log-analytics-and-azure-monitor.png" alt-text="Screenshot showing Azure Monitor graph of endpoint latency.":::

- View costs 
    - Managed online endpoints let you [monitor cost at the endpoint and deployment level](how-to-view-online-endpoints-costs.md)
    
    :::image type="content" source="media/concept-endpoints/endpoint-deployment-costs.png" alt-text="Screenshot cost chart of an endpoint and deployment.":::

    > [!NOTE]
    > Managed online endpoints are based on Azure Machine Learning compute. When using a managed online endpoint, you pay for the compute and networking charges. There is no additional surcharge.
    >
    > If you use a virtual network and secure outbound (egress) traffic from the managed online endpoint, there is an additional cost. For egress, three private endpoints are created _per deployment_ for the managed online endpoint. These are used to communicate with the default storage account, Azure Container Registry, and workspace. Additional networking charges may apply. For more information on pricing, see the [Azure pricing calculator](https://azure.microsoft.com/pricing/calculator/).

For a step-by-step tutorial, see [How to deploy online endpoints](how-to-deploy-managed-online-endpoints.md).

## What are batch endpoints?

**Batch endpoints** are endpoints that are used to do batch inferencing on large volumes of data over a period of time.  **Batch endpoints** receive pointers to data and run jobs asynchronously to process the data in parallel on compute clusters. Batch endpoints store outputs to a data store for further analysis.

:::image type="content" source="media/concept-endpoints/batch-endpoint.png" alt-text="Diagram showing that a single batch endpoint may route requests to multiple deployments, one of which is the default.":::

### Batch deployment requirements

To create a batch deployment, you need to specify the following elements:

- Model files (or specify a model registered in your workspace)
- Compute
- Scoring script - code needed to do the scoring/inferencing
- Environment - a Docker image with Conda dependencies

If you're deploying [MLFlow models](how-to-train-cli.md#model-tracking-with-mlflow), there's no need to provide a scoring script and execution environment, as both are autogenerated.

Learn how to [deploy and use batch endpoints with the Azure CLI](how-to-use-batch-endpoint.md) and the [studio web portal](how-to-use-batch-endpoints-studio.md)

### Managed cost with autoscaling compute

Invoking a batch endpoint triggers an asynchronous batch inference job. Compute resources are automatically provisioned when the job starts, and automatically de-allocated as the job completes. So you only pay for compute when you use it.

You can [override compute resource settings](how-to-use-batch-endpoint.md#configure-the-output-location-and-overwrite-settings) (like instance count) and advanced settings (like mini batch size, error threshold, and so on) for each individual batch inference job to speed up execution and reduce cost.

### Flexible data sources and storage

You can use the following options for input data when invoking a batch endpoint:

- Cloud data - Either a path on Azure Machine Learning registered datastore, a reference to Azure Machine Learning registered V2 data asset, or a public URI. For more information, see [Connect to data with the Azure Machine Learning studio](how-to-connect-data-ui.md)
- Data stored locally - it will be automatically uploaded to the Azure ML registered datastore and passed to the batch endpoint.

> [!NOTE]
> - If you are using existing V1 FileDataset for batch endpoint, we recommend migrating them to V2 data assets and refer to them directly when invoking batch endpoints. Currently only data assets of type `uri_folder` or `uri_file` are supported. Batch endpoints created with GA CLIv2 (2.4.0 and newer) or GA REST API (2022-05-01 and newer) will not support V1 Dataset.
> - You can also extract the URI or path on datastore extracted from V1 FileDataset by using `az ml dataset show` command with `--query` parameter and use that information for invoke.
> - While Batch endpoints created with earlier APIs will continue to support V1 FileDataset, we will be adding further V2 data assets support with the latest API versions for even more usability and flexibility. For more information on V2 data assets, see [Work with data using SDK v2 (preview)](how-to-use-data.md). For more information on the new V2 experience, see [What is v2](concept-v2.md).

For more information on supported input options, see [Batch scoring with batch endpoint](how-to-use-batch-endpoint.md#invoke-the-batch-endpoint-with-different-input-options).

Specify the storage output location to any datastore and path. By default, batch endpoints store their output to the workspace's default blob store, organized by the Job Name (a system-generated GUID).

### Security

- Authentication: Azure Active Directory Tokens
- SSL: enabled by default for endpoint invocation
- VNET support: Batch endpoints support ingress protection. A batch endpoint with ingress protection will accept scoring requests only from hosts inside a virtual network but not from the public internet. A batch endpoint that is created in a private-link enabled workspace will have ingress protection. To create a private-link enabled workspace, see [Create a secure workspace](tutorial-create-secure-workspace.md).

## Next steps

- [How to deploy online endpoints with the Azure CLI](how-to-deploy-managed-online-endpoints.md)
- [How to deploy batch endpoints with the Azure CLI](how-to-use-batch-endpoint.md)
- [How to use online endpoints with the studio](how-to-use-managed-online-endpoint-studio.md)
- [Deploy models with REST](how-to-deploy-with-rest.md)
- [How to monitor managed online endpoints](how-to-monitor-online-endpoints.md)
- [How to view managed online endpoint costs](how-to-view-online-endpoints-costs.md)
- [Manage and increase quotas for resources with Azure Machine Learning](how-to-manage-quotas.md#azure-machine-learning-managed-online-endpoints)
