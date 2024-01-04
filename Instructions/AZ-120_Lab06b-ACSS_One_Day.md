---
lab:
    title: '06b - Overview of deployment and maintenance of Azure Center for SAP solutions (ACSS)'
    module: 'Design and implement an infrastructure to support SAP workloads on Azure'
---

# AZ 1006 Module: Design and implement an infrastructure to support SAP workloads on Azure
# AZ-1006 1-day course lab: Overview of deployment and maintenance of Azure Center for SAP solutions (ACSS)

Estimated Time: 100 minutes

All tasks in this AZ-1006 1-day course lab are performed from the Azure portal

## Objectives

After completing this lab, you will be able to:

- Deploy and maintain the infrastructure hosting SAP workloads in Azure by using Azure Center for SAP solutions

## Instructions

### Exercise 0: Create and configure the virtual network that will host the deployment

  >**Note**: Check with your Instructor if you will use a bicep scripted solution for exercise zero.

1. On the lab computer, in the Microsoft Edge window displaying the Azure portal, in the **Search** text box, search for and select **Virtual networks**.
1. On the **Virtual networks** page, select **+ Create**.
1. On the **Basics** tab of the **Create virtual network** page, specify the following settings and select **Next**:

    |Setting|Value|
    |---|---|
    |Subscription|The name of the Azure subscription you are using in this lab|
    |Resource group|**CONTOSO-VNET-RG**|
    |Virtual network name|**CONTOSO-VNET**|
    |Region|the name of the Azure region in which you provisioned resources earlier in this lab|

1. On the **Security** tab, accept the default settings and select **Next**.

    >**Note**: You could provision at this point both Azure Bastion and Azure Firewall, however you will provision them separately once the virtual network is created.

1. On the **IP addresses** tab, specify the following settings and then select **Review + create**:

    |Setting|Value|
    |---|---|
    |IP address space|**10.5.0.0/16 (65,536 addresses)**|

    >**Note**: Delete any pre-created subnet entries. You will add subnets after the virtual network is created.

1. On the **Review + create** tab, wait for the validation process to complete and select **Create**.
1. Navigate back to the **Virtual networks** page, select the **CONTOSO-VNET** entry. 
1. On the **CONTOSO-VNET** page, in the vertical menu bar on the left side of the page, select **Subnets**.
1. On the **CONTOSO-VNET \| Subnets** page, select **+ Subnet**. 
1. On the **Add subnet** pane, specify the following settings and select **Save**:

    |Setting|Value|
    |---|---|
    |Name|**app**|
    |Subnet address range|**10.5.0.0/24**|
    |Network security group|**ACSS-DEMO-NSG**|
    |Route table|**ACSS-ROUTE**|

1. Back on the **CONTOSO-VNET \| Subnets** page, select **+ Subnet**. 
1. On the **Add subnet** pane, specify the following settings and select **Save**:

    |Setting|Value|
    |---|---|
    |Name|**AzureBastionSubnet**|
    |Subnet address range|**10.5.1.0/26**|

1. Back on the **CONTOSO-VNET \| Subnets** page, select **+ Subnet**. 
1. On the **Add subnet** pane, specify the following settings and select **Save**:

    |Setting|Value|
    |---|---|
    |Name|**db**|
    |Subnet address range|**10.5.2.0/24**|
    |Network security group|**ACSS-DEMO-NSG**|
    |Route table|**ACSS-ROUTE**|

1. Back on the **CONTOSO-VNET \| Subnets** page, select **+ Subnet**. 
1. On the **Add subnet** pane, specify the following settings and select **Save**:

    |Setting|Value|
    |---|---|
    |Name|**AzureFirewallSubnet**|
    |Subnet address range|**10.5.3.0/24**|

### Task 8: Deploy Azure Firewall into the virtual network that will host the deployment

>**Note**: Before you deploy an Azure Firewall instance, you will first create a firewall policy and a public IP address to be used by the instance.

1. On the lab computer, in the Microsoft Edge window displaying the Azure portal, in the **Search** text box, search for and select **Firewall Policies**.
1. On the **Firewall Policies** page, select **+ Create**.
1. On the **Basics** tab of the **Create an Azure Firewall Policy** page, specify the following settings and select **Next: DNS Settings >**:

    |Setting|Value|
    |---|---|
    |Subscription|The name of the Azure subscription you are using in this lab|
    |Resource group|**CONTOSO-VNET-RG**|
    |Name|**FirewallPolicy_contoso-firewall**|
    |Region|the name of the Azure region in which you provisioned resources earlier in this lab|
    |Policy tier|**Standard**|
    |Parent policy|**None**|

1. On the **DNS Settings** tab, accept the default **Disabled** option and select **Next: TLS inspection >**.
1. On the **TLS inspection** tab, select **Next: Rules >**.
1. On the **Rules** tab, select **+ Add a rule collection**.
1. On the **Add a rule collection** pane, specify the following settings:

    |Setting|Value|
    |---|---|
    |Name|**AllowOutbound**|
    |Rule collection type|**Network**|
    |Priority|**101**|
    |Rule collection action|**Allow**|
    |Rule collection group|**DefaultNetworkRuleCollectionGroup**|

