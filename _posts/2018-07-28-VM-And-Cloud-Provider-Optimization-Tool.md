---
layout: post
title: Cloud Provider and VM Optimization Tool
---

_In this article we describe the BigBitBus VM and cloud provider optimization tool, available [here](https://tools.bigbitbus.com/optimizer/)._

_The tool may not render correctly on lower-resolution mobile device screens; for the best experience please use a desktop._

The VM and cloud provider optimization tool displays relative CPU performance of different virtual machines across different providers. The current tool covers many VMs of up to 16 vcpus from Amazon AWS, Google GCP, Microsoft Azure and Digital Ocean. The CPU performance is reported based on our VM CPU benchmarking tests. Other VM characteristics, such as the number of vCPUs, RAM, and cost have been extracted via provider APIs or (web-)scraped from cloud provider documentation; these are periodically updated to reflect the latest data available from providers.

## How it works

The user interface has a series of input controls on the left-hand side and a stacked-bar chart in the center of the screen. The stacked-bars show the CPU utilization of the corresponding VM (in pink) and the amount of "unutilized" CPU (in green). Information about the VM, such as number of vCPUs, RAM and cost, is printed across each bar corresponding to that VM. When the user changes the input controls the back-end is queried and VMs with the lowest cost that satisfy the input control constraints will be rendered on the stacked-bar plot.

The key feature of the tool is that the CPU utilization of each VM depicted on stacked-bars chart is representative of the "same" workload being applied to each VM. For example, if the CPU utilization  (pink bar) is 50% for VM_1 and 25% for another VM_2, then the conclusion is that when the _same workload_ is applied to VM_1 and VM_2, the corresponding CPU utilizations will be 50% and 25% respectively. This underlying property lets us compare different VMs' CPU characteristics.

<p align="center">
<b>Fig.1: The BigBitBus Cloud Provider and VM Optimizer Tool User Interface </b><br>
 <img src="/assets/post8/optimizer.png">
</p>

### Input Controls on the User Interface

The user can query the performance and catalogue data we have collected by changing the settings on the left-side panels. The different settings are explained below:

#### Cloud Provider and Baseline Machine

Choose the baseline virtual machine. The cloud provider dropdown is used to filter available VMs by provider; the baseline machine can then be selected from the presented choices. We are constantly working to increase the size of this catalog. The top-most bar on the chart always corresponds to the baseline VM.

#### CPU Utilization

There are two sliders in this box. The CPU utilization slider represents the CPU utilization of the baseline VM. For example, if your monitoring dashboards indicate that the baseline VM has a peak CPU utilization of 70% then you should set this slider to 70%.

The second slider is to set the maximum CPU utilization of the target (alternatives to the baseline) VMs  that will be returned by the tool. For example, if you want that the CPU utilization of the target VMs should not exceed 50% then this slider can be set to 50%. If you are willing to push the CPU utilization to a higher number, set this slider to a high percentage and the tool will find smaller (and usually cheaper) VMs that run "hotter" when servicing the applied workload.

#### Target Clouds

The Target clouds check-boxes let you select which cloud providers' VMs are returned by the tool. For example if you only want to consider AWS VMs then select the AWS check-box and unselect all other cloud provider check-boxes.

#### Refine Search

The minimum and maximum number (amount) of vCPUs (RAM) can be constrained using these minimum/maximum range sliders. Only target machines which satisfy these constraints will be displayed.

## Use-cases

We illustrate possible uses of the tool through a few use-cases.

### Use-case 1: Switching a Cloud Provider

A user who wishes to research different cloud providers and switch the baseline VM running in a cloud provider to another cloud provider can use the tool to find the performance and cost of analogous VMs on different cloud providers. Select the baseline VM and choose target clouds, along with any CPU utilization or vCPU number/RAM constraints and the tool will return up to 9 target VMs that satisfy all the requested constraints at the lowest cost.

### Use-case 2: Lowering costs by down-sizing a VM

If a user finds that a VM is idling (low CPU utilization) then s/he can select this VM type as the baseline VM and set the CPU utilization to the low number. Then, from the options displayed in the chart, a smaller VM which is better utilized for the same workload can be selected. This is a great way to reduce cloud spend.

### Use-case 3: Choosing a VM with greater CPU headroom

Suppose a user finds that her/his VMs are running hot (high CPU utilization) then the tool can help find appropriate VMs that run cooler. Instead of guessing and switching to a much bigger VM (ending up with unnecessarily low utilization), the user can fix the "maximum CPU utilization" slider to the desired peak "hotness" and the tool will return the lowest cost VMs that satisfy the constraints.

## FAQs

* _Why did the tool stop responding after a while?_ We throttle requests to protect our servers. If you hit the throttle limit, please wait till an hour has passed before using the tool.

* _Why are other cloud providers not included in the comparison?_
We are working toward integrating more providers' data into our system; please check back as we expand our provider coverage. If you represent a cloud provider then please contact us so we can work on accelerating on-boarding your offerings into our tools.

* _Why don't I see data for the entire catalogue of the providers?_
We currently focus on VMs with 16 or fewer vCPUs (since these comprise the vast majority of deployed VMs); we have also excluded high memory or storage-optimized VMs (since all our testing is CPU based currently); please check back as we expand our VM coverage.

* _How accurate is the tool?_
Generic CPU benchmarks, such as the ones that form the basis of this tool, are rarely representative of actual production workloads' performance. The tool's data is a basis for comparing different VMs and gives us a "rule-of-thumb" or "back-of-envelope" comparison between different VMs to quickly whittle down the myriad VM choices across cloud providers. We encourage users to investigate short-listed VMs thoroughly against their custom workloads with their custom testing to switch VMs and providers with confidence.

* _I found an inconsistency/bug in the tool. How to report it/get it fixed?_
Fantastic! The tool is new and in beta testing, please help us by emailing any bugs, ideas, comments, or concerns to _contact@bigbitbus.com_.

* _Which cloud provider datacenters/regions were used in our testing?_
We primarily used eastern US cloud regions to perform all performance testing; cost data is also limited to this region. Giving users the ability to select specific data-centers in different cloud provider regions is on our product road map.

* _What prices are shown by the tool?_
We show retail costs (no discounts). We are aware that cloud providers offer sustained usage discounts, volume discounts, negotiated customer-specific discounts and other promotions to customers. Allowing users to apply such discounts to the cost numbers shown in the tool is on our product road map.

* _Does the tool compare other characteristics like IO and network latency?_
This tool compares VMs on the basis of CPU utilization. Building analogous tools for IO and network comparisons is on our product road map.

* _What is the [VM Comparer](https://tools.bigbitbus.com/comparer/) tool link on the top of the page?_ We are working on another tool to compare 2 VMs between different cloud providers. This tool is still in alpha as we collect better data. Feel free to give it a spin and please help us by emailing any bugs, ideas, comments, or concerns to _contact@bigbitbus.com_.


Back to the [VM and cloud provider optimization tool](https://tools.bigbitbus.com/optimizer/)

_BigBitBus is on a mission to bring greater transparency in public cloud and managed big data and analytics services._