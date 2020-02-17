---
layout:     post
title:      Data  Driven  Decisions  behind  AWS  Reserved  Instance  Commitments
published:  true
---

Amazon AWS gives users the option to purchase virtual machines (VMs) on a pay-as-you-go on-demand basis or commit to 1 or 3 year “reservations” and get a discounted price. The longer the commitment, the higher the discount. We wanted to answer some questions about this  pricing mechanism:

  1. Do 1- and 3-year reservations offer users uniform discounts over on-demand prices - irrespective of the type of virtual machine? 
_Answer: No, the discounts vary significantly across different VM classes. We will highlight some virtual machine classes that offer deeper commitment discounts than others._

  2. Are 1- and 3-year (relative) reservation price differences equal, or are some VMs more deeply discounted than others - is AWS incentivizing users into buying longer reservations more aggressively for some VM service classes as compared to others?
_Answer: Amazon discounts 3-year reservations more on some classes of VMs. We will hypothesize what may be the reasons AWS wants to discount some VM types more than others._

  3. What strategies can you adopt while choosing the type of reservations on AWS?
_Answer: It depends on your situation, but we offer some guidance toward the end of this article._

There are 100s of AWS VM types across dozens of data-centers, for this analysis we focused on pricing data from the us-east-1 AWS location. We based our analysis on information available from the AWS EC2 pricing available online. VM service offerings and prices may change over time, all data presented here was collected in January 2020. 

Figs. 1 and 2 show columns of different VM classes offered by AWS in us-east-1 (for example, t3, c5n, p3 etc) for 1-year and 3-year reservations respectively. Each VM class may have different sizes of VMs, for example the t3 class of VMs offers t3.nano, t3.small and t3.xlarge VMs with varying number of CPU cores and memory sizes. The per-hour on-demand cost of the  highest priced VM  of each class is listed at the top of each column (highlighted in blue). There are 3 bars for each VM type -  the left two bars - the dark-green and blue bars show the non-convertible or standard reservation discount percentage and the convertible reservation discount percentage over the on-demand price respectively. The red bar shows the premium AWS charges for users to have the convertible option (the difference between the green and blue bars equals the red bar).


<p align="center">
 <img src="/assets/post11/1yrPrem.png">
 <b>Fig.1: One year reservation discount percentages over on-demand prices </b>
 </p>

Comparing VM classes between Fig.1 (1-year commitment) and Fig.2 (3 year commitment) makes it clear that the longer commitments net bigger discounts. No surprise here as AWS gets to plan capacity over a longer window, the same datacenter hardware can potentially be used over longer periods, and most importantly the user is “locked-in” into AWS for a longer commitment. AWS definitely wants users to commit to longer leases.


<p align="center">
 <img src="/assets/post11/3yrPrem.png">
 <b>Fig.2: Three year reservation discount percentages over on-demand prices </b>
 </p>

The more interesting aspect is the comparison between different VM classes. The premium for convertibility is significantly higher for d2, p3, and p3dn VM classes. Why? To understand this, we first need to understand what the AWS VM convertible option means

“You can exchange one or more Convertible Reserved Instances for another Convertible Reserved Instance with a different configuration, including instance family, operating system, and tenancy. There are no limits to how many times you perform an exchange, as long as the target Convertible Reserved Instance is of an equal or higher value than the Convertible Reserved Instances that you are exchanging.” - from AWS Documentation 

Convertible VMs still give AWS “user lock-in” - a user will not migrate off AWS unless s/he agrees to forfeit all the reserved sunk cost they incurred when they bought the reservation; but AWS loses accuracy on their multi-year capacity planning front when users convert their reservations  - for example, if a user converted their p3 instances (featuring custom Nvidia tesla core GPUs) into general purpose VMs then AWS is potentially left holding Nvidia GPUs that may sit unused.

Interestingly, the convertible premium spread for 1-year reservations  in Fig.1 is much lesser (~10% or lower across all VM classes)  than 3-year reservations of Fig. 2 where the spread can reach up to ~20%. 

From Fig. 2 (3-year reserved instances), we see some of the highest convertible premiums for these VM classes:

| VM instance class | Description                                                                                                                                                                                  |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| D2 (17.53%)       | D2 instances are designed for workloads that require high sequential read and write access to very large data sets, such as Hadoop distributed computing                                     |
| P3 (20.90%)       | P3 instances deliver high performance compute in the cloud with up to 8 NVIDIA® V100 Tensor Core GPUs and up to 100 Gbps of networking throughput for machine learning and HPC applications. |
| P3dn (18.65%)     | P3dn instances are specialized compute units designed to accelerate machine learning training and inferencing for large, deep neural networks.                                               |


And some of the lowest convertible premiums as well:

| VM instance class | bigbitbusgithubio_jekyll_1Description |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| M5n (5.89%)       | These are a great fit for databases, High Performance Computing, analytics, and caching fleets that can take advantage of improved network throughput and packet rate performance                                    |
| C5n (5.67%)       | C5n instances that can utilize up to 100 Gbps of network bandwidth. C5n instances offer significantly higher network performance across all instance sizes.     |
| X1, x1e (4.95%)   | Part of the Amazon EC2 Memory Optimized instance family, designed for running high-performance databases, in-memory workloads such as SAP HANA, and other memory intensive enterprise applications in the AWS Cloud. |


Special purpose VMs - for example, the P3 VMs that contain Nvidia machine learning hardware have high convertible premiums. On the other hand, more generic hardware like the C5n or M5n seems to have a lower premium. AWS probably finds it much harder to accurately track demand for specialized hardware and wants users to sign up for longer leases when possible on these classes of VMs. 

There is approximately a 20% difference between the 1- and 3-year discounts. Should you bite the bullet and go for the higher 3-year discounts? In our opinion, it depends on two factors:

 1. **Your Demand Forecast Accuracy:** Get your best architect or engineering team on the job to predict future usage - don’t leave it to the junior business analyst to model the what-if scenarios and collect data for making reserved pricing decisions. If your capacity planning is accurate and your business horizon is reasonably certain, then a 3-year reservation is definitely worth it. Even the 1-year reserved instances are well worth the exercise - it can easily shave off over 30% of your VM instance bill.

 2. **Age of the VM Service Type:** Always find out about how “old” a VM-type being offered is. If they are older generations then you should consider switching your VM class to a newer class before you buy reserved instances. Afterall, if you buy 3-year instances on a VM class that was released 3-4 years ago, then you will be stuck with running your applications on 6-7 year old server hardware later.

In addition to these two considerations it is important to consider convertible reservations; but first check the convertible premium for the reserved instances you are planning to use.

---

_Saksham Bhatnager is a Data Analyst._

_Sachin Agarwal is a computer systems researcher and the founder of BigBitBus._

_[BigBitBus](https://www.bigbitbus.com) is on a mission to bring greater transparency in public cloud and managed big data and analytics services. Talk to us about how you can architect your cloud IT assets to maximize your returns on investment._