
# Using low priority VMs in batch deployments

[!INCLUDE [cli v2](../../includes/machine-learning-dev-v2.md)]

Azure Batch Deployments supports low priority VMs to reduce the cost of batch inference workloads. Low priority VMs enable a large amount of compute power to be used for a low cost. Low priority VMs take advantage of surplus capacity in Azure. When you specify low priority VMs in your pools, Azure can use this surplus, when available.

The tradeoff for using them is that those VMs may not always be available to be allocated, or may be preempted at any time, depending on available capacity. For this reason, __they are most suitable for batch and asynchronous processing workloads__ where the job completion time is flexible and the work is distributed across many VMs.

Low priority VMs are offered at a significantly reduced price compared with dedicated VMs. For pricing details, see [Azure Machine Learning pricing](https://azure.microsoft.com/pricing/details/machine-learning/).

## How batch deployment works with low priority VMs

Azure Machine Learning Batch Deployments provides several capabilities that make it easy to consume and benefit from low priority VMs:

- Batch deployment jobs consume low priority VMs by running on Azure Machine Learning compute clusters created with low priority VMs. Once a deployment is associated with a low priority VMs' cluster, all the jobs produced by such deployment will use low priority VMs. Per-job configuration is not possible.
- Batch deployment jobs automatically seek the target number of VMs in the available compute cluster based on the number of tasks to submit. If VMs are preempted or unavailable, batch deployment jobs attempt to replace the lost capacity by queuing the failed tasks to the cluster.
- When a job is interrupted, it is resubmitted to run again. Rescheduling is done at the mini batch level, regardless of the progress. No checkpointing capability is provided.
- Low priority VMs have a separate vCPU quota that differs from the one for dedicated VMs. Low-priority cores per region have a default limit of 100 to 3,000, depending on your subscription offer type. The number of low-priority cores per subscription can be increased and is a single value across VM families. See [Azure Machine Learning compute quotas](how-to-manage-quotas.md#azure-machine-learning-compute).

## Considerations and use cases

Many batch workloads are a good fit for low priority VMs. Although this may introduce further execution delays when deallocation of VMs occurs, the potential drops in capacity can be tolerated at expenses of running with a lower cost if there is flexibility in the time jobs have to complete. 

Since batch endpoints distribute the work at the mini-batch level, deallocation only impacts those mini-batches that are currently being processed and not finished on the affected node. 

## Creating batch deployments with low priority VMs

Batch deployment jobs consume low priority VMs by running on Azure Machine Learning compute clusters created with low priority VMs. 

> [!NOTE] 
> Once a deployment is associated with a low priority VMs' cluster, all the jobs produced by such deployment will use low priority VMs. Per-job configuration is not possible.

You can create a low priority Azure Machine Learning compute cluster as follows:

   # [Azure CLI](#tab/cli)
   
   Create a compute definition `YAML` like the following one:
   
   __low-pri-cluster.yml__
   ```yaml
   $schema: https://azuremlschemas.azureedge.net/latest/amlCompute.schema.json 
   name: low-pri-cluster
   type: amlcompute
   size: STANDARD_DS3_v2
   min_instances: 0
   max_instances: 2
   idle_time_before_scale_down: 120
   tier: low_priority
   ```
   
   Create the compute using the following command:
   
   ```azurecli
   az ml compute create -f low-pri-cluster.yml
   ```
   
   # [Python](#tab/sdk)
   
   To create a new compute cluster with low priority VMs where to create the deployment, use the following script:
