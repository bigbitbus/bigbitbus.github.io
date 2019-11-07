---
layout: post
title: Public Cloud Object-stores Are Very Unequal
---
_For this article we compared the object-store performance of Amazon Web Services (S3), Google Cloud Storage and Microsoft Azure Blobs in locally redundant configurations (without geo-replication). We found very significant performance differences that can have a direct impact on user applications._

[Object or blob store](https://en.wikipedia.org/wiki/Object_storage) services on the cloud offer content addressable storage where users can save arbitrary files that can tbe accessed via a URL over HTTP(s) connections and simple CRUD semantics (GET to download, PUT to upload etc.). Object storage is convenient and cheap, and this has made it the storage back-end of choice for everything from small configuration files of less than a few kilobytes to huge VM images or backup archives. It is also the most common storage option for persisting raw data files used in big data analyses.

Lower object-store latency (time to upload and download files) is important in many use cases. For example, the time taken to download a backup copy of a database will be the dominant factor in the recovery time objective for disaster recovery planning. Big data applications such as Apache spark may seem sluggish if the back-end object-store hosting raw data has a high file-serving latency. There are many applications that repeated and frequently read and write small files to object stores (e.g. image thumbnails); these will benefit from lower latency small object performance.

Our key findings are:
1. Large blob downloads are significantly slower (up to 4x) in Azure as compared to Google cloud storage or AWS S3 large object downloads.
2. Small-sized Azure blobs have lower upload latency.
3. In general the (relatively newer) Canadian regions have lower latency for object store operations as compared to the older US east regions.

## Setup

We setup locally redundant object-store buckets for AWS-S3, Google cloud storage, and Azure blob storage in a cloud region and created one virtual machine (per provider) in the same cloud region. By "locally redundant" we mean that the objects were not geo-replicated to another region; we will be analyzing geo-replicated objects in another article.

<p align="center">
<b>Fig.1: Test Setup for locally-redundant object-store testing. We report the upload and download latency of the client putting/getting objects to/from the object-store. </b><br>
 <img src="/assets/post6/BigBitBus_objectstore_architecture.png">
</p>

A load tester virtual machine was loaded up with our custom-built open-source benchmarking program called [object bench](https://github.com/bigbitbus/objectbench) that can upload and download different sized randomly-generated files to the object-stores. This tool uses python SDKs from each of the providers (so the client implementation is strictly as per provider standards). The tool  was setup to serially upload and download different-sized randomly-generated files (ranging from 1kB to 100MB in size). We repeated the experiment 100 times and all our results are averaged over these 100 runs; we also show error-bars in our plots.

## Results

We measured latency as seen by an application which uploads and downloads objects from the object-store. We present results for a US east region and a Canadian region for each provider (the exact names differ across providers). By selecting two different regions for each provider we eliminated the possibility of a bad load testing VM client or a badly configured object store in a specific region. We also unearthed performance differences between the regions for the same cloud provider; users looking for the best performance on public cloud object-stores should carefully benchmark performance differences across regions before choosing a specific region. All cloud regions  _do not_ have the same performance.

### US Region
We chose _us-east-1_, _us-east1_ and _eastus_ regions for AWS, Google cloud and Azure respectively (collectively referred to as USEast in the below plots). The load testing VMs were spun up in one of the zones belonging to these regions for each cloud provider.

#### Small object sizes

Figs.2 and 3 show small object upload and download latencies in US East regions. The Azure blob store offers significantly lower upload latency as compared to AWS S3 or Google Cloud Storage. Its hard to say why the stark difference without knowing the implementation. We have a controversial hypothesis - perhaps uploads (writes) to the Azure blob store are cached in memory (to be persisted on disk later) and the acknowledgement sent immediately to the uploading client.

<p align="center">
<b>Fig.2: Small objects (up to 100kB) upload latency in US East </b><br>
 <img src="/assets/post6/BigBitBus_small_upload_USEast.png">
</p>

<p align="center">
<b>Fig.3: Small objects (up to 100kB) download latency in US East </b><br>
 <img src="/assets/post6/BigBitBus_small_download_USEast.png">
</p>

#### Large object sizes

Figs.4 and 5 show large object upload and download latencies in US East regions. The performance of all three object-stores is very similar for uploads. The strikingly slower Azure download is the highlight here (Fig. 5). We think this is a serious problem in Azure - especially for the backup/restore use-case. The data says that a 100MB object takes over 4 seconds to download from Azure blob-store, as compared to ~1 second in Google cloud storage. Downloading a 100GB backup set composed of 1000 such 100MB objects will take over 11 hours in Azure as compared to less than 3 hours in Google cloud. That is a huge hit on the recovery time objective for Azure users.

<p align="center">
<b>Fig.4: Large objects (1MB - 100MB) upload latency in US East</b><br>
 <img src="/assets/post6/BigBitBus_large_upload_USEast.png">
</p>

<p align="center">
<b>Fig.5: Large objects (1MB - 100MB) download latency in US East</b><br>
 <img src="/assets/post6/BigBitBus_large_download_USEast.png">
</p>

#### Object Deletion

Fig.6 shows the deletion latency for different-sized objects. The notable feature here is the consistency in the Google cloud (GCP) numbers.

<p align="center">
<b>Fig.6: Object deletion latency in US East</b><br>
 <img src="/assets/post6/BigBitBus_object_deletion_USEast.png">
</p>


### Canadian Region

We repeated all the above experiments on Canadian public cloud regions. Figs.7-11 show the corresponding Canadian region numbers. Notice the different Y-axis on some of these graphs; in general the latency numbers are lower in Canadian regions than US East regions. We hypothesize that this is because of the relative newness and lower utilization of the Canadian regions. The same superior small-object performance and dismal large-blob download performance of Azure blobs was seen in these results as well.

We chose _ca-central-1_, _northamerica-northeast1_ and _canadacentral_ regions for AWS, Google cloud and Azure respectively (collectively referred to as Canada in the below plots).

#### Small object sizes 

<p align="center">
<b>Fig.7: Small objects (up to 100kB) upload latency in Canada </b><br>
 <img src="/assets/post6/BigBitBus_small_upload_Canada.png">
</p>

<p align="center">
<b>Fig.8: Small objects (up to 100kB) download latency in Canada </b><br>
 <img src="/assets/post6/BigBitBus_small_download_Canada.png">
</p>


#### Large object sizes

<p align="center">
<b>Fig.9: Large objects (1MB - 100MB) upload latency in Canada</b><br>
 <img src="/assets/post6/BigBitBus_large_upload_Canada.png">
</p>

<p align="center">
<b>Fig.10: Large objects (1MB - 100MB) download latency in Canada</b><br>
 <img src="/assets/post6/BigBitBus_large_download_Canada.png">
</p>

#### Object Deletion
<p align="center">
<b>Fig.11: Object deletion latency in Canada </b><br>
 <img src="/assets/post6/BigBitBus_object_deletion_Canada.png">
</p>

## Outlook

The latency metrics reported in this article are critical for many user applications. Our results show a clear disadvantage when using the Azure blob store for large objects - operations like restoring backups, downloading large media files and VM images, etc. The Azure service wins for small object sizes - uploads were consistently faster than AWS S3 and Google cloud storage object stores. Object deletion time is important for applications that update, save and delete a large number of temporary objects. We were impressed by the consistency in the Google cloud deletion times as compared to other object stores.

Our aim was to capture performance differences due to different object-store implementations. Given the performance differences across the implementations we hope the engineering teams behind these services will tune and improve their systems to bring their systems at par with the best.

Stay tuned as we investigate geo-replicated object performance, cold-storage object stores and object metadata performance in this series.

_Sachin Agarwal is a computer systems researcher and the founder of BigBitBus._

_BigBitBus is on a mission to bring greater transparency in public cloud and managed big data and analytics services._
