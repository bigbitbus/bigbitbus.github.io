---
layout:     post
title:      Google Cloud's E2 VMs compared to the N2 VMs
published:  true
---

In this post we compare Google Cloud's E2- class of virtual machines with N2- class of virtual machines. We compare their performance and cost so you can make data-driven decisions when choosing between the two classes.

## E2 and N2 VM Products

Here is how the [Google cloud VM types](https://cloud.google.com/compute/docs/machine-types) page describes the E2 and N2 products:

"E2 machine types are cost-optimized VMs that offer up to 32 vCPUs with up to 128 GB of memory with a maximum of 8 GB per vCPU. N2 machine types are the second generation general-purpose machine types that offer flexible sizing between 2 to 80 vCPUs and 0.5 to 8 GB of memory per vCPU."


The description suggests that the E2 vCPU is a cost effective as compared to the N2 but has a lower performance. We wanted to understand the relative performance characteristics of the E2 and N2 VMs' vCPUs. So we ran the [stress tool  ](https://wiki.ubuntu.com/Kernel/Reference/stress-ng) to compare vCPU performance. Here is what we found when we compared an E2-medium (2vCPU, 4GB RAM) with an N2-Standard-2 (2vCPU, 8GB RAM). In this experiment, we spun up an E2 and N2 virtual machine and ran the stress tool on them. The stress tool can cycle through 100s of different CPU-consuming tasks (BOGO operations on the y-axis below) and adjust their frequency until the desired utilization (stress percent in the x-axis below) is achieved.

<p align="center">
<img src="https://docs.google.com/spreadsheets/d/e/2PACX-1vS0riPn6tgRNsBP1ONVr-ALqMRgMudq4eN61hUIqBGbO92DdcPmEH5QKlm-4JDks5Ly6tvqM0j0WD5C/pubchart?oid=1451668472&format=image"/>
</p>

The E2 is less performant than the N2. The remarkable insight from the graph is that the difference in their performance is dependent on the load applied to the VM. In fact the E2 VM closely follows the N2 until about 25% utilization (stress percent), but afterwards its performance falls off as compared to the N2. 

Since its unlikely that the silicon was designed with this characteristic we delved into E2 documentation and came across [Google's blog](https://cloud.google.com/blog/products/compute/understanding-dynamic-resource-management-in-e2-vms) on the how E2 VMs work. These VMs use "Performance-driven dynamic resource management". E2 VMs are more tightly packed into physical cores and the idea is to continiously monitor the VM's usage and "live migrate" it to another hypervisor as and when needed. The blogpost notes that users should consider N2 or C2 VMs for higher-CPU loads. So we believe that the non-linear E2 behavior we saw in our experiments was a result of triggering the dynamic resource management algorithms.

## Pricing

You can use our free [B3Console](https://b3console.bigbitbus.com/login) tool to compare VMs across different cloud providers, but we have collected the on-demand pricing per month (720 hours) for E2 and N2 VMs in Google's us-central1 datacenter (as of June 10th, 2021):

| Product | vCPU pm| 1 GB RAM pm |
| -------- | ------------- | ----- |
| E2 | $15.70 | $2.10 |
| N2 |  $22.76 | $3.05 | 
| ---- | ------ | ------ |


N2 is at an approximate 50% premium as compared to E2.

## Outlook

So, which of the product families should you choose? For low utilization workloads (your workload seldom spikes about 25% CPU utilization) the E2 is a great option; this includes a lot of dev/test environments for example.

But for workloads that demand consistency and constant performance go with the N2. Thats because its almost certainly more expensive to scale compute horizontally than vertically. For example a 100 VM N2 cluster running any distributed application will probably outperform a 150 VM E2 cluster, simply because coordinating nodes in distributed systems is slow and expensive given the relatively slow networks that connect VMs. 

I would also be wary of using E2 VMs in production workloads where usage patterns can be very non-linear and unpredictable. You don't want to be scratching your head about why your application that supported 10K users yesterday suddenly struggles with 7K users today, just because the E2 VMs hosting your application do not guarantee consistent performance!

*

_Sachin Agarwal is a computer systems researcher and the founder of BigBitBus._

_BigBitBus is on a mission to bring greater transparency in public cloud and managed big data and analytics services._