1. On the **Add a rule collection** pane, in the **Rules** section, add a rule with the following settings:

    |Setting|Value|
    |---|---|
    |Name|**RHEL**|
    |Source type|**IP Address**|
    |Source|*|
    |Protocol|**Any**|
    |Destination Ports|*|
    |Destination Type|**IP Address**|
    |Destination|**13.91.47.76,40.85.190.91,52.187.75.218,52.174.163.213,52.237.203.198**|

    >**Note**: To identify the IP addresses to use for RHEL, refer to [Prepare network for infrastructure deployment](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network)

    |Setting|Value|
    |---|---|
    |Name|**ServiceTags**|
    |Source type|**IP Address**|
    |Source|*|
    |Protocol|**Any**|
    |Destination Ports|*|
    |Destination Type|**Service Tag**|
    |Destination|**AzureActiveDirectory,AzureKeyVault,Storage**|

    >**Note**: If preferred, it is possible to use service tags with regional scopes. 

    |Setting|Value|
    |---|---|
    |Name|**SUSE**|
    |Source type|**IP Address**|
    |Source|*|
    |Protocol|**Any**|
    |Destination Ports|*|
    |Destination Type|**IP Address**|
    |Destination|**52.188.224.179,52.186.168.210,52.188.81.163,40.121.202.140**|

    >**Note**: To identify the IP addresses to use for SUSE, refer to [Prepare network for infrastructure deployment](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network)

    |Setting|Value|
    |---|---|
    |Name|**AllowOutbound**|
    |Source type|**IP Address**|
    |Source|*|
    |Protocol|**TCP,UDP,ICMP,Any**|
    |Destination Ports|*|
    |Destination Type|**IP Address**|
    |Destination|*|

1. Select the **Add** button to save all rules.
1. Back on the **Rules** tab, select **Next: IDPS >**.
1. On the **IDPS** tab, select **Next: Threat intelligence >**.

    >**Note**: The IDPS functionality requires the Premium SKU.

1. On the **Threat intelligence** tab, review the available settings without making any changes and then select **Review + create**.
1. On the **Review + create** tab, wait for the validation process to complete and select **Create**.

    >**Note**: Wait for the provisioning of the Firewall Policy to complete. The provisioning should take about 1 minute.

1. On the lab computer, in the Microsoft Edge window displaying the Azure portal, in the **Search** text box, search for and select **Public IP addresses**.
1. On the **Public IP addresses** page, select **+ Create**.
1. On the **Basics** tab of the **Create a public IP address** page, specify the following settings and then select **Review + create**:

    |Setting|Value|
    |---|---|
    |Subscription|The name of the Azure subscription you are using in this lab|
    |Resource group|**CONTOSO-VNET-RG**|
    |Region|the name of the Azure region in which you provisioned resources earlier in this lab|
    |Name|**contoso-firewal-pip**|
    |IP Version|**IPv4**|
    |SKU|**Standard**|
    |Availability zone|**No zone**|
    |Tier|**Regional**|
    |Routing preference|**Microsoft network**|
    |Idle timeout (minutes)|**4**|
    |DNS name label|not set|

1. On the **Review + create** tab, wait for the validation process to complete and select **Create**.

    >**Note**: Wait for the provisioning of the public IP address to complete. The provisioning should take a few seconds.

1. On the lab computer, in the Microsoft Edge window displaying the Azure portal, in the **Search** text box, search for and select **Firewalls**.
1. On the **Firewalls** page, select **+ Create**.
1. On the **Basics** tab of the **Create a firewall** page, specify the following settings and then select **Review + create**:

    |Setting|Value|
    |---|---|
    |Subscription|The name of the Azure subscription you are using in this lab|
    |Resource group|**CONTOSO-VNET-RG**|
    |Name|**contoso-firewall**|
    |Region|the name of the Azure region in which you provisioned resources earlier in this lab|
    |Availability zone|**None**|
    |Firewall SKU|**Standard**|
    |Firewall management|**Use a Firewall Policy to manage this firewall**|
    |Firewall policy|**FirewallPolicy_contoso-firewall**|
    |Choose a virtual network|**Use existing**|
    |Virtual network|**CONTOSO-VNET**|
    |Public IP address|**contoso-firewall-pip**|
    |Forced tunneling|**Disabled**|

    >**Note**: Wait for the provisioning of the Azure Firewall to complete. The provisioning might take about 3 minutes.

1. In the Azure portal, navigate back to the **Firewalls** page.
1. On the **Firewalls** page, select the **contoso-firewall** entry.
1. On the **contoso-firewall** page, note the **Private IP** entry set to **10.5.3.4** representing the private IP address of the Azure Firewall instance.

    >**Note**: In order for the network traffic to be routed via Azure Firewall, you need to add user defined routes to the route tables associated with the app and db subnets of the virtual network that will host the SAP deployment.

1. In the Azure portal, in the **Search** text box, search for and select **Route tables**.
1. On the **Route tables** page, select the **ACSS-ROUTE** entry.
1. On the **ACSS-ROUTE** page, select **Routes**.
1. On the **ACSS-ROUTE \| Routes** page, select **+ Add**.
1. On the **Add route** pane, specify the following settings and then select **Add**:

    |Setting|Value|
    |---|---|
    |Route name|**Firewall**|
    |Destination type|**IP Addresses**|
    |Destination IP Addresses/CIDR ranges|**0.0.0.0/0**|
    |Next hop type|**Virtual appliance**|
    |Next hop address|**10.5.3.4**|

