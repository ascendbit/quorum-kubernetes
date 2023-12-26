# Huawei Cloud

## Background

This guide will help you run Hyperledger Besu or GoQuorum clients in Huawei Cloud Container Engine (CCE) for both development and production scenarios. It's recommended to familiarize yourself with CCE (or equivalent Kubernetes infrastructure) before running things in production on Kubernetes.

The setup comprises base infrastructure that is used to build the cluster & other resources in Huawei Cloud. We also make use of some Huawei Cloud native services and features after the cluster is created. These include:

- [Workload identities](https://support.huaweicloud.com/intl/en-us/bestpractice-cce/cce_bestpractice_0333.html).
- [CCE CSI drivers](https://github.com/huaweicloud/huaweicloud-csi-driver)
- Data is stored using dynamic StorageClasses backed by Huawei Cloud EVS. Note that the [Volume Claims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) are fixed sizes and can be updated as you grow via a helm update, and will not need reprovisioning of the underlying storage class.
- [CNI](https://support.huaweicloud.com/intl/en-us/basics-cce/kubernetes_0023.html) networking mode for CCE.

### Operation flow:

1. Read this file in its entirety before proceeding
2. See the [Prerequisites](#prerequisites) section to enable some features before doing the deployment
3. See the [Usage](#usage) section

#### Helm Charts:

The dev charts are aimed at getting you up and running so you can experiment with the client and functionality of the tools, contracts etc. They embed node keys etc as secrets so that these are visible to you during development and you can learn about discovery. The prod charts utilize all the built in Huawei Cloud functionality and recommended best practices such as identities, secrets stored in keyvault with limited access etc. **When using the prod charts please ensure you add the necessary values to the `hwc` section of the values.yml file**

#### Warning:

1. Please do not create more than one CCE cluster in the same subnet.
2. CCE clusters may **not** use _169.254.0.0/16, 172.30.0.0/16, 172.31.0.0/16, or 192.0.2.0/24_ for the Kubernetes service address range.

## Pre-requisites:

You will need to install [KooCLI](https://support.huaweicloud.com/intl/en-us/qs-hcli/hcli_02_003.html)

## Usage

1. Update the [cluster.yml](./templates/cluster.yml) with your VPC details
2. Deploy the template

```bash
hcloud CCE CreateCluster -f ./templates/cluster.yml
```