---
title: Use the autoscaler profile to configure cluster autoscaling in Azure Kubernetes Service on Azure Stack HCI and Windows Server
description: Learn how to use the autoscaler profile to configure Cluster autoscaler in Azure Kubernetes Service on  Azure Stack HCI and Windows Server
ms.topic: how-to
author: mattbriggs
ms.author: mabrigg 
ms.lastreviewed: 03/16/2022
ms.reviewer: mikek
ms.date: 03/16/2022

# Intent: As a Kubernetes user, I want to use cluster autoscaler to grow my nodes to keep up with application demand.
# Keyword: configure cluster autoscaling

---

# Use the autoscaler profile to configure cluster autoscaling in Azure Kubernetes Service on Azure Stack HCI and Windows Server

You can use the parameters in the autoscaler profile object to define scale events. The cluster autoscaler profile affects all node pools that use the cluster autoscaler. You can't set an autoscaler profile per node pool.

The profiles have the following attributes:

- Autoscaler profiles will apply to all node pools in a cluster.
- Profiles are global deployment level objects.
- There can be multiple profiles available in an AKS on Azure Stack HCI and Windows Server deployment.
- Only one profile is assigned to a given cluster. The profile is used for all node pools in the cluster.
- Changes to the profile are applied to all node pools, which have the autoscaler function enabled.

> [!IMPORTANT]
> Cluster autoscaler is currently in PREVIEW.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.
 
## Profile settings

The default profile consists of the below default values. You can update  the following settings:

| Setting | Description | Default value |
| --- | --- | --- |
| min-node-Count | The minimum node count the node pool this profile is assigned can scale down to. | 0 |
| max-node-Count | The maximum node count the node pool this profile is assigned to can scale up to. | 1 |
| scan-interval | How often cluster is reevaluated for scale up or down. | 10 seconds |
| scale-downdelay-afteradd | How long after scale up that scale down evaluation resumes. | 10 minutes |
| scale-downdelay-afterdelete | How long after node deletion that scale down evaluation resumes. | scaninterval |
| scale-downdelay-afterfailure | How long after scale down failure that scale down evaluation resumes. | 3 minutes |
| scale-downunneededtime | How long a node should be unneeded before it's eligible for scale down. | 10 minutes |
| scale-downunreadytime | How long an unready node should be unneeded before it's eligible for scale down. | 20 minutes |
| scale-downutilizationthreshold | Node utilization level, defined as sum of requested resources divided by capacity, below which a node can be considered for scale down. | 0.5 |
| max-gracefulterminationsec | Maximum number of seconds the cluster autoscaler waits for pod termination when trying to scale down a node. | 600 seconds |
| balancesimilarnodegroups | Detects similar node pools and balances the number of nodes between them. | false |
| expander | Type of node pool expander to be used in scale up. Possible values: most-pods, random, least-waste, priority. | random |
| skip-nodeswith-localstorage | If true cluster autoscaler will never delete nodes with pods with local storage, for example, EmptyDir or HostPath. | true |
| skip-nodeswithsystempods | If true cluster autoscaler will never delete nodes with pods from kube-system (except for DaemonSet or mirror pods). | true |
| max-emptybulk-delete | Maximum number of empty nodes that can be deleted at the same time. | 10 nodes |
| new-podscale-updelay | For scenarios like burst/batch scale where you don't want CA to act before the kubernetes scheduler could schedule all the pods, you can tell CA to ignore unscheduled pods before they're a certain age. | 0 seconds |
| max-totalunreadypercentage | Maximum percentage of unready nodes in the cluster. After this percentage is exceeded, CA halts operations. | 45% |
| max-nodeprovisiontime | Maximum time the autoscaler waits for a node to be provisioned. | 15 minutes |

## Notes on configuration the autoscaler

You can change settings in the cluster autoscaler profile using the cmdlet [Set-AksHciAutoScalerConfig](work-with-horizontal-autoscaler.md#change-an-existing-akshciautoscalerconfig-profile-object).

The cluster autoscaler makes scaling decisions based on the minimum and maximum counts set on each node pool, but it doesn't enforce them after updating the min or max counts. For example, setting a min count of five when the current node count is three won't immediately scale the pool up to five. 

If the minimum count on the node pool has a value higher than the current number of nodes, the new min or max settings will be respected when there are enough unschedulable pods present that would require two new additional nodes and trigger an autoscaler event. After the scale event, the new count limits are respected. 

You can also configure more granular details of the cluster autoscaler by changing the default values in the cluster-wide autoscaler profile. For example, a scale down event happens after nodes are under-utilized after 10 minutes. If you had workloads that ran every 15 minutes, you may want to change the autoscaler profile to scale down under utilized nodes after 15 or 20 minutes. When you enable the cluster autoscaler, a default profile is used unless you specify different settings.

## Save and load the autoscaler profile

You can save and store your autoscaler profile in a profile definition as a `yaml` file. You can manually edit the yaml file from a text editor. And you can load saved definitions.
### Save your profile definition

You save a copy of the profile using `kvactl` as a `yaml` file. After you've defined your profile, run the following commands:

```powershell
kvactl.exe autoscalerprofile get --name default --kubeconfig (Get-AksHciConfig).Kva.kubeconfig --outputformat=yaml > def.yaml
```
### Edit your profile definition

You can edit the profile definition in the `yaml` file. For example, you can open `def.yaml` in notepad, Visual Studio Code, or other text editors.
### Load your profile definition

You can load the profile definition using `kvactl` from the saved `yaml` file. Run the following commands:

```powershell
kvactl.exe autoscalerprofile create --profileconfig .\def-new.yaml --kubeconfig (Get-AksHciConfig).Kva.kubeconfig
```
## Next steps
- [Use PowerShell for cluster autoscaling](work-with-horizontal-autoscaler.md)
- [Learn about cluster autoscaling](concepts-cluster-autoscaling.md)