### Task 9: Deploy Azure Bastion into the virtual network that will host the deployment

1. On the lab computer, in the Microsoft Edge window displaying the Azure portal, in the **Search** text box, search for and select **Bastions**. 
1. On the **Bastions** page, select **+ Create**.
1. On the **Basics** tab of the **Bastions** page, specify the following settings and select **Next : Tags >**:

    |Setting|Value|
    |---|---|
    |Subscription|The name of the Azure subscription you are using in this lab|
    |Resource group|**CONTOSO-VNET-RG**|
    |Name|**ACSS-BASTION**|
    |Region|the name of the Azure region in which you provisioned resources earlier in this lab|
    |Tier|**Basic**|
    |Instance count|**2**|
    |Virtual network|**CONTOSO-VNET**|
    |Subnet|**AzureBastionSubnet**|
    |Public IP address|**Create new**|
    |Public IP address name|**ACSS-BASTION-PIP**|

1. On the **Tags** tab, select **Next : Advanced >**
1. On the **Advanced** tab, review the available settings without making any changes and then select **Next : Review + create >**
1. On the **Review + create** tab, wait for the validation process to complete and select **Create**.

    >**Note**: Do not wait for the provisioning of the Bastion host to complete. Instead, proceed to the next task. The provisioning might take about 15 minutes.

### Exercise 1: Deploy the infrastructure that will host SAP workloads in Azure by using Azure Center for SAP solutions

Duration: 40 minutes

In this exercise, you will perform deployment of Azure Center for SAP solutions. This will include the following activity:

- Use Azure Center for SAP solutions to deploy the infrastructure capable of hosting SAP workloads into an Azure subscription.

This activity corresponds to the following task of this exercise:

- Task 1: Create Virtual Instance for SAP (VIS) solutions

>**Note**: Following the successful deployment, you could proceed with installing SAP software by using Azure Center for SAP solutions. However, installing of SAP software is not included in this lab. 

>**Note**: For information regarding installing SAP software by using Azure Center for SAP solutions, refer to the Microsoft Learn documentation that describes how to [Get SAP installation media](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/get-sap-installation-media) and [Install SAP software](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/install-software). 

### Task 1: Create Virtual Instance for SAP (VIS) solutions

1. On the lab computer, in the Microsoft Edge window displaying the Azure portal, in the **Search** text box, search for and select **Azure Center for SAP Solutions**. 
1. On the **Azure Center for SAP Solutions \| Overview** page, select **Create a new SAP system**.
1. On the **Basics** tab of the **Create Virtual Instance for SAP solutions** page, specify the following settings and select **Next : Virtual machines**

   |Setting|Value|
   |---|---|
   |Subscription|The name of the Azure subscription you are using in this lab|
   |Resource group|The name of a **new** resource group **acss-vi-RG**|
   |Name (SID)|**VI1**|
   |Region|the name of the Azure region hosting the ACSS-registered SAP deployment or another region in the same geography|
   |Environment type|**Non-production**|
   |SAP product|**S/4HANA**|
   |Database|**HANA**|
   |HANA scale method|**Scale up (recommended)**|
   |Deployment type|**Distributed with High Availability (HA)**|
   |Compute availability|**99.95 (Availability Set)**|
   |Virtual network|**CONTOSO-VNET**|
   |Application subnet|**app (10.5.0.0/24)**|
   |Database subnet|**db (10.5.2.0/24)**|
   |Application OS image options|**Use a marketplace image**|
   |Application OS image|**Red Hat Enterprise Linux 8.2 for SAP Applications - x64 Gen2 latest**|
   |Database OS image options|**Use a marketplace image**|
   |Database OS image|**Red Hat Enterprise Linux 8.2 for SAP Applications - x64 Gen2 latest**|
   |SAP transport option|**Create a new SAP transport directory**|
   |Transport resource group|**ACSS-DEMO**|
   |Storage account name|no entry|
   |Authentication type|**SSH public**|
   |Username|**contososapadmin**|
   |SSH public key source|**Generate new key pair**|
   |Key pair name|**contosovi1key**|
   |SQP FQDN|**sap.contoso.com**|
   |Managed identity source|**Use existing user assigned managed identity**|
   |Managed identity name|**Contoso-MSI**|

1. On the **Virtual machines** tab, specify the following settings:

   |Setting|Value|
   |---|---|
   |Generate Recommendation based on|**SAP Application Performance Standard (SAPS) - Select this option to provide SAPS for application tier and Database Memory size and click Generate Recommendations**|
   |SAPS for application tier|**1000**|
   |Memory size for database (GiB)|**128**|

