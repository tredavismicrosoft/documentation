---
title: Production Ready Deployment of Azure VMware Solution
description: This article outlines the workflow of an Azure VMware Solution (AVS) deployment.  The final result will be an environment ready for VM creation and migration.
ms.topic: conceptual
ms.date: 08/04/2020
ms.author: tredavis 
---
# Azure VMware Solution (AVS) Deployment
This article outlines the workflow of an Azure VMware Solution (AVS) deployment. The final result will be a production ready environment.

## Planning
In this section you will be collecting data to use in subsequent steps.  

If you would like you can [download this file](https://github.com/tredavismicrosoft/documentation/raw/master/avs-pre-deployment-checklist.docx) so you can document all the information collected in this planning phase.  
 
### Identify - IP Address Segment for AVS Platform

The first step in deploying AVS will be to plan out the IP segmentation.  

During deployment AVS injests a /22 network you provide, and then carves it up into smaller segments and use those segments for IPing vCenter, HCX, NSX, vMotion network, etc...

AVS connects to your Azure vNet via an internal Express Route and (in most cases) will ultimately connect to your datacenter via Global Reach (Express Route and Global Reach will be discussed later in detail). 

What you need to be aware of is that AVS, your existing Azure environment and your on-premesis enviornment will all exchange routes (typically). That being the case you will need to ID a /22 network to input during the AVS deployment which does not overlap anything you already have on-premesis or in Azure.

**Example:** 10.0.0.0/22

[Please see this link for details](https://docs.microsoft.com/en-us/azure/azure-vmware/tutorial-network-checklist#network-connectivity).  

![](/IPs.png)

---

### Identify - IP Address Segment for VM Workloads in AVS

Any IP segments you create in AVS need to be unique across your Azure and on-premises footprint.  

Identify a IP segment which you will use to create your first network (NSX Segment) in your AVS private cloud.

**Example:** 10.0.4.0/24

![](/nsxsegment.png) 

---

### Identify - Networks Which Will Be Extended to AVS From On-Premises (Optional)

You may choose to extend network segments from on-premises to AVS.  If you are planning on extending networks from on-premises those networks must connected to a [vSphere Distributed Switch](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.networking.doc/GUID-B15C6A13-797E-4BCB-B9D9-5CBC5A60C3A6.html) in your on-premises VMware environment.  If the network(s) you will be extending live on a [vSphere Standard Switch](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.networking.doc/GUID-350344DE-483A-42ED-B0E2-C811EE927D59.html) they cannot be extended.  

---

### Identify - Global Reach Peering Network
A /29 non-overlapping network address block for the ExpressRoute Global Reach peering.  The IPs in this segment will be used at each end of the Global Reach connection for purposes of connecting the two Express Routes.

**Example:** 10.1.0.0/29

![](/grip.png)

---

### Identify - Subscription
The subscription you plan to use for the deployment.  You can either create a new subscription or re-use an existing one.

**NOTE:** This subscription must be associated with a Microsoft Enterprise Agreement.

---

### Identify - Resource Group
Generally, a new resource group will be created for AVS and it is supporting Azure components, but this is not a requirement.

---

### Identify - Region
Identify which region you want AVS deployed, please see the [Azure Products Available By Region Guide](https://azure.microsoft.com/en-us/global-infrastructure/services/?products=azure-vmware-cloudsimple).

---

### Identify - Resource Name
This is a friendly name which you will title your AVS private cloud.

---

### Identify - Size Nodes
Which size nodes you want to build out your AVS with.  [See AVS documentation for a complete list](https://docs.microsoft.com/en-us/azure/azure-vmware/concepts-private-clouds-clusters#hosts).

---

### Identify - Number of Hosts
How many hosts will you build out the AVS cluster with.  The minimum node count is 3, maximum is 16 per cluster.  See [this link](https://docs.microsoft.com/en-us/azure/azure-vmware/concepts-private-clouds-clusters#clusters) for more details.

---

### Identify - vCenter Admin Password
During the deployment you will be prompted to define the vCenter admin password.

---

### Identify - NSX Admin Password
During the deployment you will be prompted to define the NSX admin password.

---

### Azure vNet To Attach AVS
To access your AVS private cloud the Express Route which comes with AVS will need to attach to an Azure vNet.  

During deployment you can define a new vNet or choose an existing vNet and then during the AVS deployment the Express Route from AVS will be connected to an Express Route Gateway on the vNet defined during deployment.  

It is important to note that if you choose an existing vNet you need to select one which does not have a pre-existing Gateway Subnet.  

If you want to connect the Express Route from AVS to an existing Express Route Gateway, this can be done after deployment.  

So in summary, do you want to connect AVS to an existing Express Route Gateway?  
Yes = Identify the vNet for planning purposes, but during deployment this will not be used.
No = Identify an existing vNet or create a new vNet a new vNet during deployment.

Either way, document what you want to do in this step.

**NOTE:**  This vNet will be seen by your on-premises environment and AVS so make sure whatever IP segment is used in this vNet and subnets do not overlap.

![](/avsexr.png)

## Deployment and Configuration of AVS Platform
In this section we will be taking the information collected [in the previous section](/production-ready-deployment-steps.md#planning) and using it to depoloy the Azure VMware Solution Private Cloud.

### Register the AVS Resource Provider
Follow [the steps shown in this link](https://docs.microsoft.com/en-us/azure/azure-vmware/tutorial-create-private-cloud#register-the-resource-provider) to register the resource provider for AVS with your subscription.

### Deploy AVS
Take the information you collected from the [Planning section](/production-ready-deployment-steps.md#planning) above and [deploy the AVS Private Cloud](https://docs.microsoft.com/en-us/azure/azure-vmware/tutorial-create-private-cloud#azure-portal).

**NOTE:** The person deploying AVS must be at minimum contributor level in the subscription.

### Create AVS Jumpbox

When the AVS private cloud completes deployment you will need a virtual machine in the vNet where AVS connects so you can reach vCenter and NSX.  Typically this jumpbox will not be needed once all the networking is configured (express route to/from on-premises and global reach).  But until then it is a handy tool to have so you can reach vCenter and NSX in AVS.  

Here are instructions on how to create a [virtual machine in Azure](https://docs.microsoft.com/en-us/azure/azure-vmware/tutorial-access-private-cloud#create-a-new-windows-virtual-machine).  You will want to create this virtual machine in whatever vNet you have [identified and/or created as part of the deployment process](/production-ready-deployment-steps.md#azure-vnet-to-attach-avs).

### Connect AVS to vNet with Express Route 
If you did not define a vNet in the [deployment step](/production-ready-deployment-steps.md/#deploy-avs) and your intent is to connect the AVS Express Route to an existing Express Route Gateway then follow [steps 1-4 in this section](https://docs.microsoft.com/en-us/azure/azure-vmware/tutorial-configure-networking#connect-expressroute-to-the-virtual-network-gateway), then move to the next section.

If you already defined a vNet in the [deployment step](/production-ready-deployment-steps.md/#deploy-avs) then skip to the next section.

### Verify Network Routes Being Advertised From AVS to Azure vNet
At this point you should have a jumpbox in the vNet where AVS connects via it's Express Route.  Go to that jumpbox's network interface in Azure and [view the effective routes](https://docs.microsoft.com/en-us/azure/virtual-network/manage-route-table#view-effective-routes).

You should see in the effective route list the networks which were created as part of the AVS deployment.  You will see multiple /24 networks which were derived from the [/22 network you defined](/production-ready-deployment-steps.md#ip-address-segment-for-avs-platform) input during the [deployment step](/production-ready-deployment-steps.md/#deploy-avs)

It will look something similar to this.  Notice there are a series of /24 networks, these were all derived from the 10.74.72.0/22 network being input during the deployment, in this example.  

If you see something similar to this you should be able to connect to vCenter in AVS.

![](/effectiveroutes.png)

### Connect and Log Into vCenter and NSX-T
You will need to log into the jumpbox you created in the earlier step, when you have logged into the jumpbox open a web browser and navigate to and log into both vCenter and NSX-T admin console.  

You can identify the IP addresses and credentials of both vCenter and NSX-T admin console by [following the instructions here](https://docs.microsoft.com/en-us/azure/azure-vmware/tutorial-access-private-cloud#connect-to-the-local-vcenter-of-your-private-cloud).

### Create a Network Segment on AVS
If you plan on creating new network segments in your AVS environment you will need to create those via NSX-T.  In the planning section you may have defined which networks you are going to create in AVS, you can create those now.  If you didn't already decide which networks you are going to create do that now.  

Just like on-premises you are going to want to define a CIDR block of your choice which does not overlap with anything in Azure or on-premises.  If you are not planning on connecting AVS back to your on-premises environment, still, don't overlap the networks, you never know down the road if you will connect them.

Follow [these instructions](https://docs.microsoft.com/en-us/azure/azure-vmware/tutorial-nsx-t-network-segment) to create an AVS network segment.

### Verify NSX-T Segment is Advertised
Re-do the [Verify Network Routes Being Advertised From AVS to Azure vNet](/production-ready-deployment-steps.md#verify-network-routes-being-advertised-from-avs-to-azure-vnet) step. 

But now you should see an additional route in the list, you should see the network segment(s) you just created in the previous step.

### Identify DNS Server
In the next steps you are going to be configuring either statically a virtual machine on the NSX-T segment or assigning IP addresses via DHCP to that virtual machine, either way, you are going to need to have a DNS server identified.  You can use a DNS server in your environment which is accessible, but that this point you may not have access to the DNS server in your environment because Global Reach has not been configured.  

If you have access to a DNS server you have already created somewhere in your environment, use that, if not identify the DNS server being used by your jumpbox and use that same DNS server.

### Provide DHCP Services to NSX-T Network Segment (Optional)
If you want to use DHCP on your NSX-T segment(s), follow the guidance in this step, if not, skip to the [next section](production-ready-deployment-steps.md#put-a-virtual-machine-on-the-nsx-t-network-segment).  

Now that you have created your NSX-T network segment in AVS you can either leverage NSX-T as a DHCP Server or you can relay DHCP requests from the NSX-T segment(s) to a DHCP server somewhere else in your environment.

If you want to use NSX-T as the DHCP Server for the segment(s) you just created, you will want to [create a DHCP server in NSX-T](https://docs.microsoft.com/en-us/azure/azure-vmware/manage-dhcp#create-dhcp-server) and [relay to that server](https://docs.microsoft.com/en-us/azure/azure-vmware/manage-dhcp#create-dhcp-relay-service).

If you want to relay DHCP requests to a DHCP server somewhere in your environment which you have built or already have, then [only do the relay configuration](https://docs.microsoft.com/en-us/azure/azure-vmware/manage-dhcp#create-dhcp-relay-service).

### Put a Virtual Machine On the NSX-T Network Segment
In your AVS vCenter you will want to deploy a VM, you will use this VM to verify connectivity from your AVS network(s) to the Internet, to Azure vNet(s) and later to on-premises.

Do this as you would in any vSphere environment.  Attach this VM to the/one of the networks which you just created in NSX-T.  If you setup DHCP in the previous step you will get your network configuration for that VM from that DHCP server (don't forget to setup the scope), and if you are going to statically configure, then just configure as you normally would.

### Test NSX-T Segment Connectivity 
Log into the virtual machine you created in the previous step.  Verify connectivity from your AVS network(s) to the Internet, to Azure vNet(s) and to on-premises.  To do this complete the following steps;

1. Ping an IP on the internet.
2. Go to an Internet site via web browser.
3. Ping the Jumpbox which sits on the Azure vNet.

---

**At this point AVS is up and running and you have successfully established connectivity to/from Azure vNets and the Internet.**

--- 

## Connect AVS to Your On-Premises Environment
To do this there are [three pre-requistes](https://docs.microsoft.com/en-us/azure/azure-vmware/tutorial-expressroute-global-reach-private-cloud#prerequisites) - 

1. You have established connectivity to/from AVS and Azure vNets.  If you have gotten this far, this should be complete.  
2. Have an Express Route from your on-premises environment to Azure.
3. A /29 non-overlapping network address block for the ExpressRoute Global Reach peering (you should have [defined this already](/production-ready-deployment-steps.md/#global-reach-peering) as part of the planning phase).

**NOTE:** You can connect via VPN, but that is out of scope for this kickstart document.

### Establish Global Reach Connection
Follow [these instructions](https://docs.microsoft.com/en-us/azure/azure-vmware/tutorial-expressroute-global-reach-private-cloud) to establish on-premises connectivity to/from AVS by configuring Global Reach.

### Verify On-Premises Network Connectivity
You should now see in your on-premises edge router where the Express Route connects the NSX-T network segments and the AVS management segments in the route tables.

Everyone's environment will be different, some need to allow these routes to propagate back into the greater on-premises network, some will not.  Some will have firewalls protecting the Express Route, some will not.  Assuming no firewalls and no route pruning is occurring you should at this point be able to ping from your on-premises environment the vCenter server in AVS and the [virtual machine](/production-ready-deployment-steps.md/#put-a-virtual-machine-on-the-nsx-t-network-segment) you put on the NSX-T segment.  

Additionally, from the virtual machine on the NSX-T segment you should be able to reach resources in your on-premises environment.


## Deployment and Configuration of HCX for Network Extension and/or Workload Migration

### Active HCX in AVS
... insert info here ...

### Deploy and Configure HCX On-Premises
... insert info here ...