---
layout: post
title: Microsoft Azure Inter-Region Network Latency
---

_Azure offers greater geographical reach (more regions) than other public cloud providers; enterprises looking to build highly-available fail over IT architectures, especially outside the western world, will benefit from the low latencies between their relatively nearby regions_.

Since our last article on [Amazon AWS and Google Cloud ping RTT latencies]({{ site.baseurl }}{% link _posts/2018-05-07-What-Is-Your-Ping-AWS-And-Google-Cloud.md %}), we've received many queries asking about similar numbers for the Microsoft Azure cloud. ICMP pinging between Azure regions is not possible because Azure blocks all outgoing and incoming ICMP traffic. We instead used the [_paping_ tool](https://code.google.com/archive/p/paping/) to measure the TCP connection time between 23 different Azure regions. Note that this metric is different from the ICMP RTT times measured for Google Cloud and Amazon AWS in our previous article.

The list of Azure regions is available via this Azure cli command
```
az account list-locations | grep name
```
While 30 regions are listed by the above command, our Azure subscription could only provision in 23 regions (with the other 7 regions erring with codes like "LocationNotAvailableForResourceGroup" and "SkuNotAvailable" - we are happy to get inputs from any Azure experts on what the issue might be).

We spun up 23 Linux VMs (one per region) and ran the _paping_ command between each pair of VMs. The heatmap below shows the TCP connection times between the VMs.

Here are the results:

<p align="center">
  <b>Fig.1: Azure regions' TCP Connection Time </b><br>
  <img src="/assets/post3/BigBitBus_azuretcpconnect.png">
</p>

The darker the shade of the heatmap entry, the better (lesser) the TCP connection time. The absolute values of the connection times are also noted on the heatmap. The connection time from a region to all other regions can be read off the heatmap by looking at its row and and the corresponding destination region column. The heatmap reveals that each Azure region has at least one neighboring region with very low TCP connection latency (except the Brazil South region). This makes Azure a desirable destination for enterprises looking to build highly-available IT solutions across different geographies. There are no "orphan" regions as was the case with Google Cloud and Amazon AWS as seen in our [previous post]({{ site.baseurl }}{% link _posts/2018-05-07-What-Is-Your-Ping-AWS-And-Google-Cloud.md %}). We should note here that the number of regions is not indicative of the total size of the cloud providers - perhaps another cloud provider has many more hypervisors per region. We as cloud users can never know what those numbers are (going back to our observation of [lack of transparency]({{ site.baseurl }}{% link _posts/2018-04-20-Are-Similar-VM-T-shirt-Sizes-Across-Cloud-Providers-Comparable-in-Performance.md %}) on the part of public cloud providers).

We do have a hypothesis on why Microsoft chose to build out so many regions: Microsoft's flagship product - Office 365 - is a key use case for Azure data centers. Given its latency-sensitive nature, Microsoft may have decided to put down data-centers in many more geographies. Even if the demand for the Azure product is not strong in a particular region, local Office 365 subscribers will appreciate snappier Office performance!


* 

_Sachin Agarwal is a computer systems researcher and the founder of BigBitBus._

_BigBitBus is on a mission to bring greater transparency in public cloud and managed big data and analytics services._