1. Select **Generate Recommendation**.
1. Review the size and the number of VMs for the ASCS, application, and database virtual machines. 

   >**Note**: If needed, adjust the recommended sizes by selecting the **See all Sizes** link for each set of virtual machines and choosing an alternative size. By default, the distributed deployment type with high availability as well as the application tier SAPS and database memory size specified above results in the following minimum VM SKU recommendations:
   - 2 x Standard_D4ds_v5 for the ASCS VMs (4 vCPUs and 16 GiB of memory each)
   - 2 x Standard_D4ds_v5 for the application VMs (4 vCPUs and 16 GiB of memory each)
   - 2 x Standard_E16ds_v5 for the database VMs (16 vCPUs and 128 GiB of memory each)

   >**Note**: If needed, you can request quota increase by selecting the **Request Quota** link for a specific SKU of virtual machines and submitting a quota increase request. The processing of a request typically takes a few minutes.

   >**Note**: Azure Center for SAP solutions enforces the usage of the SAP supported VM SKUs during deployment.

1. On the **Virtual machines** tab, in the **Data disks** section, select the **View and customize configuration** link.
1. On the **Database disk configuration** page, review the recommended configuration without making any changes and select **Close**.
1. Back on the **Virtual machines** tab, select **Next : Visualize Architecture**.
1. On the **Visualize Architecture** tab, review the diagram illustrating the recommended architecture and select **Review + create**.
1. On the **Review + create** tab, wait for the validation process to complete, select the checkbox confirm that you have ample quota available in the deployment region to avoid running into "Insufficient Quota" error, and select **Create**.
1. When prompted, in the **Generate new key pair** window, select **Download private key and create resource**.

   >**Note**: The private key required to connect to the Azure VMs included in the deployment will be downloaded to the computer from which you are running this lab.

   >**Note**: Wait for the deployment to complete. This might take about 25 minutes.

   >**Note**: Following the deployment, you could proceed to installing SAP software by using Azure Center for SAP solutions. In this lab, you will explore the capabilities of the Azure Center for SAP solutions without installing SAP software.

### Exercise 2: Maintain SAP workloads in Azure by using Azure Center for SAP solutions

Duration: 60 minutes

In this exercise, you will review post-deployment management and monitoring of SAP workloads by using Azure Center for SAP solutions. This will include the following activities:

- Implementing prerequisites for backup of SAP workloads managed by Azure Center for SAP solutions 
- Implementing prerequisites for disaster recovery of SAP workloads managed by Azure Center for SAP solutions 
- Reviewing monitoring options available for SAP workloads managed by Azure Center for SAP solutions 
- Deleting all of the Azure resources provisioned in this lab.

These activities correspond to the following tasks of this exercise:

- Task 1: Implement prerequisites for backup of SAP workloads managed by Azure Center for SAP solutions 
- Task 2: Implement prerequisites for disaster recovery of SAP workloads managed by Azure Center for SAP solutions 
- Task 3: Review monitoring options for SAP workloads managed by Azure Center for SAP solutions 
- Task 4: Delete the Azure resources provisioned in this lab

#### Task 1: Implement prerequisites for backup of SAP workloads managed by Azure Center for SAP solutions

>**Note**: When you configure Azure Backup at the VIS resource level in the Azure Center for SAP solutions, you can, in one step, enable backup for Azure VMs hosting the database, application servers, and SAP Central Services instance, and for the HANA DB. For the HANA DB backup, Azure Center for SAP solutions automatically runs the backup pre-registration script.

1. On the lab computer, in the Microsoft Edge window displaying the Azure portal, in the **Search** text box, search for and select **Azure Center for SAP Solutions**.
1. On the **Azure Center for SAP Solutions \| Overview** page, in the vertical navigation menu on the left side, select **Virtual instances for SAP solutions** and, in the list of virtual instances, select the instance you deployed in the previous exercise.
1. On the virtual instance page, in the vertical navigation menu on the left side, in the **Operations** section, select **Backup (preview)**.
1. Note the message indicating that backup can't be setup since SAP software installation/registration for this SAP system is not complete.

   >**Note**: This is expected. You will not able to set up backup this way until the installation of SAP software is completed. However, completing the setup also involves additional prerequisites, including creation of vaults and backup policies, which we'll review here. 

1. On the lab computer, in the Microsoft Edge window displaying the Azure portal, in the **Search** text box, search for and select **Backup center**. 
1. On the **Backup center** page, in the vertical navigation menu on the left side, in the **Manage** section, select **Vaults**.
1. On the **Backup center \| Vaults** page, select **+ Vault**.
1. On the **Start: Create Vault** page, review available vault types, ensure that **Recovery services vault** (which supports **Azure virtual machines** and **SAP HANA in Azure VM** datasource types) is selected, and then select **Continue**.
1. On the **Basics** tab of the **Create Recovery Services vault** page, specify the following settings and select **Next : Redundancy**

   |Setting|Value|
   |---|---|
   |Subscription|The name of the Azure subscription you are using in this lab|
   |Resource group|The name of a **new** resource group **acss-mgmt-RG**|
   |Vault name|**acss-backup-RSV**|
   |Region|the name of the Azure region hosting the ACSS-registered SAP deployment|

1. On the **Redundancy** tab, specify the following settings and select **Next : Vault properties**

   |Setting|Value|
   |---|---|
   |Backup Storage Redundancy|**Geo-redundant**|
   |Cross Region Restore|**Enable**|

