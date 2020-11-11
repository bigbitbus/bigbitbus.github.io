---
title: BigBitBus Console Quickstart
heading: BigBitBus Console Quickstart
---

Welcome to the BigBitBus Console. The BigBitBus Console (B3Console) is a tool to compare different cloud provider services' attributes, their performance and costs. We currently offer service and on-demand/reserved pricing information for Amazon AWS, Azure Google Cloud GCP and Alibaba Cloud, primarily for compute services (VMs); we are rapidly expanding our catalog to include other cloud providers and service types.

Ready to do some quick back-of-envelope architecture and price planning? Or, curious to know if "the-other-cloud-provider" is more cost effective for your use-case? Follow along this walk-through and start making data-driven informed choices around public cloud providers.

**Application Walkthrough Steps**
- [Account Creation and Login](#account-creation-and-login)
- [Stacks](#stacks)
- [Creating New Stacks](#creating-new-stacks)
- [Stack Matching](#stack-matching)
- [Stack Optimization](#stack-optimization)
- [Service Comparison](#service-comparison)
- [Service List](#service-list)
- [Price List](#price-list)
- [Sharing List](#sharing)
- [Learn More](#learn-more)

Lets start with signing into the B3Console.

## Account Creation and Login
<a name="login"></a>

You can create an account by logging into BigBitBus using your Google, Gsuite or Github account. For this example lets use Google. Click on the "Google Login" button. The same credentials can be used to log into the console subsequently. 

![Using Google Social Login to log into the BigBitBus Console](google-login-small.gif)


<!-- <iframe width="560" height="315" src="https://www.youtube.com/embed/cbIXPA15Ewg" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe> -->



## Stacks
<a name="stacks"></a>
After you log in you will see a list of your "stacks". A few demo stacks are automatically created for new users. Stacks describe an IT application's infrastructure and may employ one or more kinds of cloud services. Identical services (e.g. a specific VM type in a given cloud provider location) are grouped as different "tiers". There can be one or more identical service types comprising a "tier". For example, the `bigbitbus_sample_bigdata_apache_spark_cluster` example has a master node tier with 3 `aws-c5.2xlarge` VMs and an application server tier comprised of 32 `aws-m5.large` VMs.

Its important to note that stacks model your infrastructure, they don't actually stand up any real infrastructure in a cloud provider! So you are free to create, update and experiment with any combination of services.

![The list of your stacks; clicking on detail lets you view the tiers and cost of your stack and its tier components.](stacks-small.gif)

<!-- <iframe width="560" height="315" src="https://www.youtube.com/embed/2axhGSL3LSE" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe> -->

Clicking on "detail" lets you view the tiers and cost of your stack and its tier components. 

## Creating New Stacks
<a name="create_new_stack"></a>
You can create your own stack by clicking on "Add stack". After specifying a name, description (optional) and metadata (optional), start adding tiers; service type and price tables are available to "populate" the forms by clicking the button `WITH SERVICES` at the top of the form.

Once you have successfully created a stack you can use it to find alternative configurations in other cloud providers.

Each tier in the stack is comprised of a service type. There may be multiple instances of the same service type within a tier. You can also add arbitrary key-value "service attributes" to each tier. For virtual machines, you can optionally add specific attributes to enter data needed for running the optimization workflow. If you don't specify these then all optimize requests will instead return attribute matching.

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

BigBitBus has collected performance data 1000s of virtual machine service types (ranging from 1-32 vCPUs) across different cloud providers. You can Use stack optimiztion by telling the system how much CPU and memory is currently utilized by the tier's services; the system then finds appropriate VMs in the target cloud provider that will satisfy your application stack's demand. If BigBitBus does not have performance data then it will fall back to match analogous service types based on attributes like the number of vCPUs or memory size.

![Optimize your stack: improve performance and/or lower costs](stack_optimization-small.gif)


## Service Comparison

You can also compare individual services with each other. After finding the service you wish to compare click on `Compare` and select a second service. A tabular comparison of the two services is presented.


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


## Sharing

Most page views inside the B3Console can be shared publicly via URL links and through LinkedIn and Twitter. A "Share" button is provided at the top right to quickly create publicly accessible "shares" of the view. Shares can be viewed by anyone with the URL and expire after a user-specified period. Note that shares are HTML files with sponsor links to support the service.

![Creating a Stack Share](share-stack.gif)

## Learn more

Visit our [website](https://www.bigbitbus.com/) to learn more.

