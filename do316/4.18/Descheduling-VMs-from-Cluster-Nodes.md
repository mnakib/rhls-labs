
## Descheduling Workloads

The built-in OpenShift scheduler handles the initial placement of a workload, by determining the best node for the workload to run on. However, the scheduler does not proactively rebalance workloads across the cluster.

The descheduler complements the scheduler by proactively rebalancing workloads across the cluster.

## The Kube Descheduler Operator

The primary function of the Kube Descheduler operator is to evict pods from nodes so OpenShift can reschedule them onto more appropriate nodes. If the evicted pod is the `virt-launcher` pod for a VM, then this action triggers a live migration of the VM to another node. This function is similar to the __*vSphere Distributed Resource Scheduler (DRS)*__.

## Descheduler Profiles

OpenShift descheduler profiles are a set of preconfigured rules used by the Kubernetes Descheduler (managed via the Kube Descheduler Operator) to periodically evaluate and rebalance workloads running in your cluster.

A __profile__ is a set of strategies. A __strategy__ represents a single operational task designed to solve a specific type of cluster imbalance. 

Depending on your cluster version and whether you are running standard containers or Virtual Machines (via OpenShift Virtualization), you can configure one of the following profiles:

`LongLifecycle`

The `LongLifecycle` profile helps to balance resources across nodes. This profile identifies underutilized and overutilized nodes. Then, the descheduler evicts workloads from the overutilized nodes to be rescheduled on the underutilized nodes.

This profile enables the following strategies:

- `LowNodeUtilization`

- `RemovePodsHavingTooManyRestarts`
 
- `PodAffinity`

`DevKubeVirtRelieveAndMigrate`

An enhanced version of the `LongLifecycle` profile that is designed for KubeVirt workloads. This profile reduces overall expenses by evicting pods from high-cost nodes, and considers workload migration.

## Descheduler Strategies

Descheduler profiles are composed of one or more strategies that determine which workloads to evict. The profiles that are compatible with OpenShift Virtualization use the following strategies:

- `RemoveDuplicates`: Evicts redundant tracker or worker pods belonging to the same controller if they land on the same node.

- `LowNodeUtilization`: Finds underutilized nodes and evicts pods from overutilized ones to spread the load.

- `RemovePodsViolatingNodeTaints`: Kills pods that can no longer tolerate a node's updated taints.


## Enabling KubeDescheduler Profiles

You configure these profiles inside the KubeDescheduler Custom Resource (CR) using the OpenShift CLI (oc) or the web console

By default, the operator runs in Predictive (simulation) mode, where it logs potential evictions without actually moving any pods. To let the scheduler actively migrate workloads, you must change the `mode` to __Automatic__

```yaml
apiVersion: operator.openshift.io/v1
kind: KubeDescheduler
metadata:
  name: cluster
  namespace: openshift-kube-descheduler-operator
spec:
  deschedulingIntervalSeconds: 3600
  mode: Automatic
  profiles:
    - AffinityAndTaints

```

## Enabling Descheduler Evictions on VMs

In OpenShift Virtualization, you can enable descheduler evictions on a VM by adding an annotation to the VM manifest. The `descheduler.alpha.kubernetes.io/evict` annotation allows the descheduler to evict the VM's pod.

<pre>
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: <VMName>
  namespace: <ProjectName>
spec:
  template:
    metadata:
      annotations:
        <b>descheduler.alpha.kubernetes.io/evict: "true"</b>
  ...
</pre>

You can also add the annotation to the VM from the GUI in __Configuration > Scheduling > Descheduler__.


## Install and Configure the Kube Descheduler Operator

To use the descheduler, you must install the Kube Descheduler Operator. You can install the operator from the OperatorHub in the OpenShift Container Platform web console. Go to __Operators__ → __Operator Hub__ in the left panel, and search for __descheduler__. Click the __Kube Descheduler Operator__ operator.



## Demo

- Install the KubeDescheduler Operator
- Create the KubeDescheduler Instance
- Create a VM where descheduler evictions are enabled.
- Create a Node Affinity rule for the VM



#### Install the KubeDescheduler Operator

From the __OpenShift Container Platform web__ console, go to __OperatorHub → Operators → Operator Hub__ in the left panel, and search for __descheduler__. Click the __Kube Descheduler Operator__ operator.

#### Create the KubeDescheduler Instance

Create a KubeDescheduler Instance with below values.

| Name | Parameter | Value |
| :--- | :--- | :--- |
| **cluster mode** | `mode` | `Predictive` |
| **deschedulingIntervalSeconds** | `deschedulingIntervalSeconds` | `60` |
| **profileCustomizations > devLowNodeUtilizationThresholds** | `profileCustomizations.devLowNodeUtilizationThresholds` | `High` |
| **profileCustomizations > devEnableEvictionsInBackground** | `profileCustomizations.devEnableEvictionsInBackground` | `Checked` |
| **profiles > Value** | `profiles` | `LongLifecycle` |


#### Create a VM where descheduler evictions are enabled.

Create a VM named descheduler-vm in the descheduler-demo namespace

```bash
# Create a cloud-config file for storing the VM credentials
cat > cloud-config.yaml
```

```yaml
#cloud-config
user: admin
password: redhat
chpasswd: { expire: False }
ssh_pwauth: True
```

```bash
# Encode the cloud-config file
base64 -w0 cloud-config.yaml > cloud-config.b64
```

```bash
# Create and label the db project
oc new-project descheduler-demo
```
```bash
# Create a VM named descheduler-vm
virtctl create vm \
  --name=descheduler-vm \
  --instancetype=u1.small \
  --preference=rhel.10 \
  --volume-datasource=src:openshift-virtualization-os-images/rhel10 \
  --namespace descheduler-demo \
  --cloud-init-user-data "$(cat cloud-config.b64)" > descheduler-vm.yaml
```

Enable the descheduler evictions 

```bash
oc patch vmi descheduler-vm -n descheduler-demo --type='merge' -p '{"metadata":{"annotations":{"descheduler.alpha.kubernetes.io/evict":"true"}}}'
```


#### Create a Node Affinity rule for the VM

Create a Node Affinity rule for the VM with below values

| Parameter | Value |
| :--- | :--- |
| **Type** | Node Affinity |
| **Condition** | Preferred during scheduling |
| **Weight** | 100 |
| **Key** | kubernetes.io/hostname In master01