1. On the **Vault properties** tab, review the **Enable immutability** setting without enabling it and select **Next : Networking**
1. On the **Networking** tab, accept the default option to **Allow public access from all networks and select **Review + create**
1. On the **Review + create** tab, wait for the validation process to complete and select **Create**.

   >**Note**: Wait for the provisioning process to complete. The provisioning might take about 2 minutes.

1. On the **Your deployment is complete** page, select **Go to resource**.
1. On the **acss-backup-RSV** page, in the vertical navigation menu on the left side, in the **Manage** section, select **Backup policies**.
1. On the **acss-backup-RSV \| Backup policies** page, select **+ Add**.
1. On the **Select policy type** page, review the available policy types.

   >**Note**: You will start by configuring the backup policy for Azure VMs hosting the database, application servers, and SAP Central Services instance.

1. On the **Select policy type** page, select **Azure Virtual Machine**.
1. On the **Create policy** page, review the differences between the **Standard** and **Enhanced** policy sub types and select **Enhanced**.

   >**Note**: You should base your choice on your resiliency needs. Similarly, you should adjust the values listed below according to your needs.

1. In the **Policy name** text box, enter **acss-vm-enhanced-backup-policy**.
1. Set the backup schedule frequency to **Hourly**, start time to **6:00PM**, schedule to **Every 6 hours**, duration to **12 hours**, and timezone to your local time zone.
1. Set the **Retain instance recovery snapshot(s) for** value to **5** days.
1. Set **Retention of daily backup points** to **60** days.
1. Set the retention period of weekly, monthly, and yearly backup points based on your preferences.

   >**Note**: To enable tiering, you have to enable monthly or yearly backup points.

   >**Note**: You also have the option of designating the naming convention of the resource group auto-generated by Azure Backup.

