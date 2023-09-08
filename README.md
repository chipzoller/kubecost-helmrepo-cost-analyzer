## Kubecost

Kubecost gives teams visibility into current and historical Kubernetes spend and resource allocation. These models provide cost transparency in Kubernetes environments that support multiple applications, teams, departments, etc.


To see more on the functionality of the full Kubecost product, please visit the [features page](https://kubecost.com/#features) on our website. 

Some of the features of Kubecost include:

- Real-time cost allocation by Kubernetes Service, Deployment, Namespace, label, StatefulSet, DaemonSet, Pod, and container
- Dynamic asset pricing enabled by integrations with AWS, Azure, and GCP billing APIs 
- Supports on-premises Kubernetes clusters with custom pricing sheets
- Allocation for in-cluster resources like CPU, GPU, memory, and persistent volumes
- Allocation for AWS and GCP out-of-cluster resources like RDS instances and S3 buckets with key (optional)
- Easily export pricing data to Prometheus with /metrics endpoint ([learn more](https://docs.kubecost.com/install-and-configure/install/custom-prom))
- Free and open source distribution (Apache2 license)

## Requirements

- Kubernetes version 1.8 or higher
- kube-state-metrics
- Node exporter
- Prometheus

## Getting Started

You can deploy Kubecost on any Kubernetes 1.8+ cluster in a matter of minutes, if not seconds.  Visit the Kubecost docs for [recommended install options](https://docs.kubecost.com/install-and-configure/install). Compared to building from source, installing from Helm is faster and includes all necessary dependencies. 

## Usage

* [User interface](https://docs.kubecost.com/using-kubecost/navigating-the-kubecost-ui)
* [Cost APIs](https://docs.kubecost.com/apis/apis-overview)
* [CLI / kubectl cost](https://github.com/kubecost/kubectl-cost)
* [Prometheus metric exporter](https://docs.kubecost.com/install-and-configure/install/custom-prom)

## Contributing

We :heart: pull requests! See the OpenCost [contribution guide](https://github.com/opencost/opencost/blob/develop/CONTRIBUTING.md) for information on building the project from source and contributing changes. 

## Licensing

Licensed under the Apache License, Version 2.0 (the "License")

## Frequently Asked Questions

### How do you measure the cost of CPU/RAM/GPU/storage for a container, pod, deployment, etc.

The Kubecost model collects pricing data from major cloud providers, e.g. GCP, Azure, and AWS, to provide the real-time cost of running workloads. Based on data from these APIs, each container/pod inherits a cost per CPU-hour, GPU-hour, Storage GB-hour and cost per RAM GB-hour based on the node where it was running or the class of storage provisioned. This means containers of the same size, as measured by the max of requests or usage, could be charged different resource rates if they are scheduled in separate regions, on nodes with different usage types (on-demand vs preemptible), etc. 

For on-premises clusters, these resource prices can be configured directly with custom pricing sheets (more below).

Measuring the CPU/Memory/GPU cost of a deployment, service, namespace, etc is the aggregation of its individual container costs.

### How do you determine RAM/CPU costs for a node when this data isn’t provided by a cloud provider?

When explicit RAM or CPU prices are not provided by your cloud provider, the Kubecost model falls back to the ratio of base CPU and RAM price inputs supplied. The default values for these parameters are based on the marginal resource rates of the cloud provider, but they can be customized within Kubecost.

These base Memory/CPU prices are normalized to ensure the sum of each component is equal to the total price of the node provisioned, based on billing rates from your provider. When the sum of Memory/CPU costs is greater (or less) than the price of the node, then the ratio between the two input prices are held constant.  

As an example, let's imagine a node with 1 CPU and 1 GB of RAM that costs $20/month. If your base CPU price is $30 and your RAM per-GB price is $10, then these inputs will be normalized to $15 for CPU and $5 for RAM so that the sum equals the cost of the node. Note that the price of a CPU remains 3x the price of a GB of RAM. 

```
NodeHourlyCost = (NORMALIZED_CPU_PRICE * # of CPUs) + (NORMALIZED_RAM_PRICE * # of RAM GB)
```

### How do you allocate a specific amount of RAM/CPU to an individual pod or container?

Resources are allocated based on the time-weighted maximum of resource requests and usage over the measured period. For example, a pod with no usage and 1 CPU requested for 12 hours out of a 24 hour window would be allocated 12 CPU hours. For pods with [BestEffort quality of service](https://kubernetes.io/docs/concepts/workloads/pods/pod-qos/#besteffort) (i.e. no requests) allocation is done solely on resource usage. 

### How do I set my AWS Spot bids for accurate allocation?

Modify `spotCPU` and `spotRAM` in the [values file](https://github.com/kubecost/cost-analyzer-helm-chart/blob/develop/cost-analyzer/values.yaml) to the price of your bid. Allocation will use these bid prices, but it does not take into account what you are actually charged by AWS. Alternatively, you can provide an AWS key to allow access to the Spot data feed. This will provide accurate Spot prices. See the documentation [here](https://docs.kubecost.com/install-and-configure/install/cloud-integration/aws-cloud-integrations/aws-spot-instances) for further details.

### Do I need a GCP billing API key?

We supply a global key with a low limit for evaluation, but you will want to supply your own before moving to production.  
  
Please reach out with any additional questions [here](https://docs.kubecost.com/#staying-in-the-loop) or via email at [team@kubecost.com](mailto:team@kubecost.com). 
