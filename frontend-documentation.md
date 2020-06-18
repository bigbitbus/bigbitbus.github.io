---
title: BigBitBus Console Quickstart
heading: BigBitBus Console Quickstart
---

Welcome to the BigBitBus Console. The BigBitBus Console is a free tool to compare different cloud provider services, and their relative costs. Users can quickly compute many what-if scenarios of different application sizing, different types of services - for example the size of VMs to build their application stack. They can choose to share these scenarios with their colleagues or on social media. We currently store service and pricing information for Amazon AWS, Azure Google Cloud GCP and Alibaba Cloud, primarily for compute services (VMs); we are rapidly expanding our catalog to include other cloud providers and service types.

Ready to do some quick back-of-envelope architecture and price planning? Or, curious to know if "that-other-cloud-provider" is more cost effective for your use-cases? Follow along this walkthrough and start making data-driven informed choices around your public cloud provider choices.


## Account Creation and Login
<a name="login"></a>

You can create an account by logging into BigBitBus using your Google account. Click on the "Google Login" button. The same credentials can be used to log into the console subsequently.

![Using Google Social Login to log into the BigBitBus Console](google-login-small.gif)

A few demo stacks are automatically created for new users.

<!-- <iframe width="560" height="315" src="https://www.youtube.com/embed/cbIXPA15Ewg" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe> -->



## Stacks
<a name="stacks"></a>
Stacks describe an IT application infrastructure that may employ one or more kind of cloud service. Identical services (e.g. a specific VM type in a given cloud provider location) are grouped as different "tiers". There can be one or more identical service types comprising a "tier". 

Its important to note that stacks model your infrastructure, they don't actually stand up any real infrastructure in a cloud provider!

![The list of your stacks; clicking on detail lets you view the tiers and cost of your stack and its tier components.](stacks-small.gif)

<!-- <iframe width="560" height="315" src="https://www.youtube.com/embed/2axhGSL3LSE" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe> -->

Clicking on "detail" lets you view the tiers and cost of your stack and its tier components. 

## Creating New Stacks
<a name="create_new_stack"></a>
You can create your own stack by clicking on "Add stack". After specifying a name, description (optional) and metadata (optional), we start adding tiers; service type and price tables are available to "populate" the forms.


For virtual machines, you can use these attributes to enter data needed for running the optimization workflow, above. If you don't specify these then all optimize requests will instead return attribute matching.

|  key | Example value  | Note |
|---|---|---|
| util  | 20  | Percentage of CPU utilization. If this is low for your application  then BigBitBus can use this to downsize VMs when trying to optimize your stack.  | 
| max_util  | 80  | Your comfort level of how hot you want to run your CPUs. BigBitBus will use this value in its performance models when it chooses alternatives.   |
| memsizemin  | 2  | In Gigabytes (GB), the minimum memory size you want your VMs to have, this value is optional  |
| memsizemax  | 8  | In Gigabytes (GB), the maximum memory size you want your VMs to have, this value is optional  |

![Optimize your stack: improve performance and/or lower costs](create_new_stack-small.gif)


<!-- <iframe width="560" height="315" src="https://www.youtube.com/embed/e1WxHlcMsGw" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe> -->



## Stack Matching
<a name="stack_matching"></a>

You can find analogous service types for each tier of your stack using the "matching-attributes" functionality. In the case of virtual machines we match the number of vCPUs and the memory (RAM). We are currently working on including non-VM service types and they will be matched based on other relavent attributes.


<!-- <iframe width="560" height="315" src="https://www.youtube.com/embed/Q_wFmeeCD5E" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
 -->
![Match your stack: find alternatives](stack_matching-small.gif)


## Stack Optimization
<a name="stack_optimization"></a>

BigBitBus has collected performance data 1000s of virtual machine service types (ranging from 1-32 vCPUs) across different cloud providers. You can Use stack optimiztion by telling the system how much CPU and memory is currently utilized by the tier's services; the system then finds appropriate VMs in the target cloud provider that will satisfy your application stack's demand. If BigBitBus does not have performance data then it will fall back to try to match analogous service types based on attributes like the number of vCPUs or memory size.

![Optimize your stack: improve performance and/or lower costs](stack_optimization-small.gif)




## Service List
<a name="service_list"></a>

BigBitBus maintains over 10,000 service types being offered by different cloud providers (AWS, Azure, GCP, Digital Ocean and Alibaba cloud currently) across dozens of their data centers. This list is available by clicking on "Services" on the top menu. Filters are provided to narrow down your searches.

There are also advanced options for creating your own "provider" and add service types to it.

![The list of cloud provider service types that can be used to compose tiers and stacks](servicelistfilter-small.gif)

<!-- <iframe width="560" height="315" src="https://www.youtube.com/embed/tjDhJ-Z36Rw" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe> -->

## Price List
<a name="price_list"></a>

Each service type can have multiple prices based on whether you are buying on-demand or reserving the service for a long period. You can look at the current prices stored by BigBitBus. Filters are provided to narrow down your searches.

There are also advanced options for creating your own price for any service type. In addition, you can specify blanket discounts for a certain provider. For example, if you have negotiated a flat 10% discount on your AWS bill then this can be applied across your stacks.

![The list of cloud provider service types that can be used to compose tiers and stacks](pricelistfilter-small.gif)

<!-- <iframe width="560" height="315" src="https://www.youtube.com/embed/XylrAw524mA" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe> -->


## Learn more

Visit our [website](https://www.bigbitbus.com/) to watch a more detailed demo and learn more.

