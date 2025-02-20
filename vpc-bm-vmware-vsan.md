---
subcollection: solution-tutorials
copyright:
  years: 2024
lastupdated: "2024-01-05"
lasttested: ""

content-type: tutorial
services: vmwaresolutions, vpc
account-plan: paid
completion-time: 1h
use-case: ApplicationModernization, Vmware
---
{{site.data.keyword.attribute-definition-list}}

# Provision vSAN storage cluster
{: #vpc-bm-vmware-vsan}
{: toc-content-type="tutorial"}
{: toc-services="vmwaresolutions, vpc"}
{: toc-completion-time="1h"}

This tutorial may incur costs. Use the [Cost Estimator](/estimator) to generate a cost estimate based on your projected usage.
{: tip}



This tutorial is part of [series](/docs/solution-tutorials?topic=solution-tutorials-vpc-bm-vmware#vpc-bm-vmware-objectives), and requires that you have completed the related tutorials in the presented order.
{: important}

You need to have a minimum of three {{site.data.keyword.bm_is_short}} with local SSDs. Make sure you provisioned your {{site.data.keyword.bm_is_short}} with a compatible [profile](/docs/vpc?topic=vpc-bare-metal-servers-profile#bare-metal-servers-profile-list).  
{: important}

In this tutorial, a vSAN cluster is created using the local disks attached the 	{{site.data.keyword.bm_is_short}}. This phase is optional, if you use NFS.
{: shortdesc}

## Objectives
{: #vpc-bm-vmware-vsan-objectives}

In this tutorial, you will create a vSAN cluster using the local disks attached the {{site.data.keyword.bm_is_short}}.

![vSAN as a Datastore](images/solution63-ryo-vmware-on-vpc/Self-Managed-Simple-20210813v1-VPC-vsan.svg "vSAN as a Datastore"){: caption="Figure 1. vSAN as a Datastore" caption-side="bottom"}


## Before you begin
{: #vpc-bm-vmware-vsan-prereqs}

This tutorial requires:

* Common [prereqs](/docs/solution-tutorials?topic=solution-tutorials-vpc-bm-vmware#vpc-bm-vmware-prereqs) for VMware Deployment tutorials in {{site.data.keyword.vpc_short}}

This tutorial is part of series, and requires that you have completed the related tutorials. Make sure you have successfully completed the required previous steps:

* [Provision a {{site.data.keyword.vpc_short}} for VMware deployment](/docs/solution-tutorials?topic=solution-tutorials-vpc-bm-vmware-vpc#vpc-bm-vmware-vpc)
* [Provision {{site.data.keyword.dns_full_notm}} for VMware deployment](/docs/solution-tutorials?topic=solution-tutorials-vpc-bm-vmware-dns#vpc-bm-vmware-dns)
* [Provision bare metal servers for VMware deployment](/docs/solution-tutorials?topic=solution-tutorials-vpc-bm-vmware-bms#vpc-bm-vmware-bms)
* [Provision vCenter Appliance](/docs/solution-tutorials?topic=solution-tutorials-vpc-bm-vmware-vcenter#vpc-bm-vmware-vcenter)

[Login](/docs/cli?topic=cli-getting-started) with IBM Cloud CLI with username and password, or use the API key. Select your target region and your preferred resource group.

When advised to use Web browser, use the Jump machine provisioned in the [{{site.data.keyword.vpc_short}} provisioning tutorial](/docs/solution-tutorials?topic=solution-tutorials-vpc-bm-vmware-vpc#vpc-bm-vmware-vpc). This Jump machine has network access to the hosts, the private DNS service and vCenter IP to be provisioned. Use url with FQDN, e.g. `https://vcenter.vmware.ibmcloud.local` as used in this example.
{: note}


## Create VLAN NICs for vSAN
{: #vpc-bm-vmware-vsan-vlannic}
{: step}

1. If you have not already done so, provision VLAN interfaces for your {{site.data.keyword.bm_is_short}} for vSAN VMKs.

See instructions in [provisioning {{site.data.keyword.bm_is_short}} for VMware deployment](/docs/solution-tutorials?topic=solution-tutorials-vpc-bm-vmware-bms#vpc-bm-vmware-bms-vlannic).

If you provisioned the vSAN VLAN NICs following the guidance above, you can recall the vSAN IPs with the following commands:
{: tip}

   ```sh
   echo "vSAN IP for BMS001 : "$VMWARE_BMS001_VSAN_IP
   ```
   {: codeblock}

   ```sh
   echo "vSAN IP for BMS002 : "$VMWARE_BMS002_VSAN_IP
   ```
   {: codeblock}

   ```sh
   echo "vSAN IP for BMS003 : "$VMWARE_BMS003_VSAN_IP
   ```
   {: codeblock}


## Configure a vSAN interface using vSphere Client
{: #vpc-bm-vmware-vsan-vmk}
{: step}

Nex, you need to configure a vSAN interface for each host:

1. Log into the vCenter Server using vSphere Client via Web Browser on the Jump machine.
2. Click to select the **host**.
3. Click the **Configuration** tab.
4. Click **Networking** under Hardware.
5. Click **Add Networking**.
6. Select **VMkernel** and click **Next**.
7. Select the existing vSwitch `vds-vpc` and click **Next**.
8. Enter a name in the Network Label to identify the network that VSAN uses.
9. Select a **VLAN ID** from the VLAN ID `300`.
10. Select Use this port group for VSAN, inherit or set the vSwitch MTU (9000) and click **Next**.
11. Enter the IP address and Subnet Mask of the host's VSAN Interface. Use the VLAN interface's IP addresses collected in during the VLAN interface provisioning.
12. Click **Next**, then click **Finish**.

Repeat this for each host.


## Create vSAN using vSphere Client
{: #vpc-bm-vmware-vsan-create}
{: step}

Next, create a vSAN cluster with two disks for Cache Tier, Select remaining disks for Capacity Tier:

1. Log into the vCenter Server using vSphere Client via Web Browser on the Jump machine.
2. Click on the **cluster**
3. Click **Configure** Tab
4. Click **VSAN**, **Services**
5. Click **Configure VSAN**
6. Single Site Cluster, Click **Next**
7. On Services, click **Next**
8. On Claim disks, select `two disks` for Cache Tier, select remaining disks for Capacity Tier for each host
9. Click **Finish**

You can order vSAN licenses through [IBM Cloud Classic portal](/classic/devices/vmwarelicenses).
{: note}

## Migrate the vCenter to vSAN
{: #vpc-bm-vmware-vsan-migratevcenter}
{: step}

If vSAN is your primary shared storage, migrate vCenter into your vSAN cluster. To migrate vCenter storage to vSAN:

1. Log into the vCenter Server using vSphere Client via Web Browser on the Jump machine.
2. Click to select the **vCenter Virtual Machine**.
3. Right **Click**, and select **migrate**.
4. Click Change storage only, click **Next**.
5. Select `vsanDatastore`, click **Next**.
6. Click **Next**, then click **Finish**.

## Next steps
{: #vpc-bm-vmware-vsan-next-steps}

The next step in the tutorial series is:

* OPTIONAL: [Provision NFS storage and attach to cluster](/docs/solution-tutorials?topic=solution-tutorials-vpc-bm-vmware-nfs#vpc-bm-vmware-nfs)
* [Provision {{site.data.keyword.vpc_short}} Subnets and configure Distributed Virtual Switch Portgroups for VMs](/docs/solution-tutorials?topic=solution-tutorials-vpc-bm-vmware-newvm#vpc-bm-vmware-newvm)
* [Provision {{site.data.keyword.vpc_short}} Public Gateways and Floating IPs for VMware Virtual Machines](/docs/solution-tutorials?topic=solution-tutorials-vpc-bm-vmware-pgwip#vpc-bm-vmware-pgwip)
