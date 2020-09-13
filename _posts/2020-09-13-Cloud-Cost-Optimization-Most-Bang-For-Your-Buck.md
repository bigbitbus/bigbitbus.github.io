---
layout:     post
title:      Cloud Cost Optimization - Where to Invest Your Efforts 
published:  true
---

*Cloud users want to optimize their public cloud bills. A multi-pronged approach will yield the maximum benefits but some methods give you more bang-for-your-buck than others. We list the methods in decreasing order of impact so you can focus on the most impactful ones.*

BigBitBus helps cloud users reduce cloud computing costs. We have spent thousands of hours looking at different application stacks across small and large organizations and learnt where to look for inefficiencies. Here is the distilled zen of that experience. Presented in decreasing order of impact, along with a percentage of relative importance in our opinion.

*Code Optimization - 30%*

If you have the luxury of actually owing the code that runs in your cloud infrastructure, or if you run open-source software, then you should first ask your engineers to comb through the code to look for efficiencies. As horizontally scalable architectures (running multiple copies of the same software for scalability and high-availability) becomes the norm, even a small change can yield huge gains.

Here are some examples to inspire you to go down this path.
  - A developer removed unnecessary logging lines from the codebase and reduced the managed logging bill by several thousand dollars per month.
  - The QA engineer tested and upgraded to the latest version of an open-source database connector library which reduced the number of connections the application was making to the cloud database, enabling the organization to reduce the database size and save a 5-digit amount.
  - A developer reduced the polling interval on a datasource from 5 seconds to 20 seconds, reducing bandwidth costs on the data source server 4-fold.


*Embracing New architectures - 30%*

If your team has the ability to evolve the architecture of your applications then it is well worth your while to have engineers periodically scan open-source and managed cloud services' landscape to modernize your applications. Re-plumbing applications is expensive, but the return on investment can be spectacular. There is incredible and rapid innovation happening out there, often driven by Internet-scale  use-cases that relentlessly optimize costs. Available to anyone who invests in upgrading their application stacks. 

Here are some examples we have come across recently:

  - A mid-size e-commerce website moved from using virtual machines to Kubernetes. Afterwards, they were able to leverage open-source monitoring (Grafana+Prometheus) instead of a vendor monitoring solution as well as use pre-built database infrastructure-as-code (helm charts) to power their dev and qa environments instead of using managed cloud databases. Moving to Kubernetes also improved developer productivity due to better devops practices, faster deployments, and easy roll-backs for operations. We estimated they will save at least 60% over a 4-year period.
  - A fintech company was struggling with sharing environments for its distributed developer team. Spinning up a new environment per team was costing them a lot of money. They evolved their development environment architecture by logically partitioning the development environment using namespaces and service mesh for traffic isolation helped them cut their cloud bill by a huge amount.
 
 *Rightsizing Workloads - 20%*

 Lets face it, many capacity plans are not accurate and applications often land on over-provisioned infrastructure. This is particularly the case if the application is bursty with weekend or seasonal lulls for example. Rightsizing workloads means reducing VM and database sizes to fit requirements. Offcourse, bursty application requirements can change every minute, so auto-scaling infrastructure to match demand is an important aspect of rightsizing solutions.

 Here are some examples of auto-scaling done right:

   - An education technology company introduced autoscaling of their application servers to track weekend and school holiday periods; it directly saved them thousands of dollars per quarter.
   - A software development company used to spin up development environments with a "small" sized managed relational database. They didn't know that a smaller "nano" size database was recently released by their cloud provider. Switching to the "nano" size for development helped them reduce their cloud bill.

The above three approaches - Code optimization (30%), new architectures (30%) and rightsizing workloads (20%) together form the big 80% of the "where" to look for cloud cost control. There are some other approaches (the last 20%) where your-mileage-will-vary  and some come with their own undesirable side-effects:

*Negotiating vendor discounts (5%)*:

Lets face it, big cloud providers have 1000s of enterprise users so no matter how big you are, there is only so much weight you carry when you ask your provider for a big discount. They may be amenable to giving you a time-limited pot of "cloud credits" to entice you to switch providers or to lock you into their product, but in the long term, you won't come out on top.

*Process, controls, locking-down(5%)*:

Locking down cloud access across the developers of your organization in the hope this will control costs is a fallacy. Sure, you can set an upper bound on what each developer can consume in order to prevent fat-finger cloud errors, but locking down your cloud stiffles innovation, promotes shadow IT (for example developers using personal cloud accounts for development), and erodes trust. Remember, developers are probably more expensive than your cloud bill!


*BigBitBus is on a mission to bring greater transparency in public cloud and managed big data and analytics services. Talk to us about how you can architect your cloud IT assets to maximize your returns on investment.*