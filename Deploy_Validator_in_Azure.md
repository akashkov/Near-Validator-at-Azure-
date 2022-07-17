#  Deployment of shardnet.near node in Azure

**Usefull links:**

Wallet: https://wallet.shardnet.near.org/

Explorer: https://explorer.shardnet.near.org/

Azure Free Account: https://azure.microsoft.com/en-ca/free/

Azure Portal: https://portal.azure.com/

#
**Step 1 -  Create free Azure Account/Subscription**
  * Visit https://azure.microsoft.com/en-ca/free/ and click "Start Free"
  * Provide existing or create a new Microsot account (Office 365, Outlook, Hotmail)
  * Complete creation of free Azure account. You will be provided you with credit that depends on your region.
  * Login to Azure portal https://portal.azure.com/ using your account and select to upgrade your account to Pay-As-You-Go. 
    You still can use your credit but also will get access to all hardware tiers including Premium SSD storage.
  * Waite until upgrade for your account will be comleted.
##
**Step 2 -  Create Azure resources for Node deployment**

       The following Azure resources will be requrd: 
       Resource Group - Logical container to keep information about rest of resources. (neer_rg)
       Virtual Network(vNet) - IP address space that will be used. (near_vnet 10.0.0.0/24) 
       Subnet - Network subnet that is part of vNet address space. (default 10.0.0.0/25)
       
  * Create Resource Group by navigating to  
       
