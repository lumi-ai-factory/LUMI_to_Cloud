---
title: "Microsoft Azure"
nav_order: 4
---

# Microsoft Azure

This section covers how to migrate your workloads from LUMI to Microsoft Azure.

## Prerequisites

Before starting the migration, ensure you have the following ready:

- **An active Azure account**: 
- **An Azure subscription linked to the Azure acount**: With a free subscription, you will initially have 200 dollars worth of resources to test and play around with services. However, upon expiry, these resources will not be accessible unless a new subscription is initiated. 
- **A LUMI account**: You should have your AI models ready on LUMI.

## 1. Ways to host on Azure 
There are currently 2 methods to do this. Fundamentally, they are connected to each other in the Virtual Machine infrastructure level. 

- **Launching Azure Virtual Machines (VMs) directly in Azure portal**: 
    - In this method, we use the service as Infrastructure as a Service (IaaS). 
    - This method is more suitable for DevOps oriented workflow and general purpose. 
    - Suitable to set up dedicated aspect of a Cloud system such as applications, servers, database and dev/test.
- **Launching Computing Cluster in Azure Machine Learning (AML) Platform**: 
    - In this method, we use the service as Machine Learning sa a Service (MaaS). 
    - This method is more suitable for Machine Learning workflow. 
    - Suitable to set up notebooks and machine learning training and experiments with integrated machine learning workspace, jobs and containers. 

Choose the suitable method to host your model is very much dependent on your final use case. Typically, for operational workflows, Azure Portal is more suitable. Meanwhile, AML is more suitable for machine learning project and keeping track with the R&D work. 

## 2. Permissions

## 3. Azure Blob storage
[ WIP ]
To store our datasets and model weights which are transferred from LUMI, we will use Azure Blob storage system. 

## 4. Choosing a machine size
Just like other providers, Azure also provides a wide range of machine sizes and types and it is crucial that users understand and know what would be suitable machine type for their usage. 

Overall, Azure classifies their machines based on Type, Series and Size. 

In terms of Type, the machines are categorized based on intended workload: 
- General purpose
- Compute optimized
- Memory optimized
- Storage optimized
- GPU accelerated
- FPGA accelerated

In terms of Series, the machines are categorized based on hardwares and features. 
In terms of Size, the machine are categorized based on specific VM configuration, including vCPUs, memory, and accelerators. 

Based on the name, you can breakdown the meaning of the machine family, subfamily, vCPUs, features, and version. To understand this better, please visit [Microsoft's official documentation](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes/overview?tabs=breakdownseries%2Cgeneralsizelist%2Ccomputesizelist%2Cmemorysizelist%2Cstoragesizelist%2Cgpusizelist%2Cfpgasizelist%2Chpcsizelist#name-structure-breakdown)


In both Azure and AML, virtual machiens can be deployed as they are under the same Microsoft infrastructure of Virtual machines. However, it is important to know that not all VMs types and sizes will be available on AML platform or will be available in your region. Before considering certain machine types or families from the general list, you will have to check the region and Microsoft services that are available in that region. If you are in finland, typically you will use `Central Sweden`. 

### How to check available hardware

For more information about Microsoft services availability in different regions, please visit this [general products list]( https://azure.microsoft.com/en-us/explore/global-infrastructure/products-by-region/table).

### GPU Families in Sweden (`Central Sweden`)


## 5. GPU quotas

A brand new Azure account will be rewarded with 200 dollars worth of any Azure resources to be used within the first 30 days. After this 30-day period ends, you will have to subscribe to keep using the service. 

GPU quotas in Azure will be measured based on vCPU. a vCPU (virtual central processing unit) is a share of a physical processor core or an entire core. When getting allocated an amount of quotas, it is important to check several quotas dimensions. 

**Dimensions**:
- Regional: The quotas will be allocated based on regions. Certain machine sizes / types might be unavailable in certain regions. 
- VM-family quota: The quotas will be allocated based on particular VM families. You cannot use quotas of one machine family for another machine family.
- Spot vCPU quota: the vCPU quota for spot VM is different from standard VM. 
- Other resources quota: The quotas are services dependent. 

### How to check for quotas practically


## 6. Cost

## 7. Logs
