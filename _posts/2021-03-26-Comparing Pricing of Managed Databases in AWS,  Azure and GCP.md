---
layout:     post
title:      Comparing Pricing of Managed Relational Databases in AWS,  Azure and GCP
published:  true
---
In this post will go over the pricing of relational database services across the three major providers viz. Amazon Web Services (AWS), Microsoft Azure, and Google Cloud Platform (GCP). We will show how using a different provider can help you save money as well as how costs scale with database size. [B3Console](https://b3console.bigbitbus.com/login) can be your inbiased data-driven decision helper in this space!


## Relational Database Services in the B3Console

Relational database services are an integral part of applications. Providing database platform-as-a-services (PaaS) is a huge market opportunity as many organizations are looking to outsource database server management. Major cloud service providers like AWS, Azure and Google provide off the-shelf solutions to run a database in the cloud with ease. These services, though easy to use, often come with a hefty price tag. For example, even a modest 4-core instance running the open-source MySQL database engine can cost upwards of $6,000 a year and costs only increase as the application scales. If a user has a database-heavy IT footprint, then they can benefit from choosing  a cost-effective database PaaS provider.

In this article we will go over the pricing of relational database services across the three major providers viz. Amazon Web Services (AWS), Microsoft Azure, and Google Cloud Platform (GCP). We will show how using a different provider can help you save money as well as how costs scale as the database scales.


## A Bird’s Eye View

This section will present an overview of pricing of relational database services from the three major providers. The following charts look at the hourly On-Demand price for relational database services across virtual CPU core count. All machines run the MySQL database engine. Hence the only major difference is the provider. For this study we looked at the average price of instances across regions and grouped them by the vCPU core count.

We find that the cheapest provider across all vCPU sizes is Microsoft Azure, whereas Google and AWS have similar pricing up until 32 vCPU cores. Beyond this point AWS becomes more expensive. 

<p align="center">
<img src="/assets/post15/rates.png"/>
</p>


 Pricing also stratifies on various machine types. Some machine types are significantly more expensive than the others. For GCP and Azure we see that memory optimized machines are more expensive than generic machines. On the other hand AWS provides many machine types leading it to be more expensive than the other providers.



<p align="center">
<img src="/assets/post15/machine_types.png"/>
</p>

There are also differences in prices across regions, and these too can be significant. The chart below shows the prices of a small single core machine (db.m1.small) for AWS running the PostgreSQL. As can be seen prices can vary widely between regions. As shown below the rate can more than double depending on the region, where the service is deployed. 

<p align="center">
<img src="/assets/post15/regions.png"/>
</p>



## Comparing Providers

There are 10s of thousands of managed database machine type skus across providers. BigBitBus has collected detailed pricing and sizing data for all these options so you can make data-driven decisions about how to choose your cloud provider and right-size your database workloads.

<p align="center">
<img src="/assets/post15/screenshot_aze.png"/>
</p>

The B3Console allows you to gain insights into your IT infrastructure costs using our comparison tool. It allows you to take the relational database service on one provider and translates it to corresponding services on another. For example, we translate the m4 machine type from AWS to GCP and Azure using the B3Console’s matching feature. The m4 machine types are the general purpose machine types from Amazon that can be used for most workloads. In particular we match the db.m4.2xlarge machine with 8 vCPU cores and 32 GB of memory running the MySQL database engine. In case of GCP, the B3Console returns the db-n1-standard-8 service with 8 cores of vCPU and 30 GB of memory. Outside of the small difference in memory the machines are the same, but the difference in price is significant. The AWS service costs $1,022 a month compared to $789 of the GCP service. This is approximately 23% in savings and can save about $2,800 a year. The results from the GCP translation can be seen in the figure below:



<p align="center">
<img src="/assets/post15/screenshot.png"/>
</p>


The results are even more drastic for Azure. According to the [B3Console](https://b3console.bigbitbus.com/login) the corresponding service for the AWS’ db.m4.2xlarge service is Azure’s general purpose Gen 5 machine (encoded as db-general-g5-8-mysql) with 8 cores and 40 GB of memory. This machine costs about $520 a month, which is almost half the price of the AWS machine. This amounts to savings of $6,000 a year. So not only the Azure service has 8 GB more in memory it also costs about half as much. The results from the B3Console can be seen the screenshot below:


## The Bottom Line

After having seen  the comparison between various providers we get an idea about the diversity among the services provided under the same class of services. This reinforces the point that when it comes to making decisions about your cloud infrastructure, technical requirements cannot be the sole criterion. There needs to be a more rigorous cost-benefit analysis; this is the gap our product aims to fill. You can access the B3Console for free at: [https://b3console.bigbitbus.com/login](https://b3console.bigbitbus.com/login)