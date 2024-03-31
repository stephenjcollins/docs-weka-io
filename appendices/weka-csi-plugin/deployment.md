# Deployment

You can deploy the WEKA CSI Plugin using the helm chart from the official [WEKA ArtifactHub repository](https://artifacthub.io/packages/helm/csi-wekafs/csi-wekafsplugin).

## Prerequisites

Ensure the following prerequisites are met:

* The privileged mode must be allowed on the Kubernetes cluster.
* The following Kubernetes feature gates must be enabled: DevicePlugins, CSINodeInfo, CSIDriverRegistry, and ExpandCSIVolumes (all these gates are enabled by default).
* A WEKA cluster is installed.
* For snapshot and directory backing, filesystems must be pre-defined on the WEKA cluster.
* Your workstation has a valid connection to the Kubernetes worker nodes.
* The WEKA client is installed on the Kubernetes worker nodes and the control plane. Adhere to the following:
  * A WEKA client part of the cluster (stateful client) is recommended rather than a stateless client. See [Add clients](../../planning-and-installation/bare-metal/adding-clients-bare-metal.md).
  * If the Kubernetes worker nodes run on the WEKA cluster backends (converged mode), ensure the WEKA processes are up before the `kubelet` process.

## Installation

1. On your workstation, add the `csi-wekafs` repository:

```
helm repo add csi-wekafs https://weka.github.io/csi-wekafs

```

2. Install the WEKA CSI Plugin. Run the following command:

```
helm install csi-wekafs csi-wekafs/csi-wekafsplugin --namespace csi-wekafs --create-namespace [--set selinuxSupport=<off | mixed | enforced>]

```

<details>

<summary>Installation output example</summary>

Once the installation completes successfully, the following output is displayed:

```
NAME: csi-wekafs
LAST DEPLOYED: Mon May 29 08:36:19 2023
NAMESPACE: csi-wekafs
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing csi-wekafs.

Your release is named csi-wekafs.
The release is installed in namespace csi-wekafs

To learn more about the release, try:

  $ helm status -n csi-wekafs csi-wekafs
  $ helm get all -n csi-wekafs csi-wekafs

Official Weka CSI Plugin documentation can be found here: https://docs.weka.io/appendix/weka-csi-plugin

Examples of how to configure a storage class and start using the driver are here:
https://github.com/weka/csi-wekafs/tree/master/examples

-------------------------------------------------- NOTICE --------------------------------------------------
| THIS VERSION INTRODUCES SUPPORT FOR ADDITIONAL VOLUME TYPES, AS WELL AS SNAPSHOT AND VOLUME CLONING CAPS |
| TO BETTER UNDERSTAND DIFFERENT TYPES OF VOLUMES AND THEIR IMPLICATIONS, REFER TO THE DOCUMENTATION ABOVE |
| ALSO, IT IS RECOMMENDED TO CAREFULLY GO OVER NEW CONFIGURATION PARAMETERS AND ITS MEANINGS, AS BEHAVIOR  |
| OF THE PLUGIN AND ITS REPORTED CAPABILITIES LARGELY DEPEND ON THE CONFIGURATION AND WEKA CLUSTER VERSION |
------------------------------------------------------------------------------------------------------------

-------------------------------------------------- WARNING -------------------------------------------------
|  SUPPORT OF LEGACY VOLUMES WITHOUT API BINDING WILL BE REMOVED IN NEXT MAJOR RELEASE OF WEKA CSI PLUGIN. |
|  NEW FEATURES RELY ON API CONNECTIVITY TO WEKA CLUSTER AND WILL NOT BE SUPPORTED ON API-UNBOUND VOLUMES. |
|  PLEASE MAKE SURE TO MIGRATE ALL EXISTING VOLUMES TO API-BASED SCHEME PRIOR TO NEXT VERSION UPGRADE.     |
------------------------------------------------------------------------------------------------------------

```

</details>

## Upgrade from any previous version to WEKA CSI Plugin v2.0&#x20;

In WEKA CSI Plugin v2.0, the CSIDriver object has undergone changes. Specifically, CSIDriver objects are now immutable. Consequently, the upgrade process involves uninstalling the previous CSI version using Helm and subsequently installing the new version. It is important to note that the uninstall operation does not delete any existing secrets, StorageClasses, or PVC configurations.

{% hint style="danger" %}
If you plan to upgrade the existing WEKA CSI Plugin and enable directory quota enforcement for already existing volumes, bind the legacy volumes to a single secret. See the [Bind legacy volumes to API](upgrade-legacy-persistent-volumes-for-capacity-enforcement.md#bind-legacy-volumes-to-api) section.
{% endhint %}

#### 1. Prepare for the upgrade

1. Uninstall the existing CSI Plugin. Run the following command line:

```
helm uninstall csi-wekafs --namespace csi-wekafs
```

2. Update the `csi-wekafs` helm repository. Run the following command line:

```
helm repo update csi-wekafs

```

3. Download the `csi-wekafs` git repository.

```
git clone https://github.com/weka/csi-wekafs.git --branch main --single-branch
```

#### 2. Install the upgraded helm release

Run the following command line:

```
helm install csi-wekafs --namespace csi-wekafs csi-wekafs/csi-wekafsplugin

```

## Upgrade from WEKA CSI Plugin WEKA 2.0 forward

If the WEKA CSI plugin source is v2.0 or higher, follow this workflow.

#### 1. Update helm repo

Run the following command line:

```
helm repo update csi-wekafs
```

#### 2. Update the helm deployment

Run the following command line:

```
helm upgrade --install csi-wekafs --namespace csi-wekafs csi-wekafs/csi-wekafsplugin
```

<details>

<summary>Output example</summary>

Once the upgrade completes successfully, the following output is displayed:

```
Release "csi-wekafs" has been upgraded. Happy Helming!
NAME: csi-wekafs
LAST DEPLOYED: Tue June  2 15:39:01 2023
NAMESPACE: csi-wekafs
STATUS: deployed
REVISION: 10
TEST SUITE: None
NOTES:
Thank you for installing csi-wekafsplugin.

Your release is named csi-wekafs.

To learn more about the release, try:

  $ helm status csi-wekafs
  $ helm get all csi-wekafs

Official Weka CSI Plugin documentation can be found here: https://docs.weka.io/appendix/weka-csi-plugin

```

</details>

#### 3. Elevate the OpenShift privileges

If the Kubernetes worker nodes run on RHEL and use OpenShift, elevate the OpenShift privileges for the WEKA CSI Plugin. (RHCoreOS on Kubernetes worker nodes is not supported.)

To elevate the OpenShift privileges, run the following command lines:

```
oc create namespace csi-wekafs
oc adm policy add-scc-to-user privileged system:serviceaccount:csi-wekafs:csi-wekafs-node
oc adm policy add-scc-to-user privileged system:serviceaccount:csi-wekafs:csi-wekafs-controller

```

#### 4. Delete the CSI Plugin pods

The CSI Plugin fetches the WEKA filesystem cluster capabilities during the first login to the API endpoint and caches it throughout the login refresh token validity period to improve the efficiency and performance of the plugin.

However, the WEKA filesystem cluster upgrade might be unnoticed if performed during this time window, continuing to provision new volumes in the legacy mode.

To expedite the update of the Weka cluster capabilities, it is recommended to delete all the CSI Plugin pods to invalidate the cache. The pods are then restarted.

Run the following command lines:

```
kubectl delete pod -n csi-wekafs -lapp=csi-wekafs-controller
kubectl delete pod -n csi-wekafs -lapp=csi-wekafs-node
```