1. Select **Create**.
1. On the **Your deployment is complete** page, select **Go to resource**.
1. On the **acss-backup-RSV** page, in the vertical navigation menu on the left side, in the **Manage** section, select **Backup policies**.
1. On the **acss-backup-RSV \| Backup policies** page, select **+ Add**.

   >**Note**: Next, you will configure the backup policies for HANA DB. The configuration presented here follows the guidance described in the MS Learn article [Back up SAP HANA database instance snapshots on Azure VMs](https://learn.microsoft.com/en-us/azure/backup/sap-hana-database-instances-backup).

1. On the **Select policy type** page, select **SAP HANA in Azure VM (Database via Backint)**
1. On the **Create policy** page, in the **Policy name** text box, enter **acss-hanadb-backint-backup-policy**.
1. Accept the default settings for the full and log backups. Keep the differential and incremental backups disabled.

   >**Note**: You have the option of enabling HANA backup compression and moving eligible recovery points to vault archive. 

1. Select **Create**.

1. On the **Your deployment is complete** page, select **Go to resource**.
1. On the **acss-backup-RSV** page, in the vertical navigation menu on the left side, in the **Manage** section, select **Backup policies**.
1. On the **acss-backup-RSV \| Backup policies** page, select **+ Add**.
1. On the **Select policy type** page, select **SAP HANA in Azure VM (DB Instance via snapshot)**
1. On the **Create policy** page, in the **Policy name** text box, enter **acss-hanadb-snapshot-backup-policy**.
1. Set the frequency of snapshot backup to 1:30 PM of your current time zone.
1. Configure to retain instant recovery snapshot for **5** days.
1. Keep the default selection of the snapshot resource group, which should be set to **acss-mgmt-RG**.
1. In the **Managed Identity** section, select **Create Managed Identity**. This will automatically open another tab in the web browser window, displaying the **Managed Identities** page in the Azure portal.
1. On the **Managed Identities** page, select **+ Create**.
1. On the **Basics** tab of the **Create User Assigned Managed Identity** page, specify the following settings and then select **Review + Create**:

   |Setting|Value|
   |---|---|
   |Subscription|The name of the Azure subscription you are using in this lab|
   |Resource group|**acss-mgmt-RG**|
   |Region|the name of the Azure region hosting the ACSS deployment|
   |Name|**acss-mgmt-MI**|

1. On the **Review** tab, wait for the validation process to complete and select **Create**.

   >**Note**: Wait for the provisioning process to complete. The provisioning should take just a few seconds.

1. Close the current browser tab and switch back to the one displaying the **Create policy** page.
1. In the **Managed Identity** section, in the **Resource group** drop-down list, select **acss-mgmt-RG** and, in the **Managed identity** drop-down list, select **acss-mgmt-MI**.
1. Select **Create**.

   >**Note**: When configuring backup at the VIS level in the Azure Center for SAP solutions interface, you will be able to leverage the existing vault and its policies.

   >**Note**: Once backup is configured at the VIS level, you can monitor the status of backup jobs of Azure VMs and HANA DB from the VIS interface in the Azure portal.

#### Task 2: Implement prerequisites for disaster recovery of SAP workloads managed by Azure Center for SAP solutions 

>**Note**: While Azure Center for SAP solutions service is a zone redundant service, there is no Microsoft initiated failover in the event of a region outage. To remediate such scenarios, you should configure disaster recovery for SAP systems that you deploy using Azure Center for SAP solutions by following guidance described in [Disaster recovery overview and infrastructure guidelines for SAP workload](https://learn.microsoft.com/en-us/azure/sap/workloads/disaster-recovery-overview-guide), which involves the use of Azure Site Recovery (ASR). In this task, you'll step through the process of implementing an ASR-based disaster recovery solution which relies on that guidance.

>**Note**: ASR is the recommended solution for application servers and SAP Central Services instances. For database servers, you should consider using native their replication functionality.

1. On the lab computer, in the Microsoft Edge window displaying the Azure portal, in the **Search** text box, search for and select **Recovery Services vaults**.
1. On the **Recovery Services vaults** page, select **+ Create**.
1. On the **Basics** tab of the **Create Recovery Services vault** page, specify the following settings (leave others with their default values) and select **Next: Redundancy**.

   |Setting|Value|
   |---|---|
   |Subscription|The name of the Azure subscription you are using in this lab|
   |Resource group|The name of a **new** resource group **acss-dr-RG**|
   |Vault name|**acss-dr-RSV**|
   |Region|the name of the Azure region that is paired up with the ACSS-registered SAP deployment|

   >**Note**: To identify the region which is a paired up with the one hosting your production workloads, refer to MS Learn documentation describing [Azure paired regions](https://learn.microsoft.com/en-us/azure/reliability/cross-region-replication-azure#azure-paired-regions).

1. On the **Redundancy** tab, specify the following settings and select **Next : Vault properties**

   |Setting|Value|
   |---|---|
   |Backup Storage Redundancy|**Locally-redundant**|

1. On the **Vault properties** tab, review the **Enable immutability** setting without enabling it and select **Next : Networking**
1. On the **Networking** tab, accept the default option to **Allow public access from all networks and select **Review + create**
1. On the **Review + create** tab, wait for the validation process to complete and select **Create**.

   >**Note**: Do not wait for the provisioning process to complete but instead proceed to the next step. The provisioning might take about 2 minutes.

   >**Note**: Now you will set up the disaster recovery environment in the paired up region in which you created the Recovery Services vault. This environment will include a virtual network that will host replicas of the Azure VMs currently hosted in the primary region where you provisioned Virtual Instance for SAP. 

1. On the lab computer, in the web browser window displaying the Azure portal, in the **Search** text box, search for and select **Virtual networks**. 
1. On the **Virtual networks** page, select **+ Create**.
1. On the **Basics** tab of the **Create virtual network** page, specify the following settings and select **Next**:

   |Setting|Value|
   |---|---|
   |Subscription|The name of the Azure subscription you are using in this lab|
   |Resource group|**acss-dr-RG**|
   |Virtual network name|**CONTOSO-VNET**|
   |Region|the name of the same Azure region you used when creating the Recovery Services vault earlier in this task|

1. On the **Security** tab, accept the default settings and select **Next**.

   >**Note**: You could provision at this point both Azure Bastion and Azure Firewall, however you should instead automate their provisioning as part of your disaster recovery failover procedure. This will minimize charges associated with maintaining the disaster recovery environment. The same should apply to other components of that environment that mirror the configuration of the primary Virtual Instance for SAP, such as Azure Premium file shares and custom routing.

1. On the **IP addresses** tab, specify the following settings and then select **Review + create**:

   |Setting|Value|
   |---|---|
   |IP address space|**10.10.0.0/16 (65,536 addresses)**|

   >**Important**: Note that the IP address spaces differ between the virtual network in the primary and secondary regions. This is intentional, since it will allow connecting the two virtual network together, which is necessary in order to configure replication between database servers hosted in the two regions. Such connection can be established by using virtual network peering. 

1. In the list of subnets, select the trash bin icon to delete the **default** subnet.
1. Select **+ Add a subnet**.
1. In the **Add a subnet** pane, specify the following settings and then select **Add** (leave others with their default values):

   |Setting|Value|
   |---|---|
   |Name|**app**|
   |Starting address|**10.10.0.0**|
   |Size|**/24 (256 addresses)**|

1. Select **+ Add a subnet**.
1. In the **Add a subnet** pane, specify the following settings and then select **Add** (leave others with their default values):

   |Setting|Value|
   |---|---|
   |Name|**db**|
   |Starting address|**10.10.2.0**|
   |Size|**/24 (256 addresses)**|

1. On the **IP addresses** tab, select **Review + create**:
1. On the **Review + create** tab, wait for the validation process to complete and then select **Create**.

   >**Note**: Do not wait for the provisioning process to complete but instead proceed to the next task. The provisioning should take just a few seconds.

1. On the lab computer, in the Microsoft Edge window displaying the Azure portal, in the **Search** text box, search for and select **Recovery Services vaults**.
1. On the **Recovery Services vaults** page, select **acss-dr-RSV**.
1. On the **acss-dr-RSV** page, in the vertical navigation menu on the left side, in the **Getting started** section, select **Site Recovery**.
1. On the **acss-dr-RSV \| Site Recovery** page, in the **Azure virtual machines** section, select **1. Enable replication**. 
1. On the **Source** tab of the **Enable replication** page, specify the following settings and then select **Next**:

   |Setting|Value|
   |---|---|
   |Region|the name of the Azure region hosting your Virtual Instance for SAP (VIS)|
   |Subscription|The name of the Azure subscription you are using in this lab|
   |Resource group|**acss-vi-RG**|
   |Virtual machine deployment model|**Resource Manager**|
   |Disaster recovery between availability zones|**No**|

   >**Note**: The **Disaster recovery between availability zones** might be not configurable, depending on whether the source region supports availability zones.

1. On the **Virtual machines** tab, select the first four virtual machines in the list (**vi1appvm0**, **vi1appvm1**, **vi1ascsvm0**, and **vi1ascsvm0**) and then select **Next**.

   >**Note**: As mentioned earlier, the ASR-based replication will be applied to the application servers and SAP Central Services instances. Database servers will be maintained in sync by relying on the native database replication functionality.

1. On the **Replication settings** tab, perform the following actions:

   1. If needed, in the **Target location** drop-down list, select the Azure region in which you created the **acss-dr-RSV** Recovery Services vault.
   1. Ensure that the name of the Azure subscription you are using in this lab appears in the **Target subscription** drop-down list.
   1. In the **Target resource group** drop-down list, select **acss-dr-RG**.
   1. In the **Failover virtual network** drop-down list, select **CONTOSO-VNET**.
   1. In the **Failover subnet** drop-down list, select **app (10.10.0.0/24)**.
   1. In the **Storage** section, select the **View/edit storage configuration** link.
   1. On the **Customize target settings** page, review the resulting configuration but do not make any changes and select **Cancel**.
   1. In the **Availability options** section, select the **View/edit availability options** link.
   1. On the **Availability options** page, note that you have the option of implementing proximity placement groups for target resources but do not make any changes and select **Cancel**.

   >**Note**: You also have the option of configuring capacity reservation.

1. Back on the **Replication settings** tab of the **Enable replication** page, select **Next**.
1. On the **Manage** tab of the **Enable replication** page, perform the following actions:

   1. Below the **Replication policy** drop-down list, select **Create new**.
   1. In the **Create replication policy** pane, in the **Name** text box, enter **2-day-retention-policy**, in the **Retention period (in days)** text box, enter **2**, and then select **OK**.
   1. Note that you have the option of creating replication groups. This option is not applicable to our use case.
   1. Leave **Update settings** option of **Extension settings** configured to **Allow ASR to manage**.
   1. Accept the default, automatically assigned name of the **Automation account**.

1. On the **Manage** tab of the **Enable replication** page, select **Next**.
1. On the **Review** tab of the **Enable replication** page, select **Enable replication**.

   >**Note**: Now you will step through enabling replication for the remaining two Azure VMs hosting the database servers.

1. Back on the **acss-dr-RSV \| Site Recovery** page, in the **Azure virtual machines** section, select **1. Enable replication**. 
1. On the **Source** tab of the **Enable replication** page, specify the following settings and then select **Next**:

   |Setting|Value|
   |---|---|
   |Region|the name of the Azure region hosting your Virtual Instance for SAP (VIS)|
   |Subscription|The name of the Azure subscription you are using in this lab|
   |Resource group|**acss-vi-RG**|
   |Virtual machine deployment model|**Resource Manager**|
   |Disaster recovery between availability zones|**No**|

   >**Note**: The **Disaster recovery between availability zones** might be not configurable, depending on whether the source region supports availability zones.

1. On the **Virtual machines** tab, select the first four virtual machines in the list (**vi1appvm0**, **vi1appvm1**, **vi1ascsvm0**, and **vi1ascsvm0**) and then select **Next**.

   >**Note**: You need to configure replication for the remaining two Azure VMs separately, since they are connected to a different subnet.

1. On the **Replication settings** tab, perform the following actions:

   1. If needed, in the **Target location** drop-down list, select the Azure region in which you created the **acss-dr-RSV** Recovery Services vault.
   1. Ensure that the name of the Azure subscription you are using in this lab appears in the **Target subscription** drop-down list.
   1. In the **Target resource group** drop-down list, select **acss-dr-RG**.
   1. In the **Failover virtual network** drop-down list, select **CONTOSO-VNET**.
   1. In the **Failover subnet** drop-down list, select **app (10.10.0.0/24)**.
   1. In the **Storage** section, select the **View/edit storage configuration** link.
   1. On the **Customize target settings** page, review the resulting configuration but do not make any changes and select **Cancel**.
   1. In the **Availability options** section, select the **View/edit availability options** link.
   1. On the **Availability options** page, note that you have the option of implementing proximity placement groups for target resources but do not make any changes and select **Cancel**.

   >**Note**: You also have the option of configuring capacity reservation.

1. Back on the **Replication settings** tab of the **Enable replication** page, select **Next**.
1. On the **Manage** tab of the **Enable replication** page, perform the following actions:

   1. Below the **Replication policy** drop-down list, select **Create new**.
   1. In the **Create replication policy** pane, in the **Name** text box, enter **2-day-retention-policy**, in the **Retention period (in days)** text box, enter **2**, and then select **OK**.
   1. Note that you have the option of creating replication groups. This option is not applicable to our use case.
   1. Leave **Update settings** option of **Extension settings** configured to **Allow ASR to manage**.
   1. Accept the default, automatically assigned name of the **Automation account**.

1. On the **Manage** tab of the **Enable replication** page, select **Next**.
1. On the **Review** tab of the **Enable replication** page, select **Enable replication**.

   >**Note**: Initial replication might take a considerable time to complete. Considering the limited time allocated to this lab, it is not feasible to illustrate the actual failover, so the remaining steps of this exercise are optional and dependent primarily on the amount of time remaining till the end of the session.

1. On the **acss-dr-RSV** page, in the vertical navigation menu on the left side, in the **Protected items** section, select **Replicated items**.
1. On the **acss-dr-RSV \| Replicated items** page, review the replication status of the four Azure VMs.

   >**Note**: It might take a few minutes before the replication status is displayed. The status should be initially set to **Enabling replication** and then transition to **x% Synchronized**, until the synchronization process completes. At that stage, the status will change to **Waiting for first recovery point** and finally switch to **Protected**. Wait until the status of the Azure VMs listed on the **acss-dr-RSV \| Replicated items** page changes to **Protected**. This indicates readiness to initiate either a test failover (if you intend to validate the resulting setup without invoking a disaster recovery) or a failover (if you want to transition the workload hosted on the primary instance to its replica). Either form of the failover will provision and bring online replicas of the primary region Azure VMs in the disaster recovery region. 

1. To perform a test failover or failover, on the **acss-dr-RSV \| Replicated items** page, select the entry representing one of the protected Azure VMs and then, on the **Replicated items** page of that Azure VM, select either **Test Failover** or **Failover**. 

   >**Note**: If you select **Test Failover**, you will be prompted to provide the recovery point to use and the Azure virtual network to which the replica Azure VM should be connected. You could potentially use for this purpose the **CONTOSO-VNET** Azure virtual network. Alternatively, you could implement a fully isolated test environment, although keep in mind that you'll need to provide connectivity to a database server replica in order to perform a more comprehensive testing.

   >**Note**: Once a test failover is completed, you invoke **Cleanup test failover** from individual **Replicated items** pages to simply delete the replica Azure VM. If you invoked a failover, you'll need to **Commit** your changes to the replica Azure VM once the failover completes, and then **Re-protect** it by reversing the direction of replication, effectively treating your primary region as the new disaster recovery site. To return to the original arrangement, perform another failover, this time from the secondary region to the primary one.

#### Task 3: Review monitoring options for SAP workloads managed by Azure Center for SAP solutions 

>**Note**: As with backup, you will not be able to fully experience the monitoring capabilities of Azure Center for SAP solutions. This requires installation of SAP software or registering an existing Azure Monitor for SAP solutions instance. Instead, in this task, you will step through the interface available in the Virtual Instance for SAP solutions to identify and review these capabilities.

1. On the lab computer, in the Microsoft Edge window displaying the Azure portal, in the **Search** text box, search for and select **Virtual Instances for SAP solutions**. 
1. On the **Virtual Instances for SAP solutions** page, review the summarized status information for the **VI1** instance, including the overall health and status visual indicators.
1. On the **Virtual Instances for SAP solutions** page, select **VI1**.
1. On the **VI1** page, in the vertical navigation menu on the left side, select **Overview** and then, in the pane on the right side, select **Monitoring**.
1. Review the monitoring telemetry displayed in the monitoring pane.

   >**Note**: The monitoring pane includes vCPU utilization graphs and metrics for application servers, database servers, and SAP Central Services instances. It also includes database disk IOPS statistic database servers. 

1. On the **VI1** page, in the vertical navigation menu on the left side, in the **Monitoring** section, select **Quality insights**.
1. On the **VI1 \| Quality insights \| Workbook 1** page, review the **Advisor Recommendation** tab, which is intended to provide recommendations for optimizing Virtual Instance for SAP solutions (VIS), Central server instance, App service instances, and databases.

   >**Note**: These recommendations require completing installation of SAP software.

1. On the **VI1 \| Quality insights \| Workbook 1** page, select the **Virtual Machine** tab and review the content of **Azure Compute**, **Compute List**, **Compute Extensions**, **Compute + OS Disk**, and **Compute + Data Disks** tabs.

   >**Note**: Each of these tabs should include actual data collected from the Azure VMs that are part of the Virtual Instance for SAP solutions.

1. On the **VI1 \| Quality insights \| Workbook 1** page, select the **Configuration Checks** tab and review the content of the **Accelerated Networking**, **Public IP**, **Backup**, and **Load Balancer** tabs. This content provides a quick overview of the performance and security related settings of compute and network components of the Virtual Instance for SAP solutions. The **Load Balancer** tab includes **Load Balancer Monitor** information that displays key load balancer metrics.
1. On the **VI1** page, in the vertical navigation menu on the left side, in the **Monitoring** section, select **Azure Monitor for SAP solutions**.
1. On the **VI1 \| Azure Monitor for SAP solutions** page, note the message stating that AMS cannot be setup since SAP software installation\registration for VIS is not complete.

   >**Note**: Once you install SAP software, you will be able to integrate it with a new or existing Azure Monitor for SAP solutions resource. Azure Monitor for SAP solutions relies on the Azure Monitor capabilities of Log Analytics and workbooks to provide a comprehensive monitoring of SAP workloads hosted on Azure VMs, including support for custom visualizations, queries, and alerts.