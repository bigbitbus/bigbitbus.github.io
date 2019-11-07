---
layout: post
title: What is your ping, Google Cloud and Amazon AWS?
---

_Public cloud providers have built data-centers - regions - around the world. In this post we'll focus on the ping round-trip-time (RTT) latency between VMs spawned in these data centers._

Short RTTs are sweet. End-users appreciate the increased responsiveness of short RTT services, distributed systems such as geo-distributed key-value stores deliver higher throughputs on low latency networks (read/write operations), and some multi-player games are not even feasible until the RTT between the client and the server is below a (low) threshold. A low RTT is also mandatory for many high-availability (HA) architecture patterns (e.g. [hot standbys](https://en.wikipedia.org/wiki/Hot_spare)) and to ensure minimum [recovery point objectives](https://whatis.techtarget.com/definition/recovery-point-objective-RPO).

RTT is correlated to the physical distance between the communicating hosts (speed of light) and to the quality of the network (e.g. number of hops, packet switching quality, network congestion, etc.).  Cloud providers have setup dozens of data-centers around the world and connected them up with high-quality links to beat the problem of high inter-continental RTTs for customers in different geographical regions. 

In this article we will analyze the inter-region data-center latency of Amazon AWS and Google Cloud. We are going to answer these questions:

1. What are the average ping RTTs between VMs in different datacenters of each cloud provider?
2. Which subsets of regions have "low RTT" neighbors that can be used for HA setups?
3. What are the RTTs between Google Cloud and AWS data centers?
4. What does a mtr traceroute between VMs in different regions look like?


## Setup

Fig.1 shows the current (May 2018) locations of [Google](https://cloud.google.com/about/locations/) and [Amazon](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html) data-centers around the world. There are multiple regions of each provider in North America, Europe and Asia and single regions in South America and Australia.

<p align="center">
  <b>Fig.1: Google Cloud and AWS Regions around the world </b><br>
  <img src="/assets/post2/BigBitBus.com.regions.png">
</p>

We created virtual machines in a zone in each of the regions shown in Fig.1 and then ran [ping tests](https://en.wikipedia.org/wiki/Ping_(networking_utility)) from each VM to every other VM. Thirty pings were exchanged between each VM pair, and the data presented below is the average of those 30 pings. We also used the [mtr](https://en.wikipedia.org/wiki/MTR_(software)) network diagnostics tool to try and discern information about the underlying network routes between different cloud regions.

## RTT Heatmaps
Fig.2 and Fig.3 show the ping RTTs (in ms) between different Google cloud and Amazon AWS regions respectively. The names of the regions are shown on the x- and y-axes. The darker the shade of the heatmap entry, the better (lesser) the RTT. The absolute values of the RTTs are also noted on the heatmap. The ping RTT from a region to all other regions can be read off the heatmap by looking at its row and and the corresponding destination region column.

Unsurprisingly, geographically close regions have the least RTT between them. For example, european data-centers have very low RTTs between them. These would be perfect candidates to setup say, a distributed data-store or a highly-available hot-standby database.

Asian regions have significantly higher RTTs between themselves than their European and North American counterparts. Perhaps the underlying infrastructure needs to be improved to bring them at par with their western counterparts.

<p align="center">
  <b>Fig.2: Google Cloud Inter-regional ping latency </b><br>
  <img src="/assets/post2/BigBitBus.com_gceping.png">
</p>

Worse, there are also regions that have extremely high RTTs to _any_ other region. For example, cloud regions in India - gce-asia-south1 and ec2aws-ap-south-1 (Mumbai) - and in Brazil - gce-sa-east1 and ec2aws-sa-east-1 (Sao Paulo) - respectively. We term these RTT orphans - the RTTs to _all_ other regions are too high. That would be problematic for many high availability setups. We hope that more regions are built near these orphan data centers to give their users netter HA setup options.

<p align="center">
  <b>Fig.3: Amazon AWS Inter-regional ping latency </b><br>
  <img src="/assets/post2/BigBitBus.com_awsping.png">
</p>

We also measured the inter-cloud-provider ping RTTs between all the 29 data centers (AWS and Google) considered in this analysis. The heat map for these measurements is presented in Fig.4. Interestingly the RTTs between some subsets of a mixture of AWS and Google regions in the same geographical areas are extremely low (see the european data centers in Fig.4 for example). A user looking for extremely high-levels of availability may want to consider using both the cloud providers to get the extra redundancy (we think that Google cloud and AWS data centers share very little!).

<p align="center">
  <b>Fig.4: Amazon AWS + Google Cloud Inter-regional ping latency </b><br>
  <img src="/assets/post2/BigBitBus.com_aws+gceping.png">
</p>

## MTR

We were also curious about how the underlying traceroute between VMs in different regions looks like. We present two mtr traces here corresponding to AWS and Google cloud from South Asia (Mumbai) to South America (Sao Paulo).

<p align="center">
  <b>Google cloud (Mumbai) to Google cloud (Sao Paulo) </b><br>
</p>
```
Start: Mon May  7 00:17:50 2018
HOST: gce-asia-south1-a-3                                            Loss%   Snt   Last   Avg  Best  Wrst StDev
  1. AS15169 37.121.199.35.bc.googleusercontent.com (35.199.121.37)   0.0%    10  387.4 387.5 387.3 388.2   0.0
```
Notice the above mtr run within Google's network shows a single [Autonomous system(AS)](https://en.wikipedia.org/wiki/Autonomous_system_(Internet)) and a single hop.

<p align="center">
  <b>Google cloud (Mumbai) to AWS (Sao Paulo) </b><br>
</p>

``` 
Start: Mon May  7 00:22:17 2018
HOST: gce-asia-south1-a-3                                                       Loss%   Snt   Last   Avg  Best  Wrst StDev
  1. AS15169 216.239.50.36                                                       0.0%    10  386.3 386.4 386.3 386.7   0.0
  2. AS15169 108.170.245.232                                                     0.0%    10  386.3 386.9 386.3 389.0   0.6
  3. AS???   52.95.216.154                                                       0.0%    10  387.4 388.1 387.3 395.1   2.4
  4. AS???   ???                                                                100.0    10    0.0   0.0   0.0   0.0   0.0
  5. AS???   ???                                                                100.0    10    0.0   0.0   0.0   0.0   0.0
  6. AS???   52.93.44.165                                                        0.0%    10  389.2 391.6 388.1 397.3   2.9
  7. AS16509 54.240.244.177                                                      0.0%    10  390.5 391.3 387.4 402.3   4.7
  8. AS16509 54.240.244.194                                                      0.0%    10  392.9 394.6 388.4 424.9  11.0
  9. AS16509 54.240.244.169                                                      0.0%    10  387.7 388.8 387.6 389.8   0.7
 10. AS16509 54.240.244.62                                                       0.0%    10  389.6 389.7 389.5 390.5   0.0
 11. AS???   ???                                                                100.0    10    0.0   0.0   0.0   0.0   0.0
 12. AS???   ???                                                                100.0    10    0.0   0.0   0.0   0.0   0.0
 13. AS???   ???                                                                100.0    10    0.0   0.0   0.0   0.0   0.0
 14. AS16509 ec2-52-67-105-127.sa-east-1.compute.amazonaws.com (52.67.105.127)   0.0%    10  388.6 388.7 388.6 388.7   0.0
 ```
The two autonomous systems in the mtr trace - AS15169 and AS16509 - correspond to the Google and Amazon AS respectively.

The ping RTT is almost identical in both the cases.

## Outlook

Cloud providers have gone to great lengths to solve the high latency issues of remote data centers by building dozens of global regions for their world-wide client base. Some of these regions have very low inter-region RTTs and can be harnessed to create cross-region highly available IT solutions. However, many regions outside the western world suffer from relatively poor connectivity to other regions. 

We have only looked inter-region ping RTTs in this article. Stay tuned as we dig more aspects of cloud provider networks such as client-to-server latencies, RTT variations (e.g. time of day) and throughput.

*

_We are always interested in including more cloud providers in our analysis. If you represent a cloud provider and would like to be included in our analyses please contact us._

_Sachin Agarwal is a computer systems researcher and the founder of BigBitBus._

_BigBitBus is on a mission to bring greater transparency in public cloud and managed big data and analytics services._