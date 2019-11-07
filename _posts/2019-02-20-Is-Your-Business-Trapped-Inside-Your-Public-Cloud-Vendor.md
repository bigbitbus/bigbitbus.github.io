---
layout: post
title: Is Your Business Trapped Inside Your Public Cloud Vendor?
---

It is easy to get locked into a specific cloud vendor. Learn how to keep your business free from cloud vendor lock-in.

<p align="center">
 <img src="/assets/post10/CSIRO_ScienceImage_1766_Venus_Fly_Trap.jpg">
 <small> Image courtesy - <a href="https://commons.wikimedia.org/wiki/File:CSIRO_ScienceImage_1766_Venus_Fly_Trap.jpg"> CSIRO Science Image </a> </small>
 </p>

 I am sorry to break it to you, but it is easier than ever to get locked into a public cloud or big data vendor. Lets see why cloud vendor lock-in is bad for your business, and then look at IT architecture patterns to mitigate the lock-in risk in your public cloud journey.

## Why is Cloud Vendor Lock-In a Bad Thing?

Public cloud providers are extremely innovative and forward-thinking, but do not confuse open-to-innovation with openness. 100s of closed-source databases, big-data systems, AI frameworks, IoT platforms, and SaaS services are launched by cloud providers every year. Every time you adopt a closed-source service the lock-in noose tightens. You are trading away your agility to migrate to a different cloud provider for the convenience and cost-saving of not building skills in your team to run the open-source counterpart of the managed service. This will come back to haunt you in the future:
   1. *Business Agility*. Your business may need a cloud-provider migration. For example, if you are a startup getting acquired by a company that uses a different cloud provider, then the valuation of your IT assets will suffer if your processes, code and data assets are too sticky to your chosen cloud provider.
   2. **Leverage**. You have no leverage when you can’t migrate off a cloud provider. When it is time to renew your contract with the cloud provider their sales negotiation team will know you cannot move to the competition.
   3. **Independence**. Your business is hamstrung by the rate and direction of innovation of your cloud provider. Although cloud providers are highly innovative, their choice of direction will control your choice of IT technologies and may force you to choose their vision of what IT should look like.
   4. **Reputational risk**. Your business is taking on reputational risk when you can’t migrate off a cloud provider. Suppose your cloud provider suffers a serious data breach or falls out of favor with a jurisdiction's government and your business needs to migrate off that cloud provider. How would you meet this challenge in a timely manner if you have not taken steps to avoid cloud vendor lock-in?
   5. **Continuity risk**. Your business will suffer if your cloud provider shuts down or sells the business. I know, how could I even contemplate Azure or AWS or GCP shutting down? But take a step back and think about all the products that came out of these companies and were pulled from the market. Remember blockbuster failures like Amazon’s Fire Phone, Microsoft’s Windows Phone, or Google Wave/Plus, for example? There will be cloud provider consolidation, and clouds will cease to exist.
   6. **Hiring risk**. Your business may find it impossible to hire skills in the future. If you have to stick with your cloud provider forever then finding skilled engineers may become challenging if your cloud provider becomes a niche player in the future.

I hope you are convinced that there are valid business reasons to consider the “my-business-needs-to-migrate-off-a-cloud-provider” scenario. Let us look at ways in which you can reduce stickiness to your public cloud provider and retain the migration option.

## Architect Against Cloud Provider Stickiness

   1. **Minimize data stickiness**. When you use closed-source databases and big data systems (e.g. Google Bigquery or AWS Redshift) make sure you regularly backup the data and schema metadata into files (on your cloud provider’s object store for example). This is an often overlooked aspect - after all the cloud provider is responsible for managed big-data service backups - but you will not have access to those backups when you migrate off the cloud provider. Making sure you have well documented documented data+metadata dumps will enable a future migration.
   2. **Favor open protocols, tools and APIs**. Choose open-standard APIs when possible. Whenever your developers have to choose between APIs (e.g. a messaging queue client or a database adapter) ask if the end-point (server) can be substituted by one outside of the cloud provider. If the answer is no, then carefully consider other options.
   3. **Clean, well defined interfaces**. If you have to use a cloud-provider-specific service, ask your developers to build a clean interface (a separate cloud-module for example) and document all the interaction points between your code and the cloud provider. Future developers who will lead your cloud migration will thank you for this some day.
   4. **Avoid provider-specific cloud orchestration tools**. (e.g. AWS Cloud formation or Azure ARM) and instead use API-driven open-source orchestration systems like Terraform when possible. Although you still end up writing provider-specific orchestration code your developers can be clever about separating cloud-agnostic orchestration code into separate modules. This will minimize the orchestration code re-write when you migrate to another cloud provider.
   5. **Avoid cloud provider SaaS tools**. For example, discourage your developers from using a cloud-provider’s custom CI/CD and SCM tools and instead rely on industry standards like Jenkins/Git*/Atlassian, etc. I know it is more work dealing with extra vendors as compared to your one cloud provider but process tools (like your devops pipelines) are among the hardest to migrate off of.
   6. **Choose third-party operational tools**. Depending on how important cloud migration is, you may want to adopt third-party cloud-agnostic monitoring and logging tools instead of using native cloud-provider services.
   7. **Talk to the competition**. Call up a competitive cloud provider occasionally and have your IT architects and developers discuss how your business could migrate to their cloud. Almost all migration stickiness issues you fear have been dealt with earlier!

*

_Sachin Agarwal is a computer systems researcher and the founder of BigBitBus._

_BigBitBus is on a mission to bring greater transparency in public cloud and managed big data and analytics services. Talk to us about how you can architect your cloud IT assets to minimize public-cloud stickiness and lock-in._