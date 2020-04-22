<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Falankinane%2Fsingledc%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.svg?sanitize=true"/>
</a>
<a href="http://armviz.io/#/?load=httpshttps%3A%2F%2Fraw.githubusercontent.com%2Falankinane%2Fsingledc%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/visualizebutton.svg?sanitize=true"/>
</a>

<h1>Single Domain Controller to Azure with DC Promo (New Domain Forest)</h1>
<p>This template deploys an Azure VM and promotes it as a domain controller in a new domain forest.  <u>This template is not suitable if you want to add a new domain controller to an existing domain</u>.</p>

<h1>Solution Overview</h1>
<p>The goal of this template is to provide a relatively simple virtual machine deployment that is cloud-based, offering flexibility and removing the need to run on-premises servers, while maintaining a familiar working environment for end users. You’ll get a virtual machine to act as a domain controller.  The virtual machine will be promoted to a domain controller and set up as a DNS server.  The virtual network will be configured to use the domain controller as the DNS server.  The virtual machine will also be backed up as part of the deployment.</p>

<h1>Detailed Description</h1>
<b>Virtual Machines</b>

There is one virtual machine:

<b>Domain controller (vm-dc-01)</b>: This machine will be deployed as a domain controller. A 4 GB data disk is attached – you can expand the size of this disk after deployment. This data disk will be used for Active Directory (SYSVOL, database, logs), configured during DCPROMO to comply with <a href="https://msdn.microsoft.com/en-us/library/azure/jj156090.aspx#BKMK_ContrastsWithOnPrem">Active Directory support requirements</a>.

OS Disk is a Standard LRS HDD managed disk and the data disk is a Standard LRS SSD managed disk.

The available virtual machine sizes, which can be changed after deployment are as list below.

Domain controller:

Standard_B2ms (DEFAULT)<br>
Standard_B2s<br>
Standard_A2_V2<br>
Standard_A4_v2<br>
Standard_DS2_v2<br>
Standard_D2S_v3<br>
Standard_D4S_v3<br>

The latest available Azure image for Windows Server 2016 Datacenter edition (no Windows Server CALS are required) is deployed in the virtual machines.

<h1>Networking</h1>
<p>A virtual network (vnet) is deployed containing one subnet:</p>

<b>sn-dc</b>: Used for the domain controller
The virtual network is configured to use the static internal IP address (10.0.1.4) of the domain controller as the DNS server.  The DNS servers of the deployed virtual network will be updated to use this as the DNS server for the virtual network.

A basic tier Azure Load Balancer (lb-dc) is deployed as a NAT device, NATing traffic from port 50001 on the Internet translated to port 3389 for admin RDP access into the virtual machine via a single static basic tier public IP address (pip-dc-lb).

A network security group has been configured to provide Layer-4 firewall security at the subnet layer:

<b>nsg-dc-sn</b>: Allows port 3389 (TCP) into the subnet via the Load Balancer for admin RDP access to the subnet from the Internet.  <u>It is recommended to lock this down to only allow traffic from your public IP address</u>.
Management Features
A recovery services vault (rsv-dc) is created for backing up the virtual machines using Azure Backup. The deployment will set this up with the below settings.  If you do not require backup or which to change these settings then you must perform this post deployment.  <u>It is your own responsibility to make sure this backup is working correctly</u>.

Backup redundancy set to locally-redundant storage (LRS)
Domain Controller VM backed up on a 30 day retention backup policy
Boot Diagnostics is enabled on the virtual machines, using a storage account called sadiags<random string> as the storage location. This will give you a screenshot of the virtual machines’ console and enable serial console access without networking via the Azure Portal.

It is recommended that you also enable Guest-Level Monitoring in Diagnostic Settings in the virtual machine, and also enable Diagnostics Logging for all possible resources in Azure Monitor, using the diagnostics storage account as the storage location.

<h1>Deployment</h1>
<p>You will need to have an active Azure subscription in the customer’s tenant and ensure that you have administrative access to this subscription, either Owner or Contributor level access is recommended.</p>
<p>Make sure to supply the following parameter values:</p>

<b>Resource group name:</b> Name of the destination resource group.<br>
<b>Deployment Location:</b> Choose as an Azure region for deployment<br>
<b>Admin User Name:</b> Provide a legitimate administrator user name for the virtual machine guest operating systems. Note that you cannot use common names such as administrator, admin, root, and so on.<br>
<b>Admin Password:</b> Enter a password of at least 12 characters, including 3 of the following – upper case, lower case, number, and special character.<br>
<b>Domain Name:</b> Enter the fully qualified domain name (FQDN) of the domain, required for the DC Promo, e.g. contoso.com<br>
<b>AD VM Size (default: Standard_B2ms):</b> Pick a series/size for the domain controller.
