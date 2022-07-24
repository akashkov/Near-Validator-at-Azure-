# Pricing  for running Near validator in Azure

**Useful links:**

Azure Portal: <https://portal.azure.com/>
___

**Benefits of deployment in Azure**

Azure cloud platform provides enterprise level functionality that can be utilized by deploying Validator node on this platform.
One of key benefits is Zone availability protection, when nodes can be running in same region but in different Zones/Data-centres.
Azure infrastructure provides high speed connectivity for nodes running in different Zones and it looks like nodes are running on the same L2 segment.
It creates opportunities for deploying nodes in HA mode.

___

Price is provided for Near Validator in Azure with following Configuration

    Operating system: Linux (ubuntu 20.04)
    VM Size: Standard D2as v4 - 2 vCPU, 8 GB of RAM 
    OS disk type: Standard SSD (locally-redundant storage) - 60GB
    Data disk:  Premium SSD drive - 500GB
    Accelerated networking: Enabled

<br>

Azure portal provides cost analysis.

Based on node configuration, price for running node in Azure will be around 125 $CA per month.

This price could be reduce by up to 65% by using "Reserved Instances", but it requires upfront payment.

For three years price for reserved instance would be around of 1575 $CA.

![Attach_Data_Disk](https://github.com/akashkov/Near-Validator-at-Azure-/blob/main/Subcription%20Cost.png?raw=true)
