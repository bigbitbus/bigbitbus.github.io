---
layout: post
title: Persistent Cloud Disk Comparison Tool
---

_In this article we describe the BigBitBus persistent cloud disk comparison tool, available [here](https://tools.bigbitbus.com/iocomparer/)._

The persistent cloud disk comparison tool can compare block device performance (IOPs, throughput) under different workloads and across different public cloud providers. 

## How it works

The user interface has a series of input controls on the left-hand side and a table comparing block devices 1 & 2 is in the center of the page. You may choose the type of persistent disk service and the type of workload applied. The following storage services have been profiled until now:

| [Amazon AWS persistent disks](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html)      | [Google Cloud Platform persistent disks](https://cloud.google.com/compute/docs/disks/)    | [Microsoft Azure persistent disks](https://azure.microsoft.com/en-us/pricing/details/managed-disks/) |
| ------------- | ------------- | ------------- |
| st1      | pd-ssd |  Standard_LRS |
| sc1      | pd-standard | Premium_LRS (caching enabled)
| gp2 | | |
| io1 (50 IOPs/GB) | | |
| standard | |  |


The figure below shows a screenshot of the tool in action.

<p align="center">
<b>Fig.1: The BigBitBus Persistent Cloud Disk Comparison Tool showing a comparison between AWS io1 and GCP pd-ssd storage for random small block read-heavy workloads.  </b><br>
 <img src="/assets/post9/iocomparer.png">
</p>

## Test Description

We used [flexible IO - fio](https://github.com/axboe/fio) to perform large and small block read and write tests on 500GB raw block storage devices (without any filesystem) attached to 8-vCPU VMs. The tables below summarize what was tested. When we say a test is read heavy, we mean that 80% of all IO requests were reads and 20% are writes. In our tests reads and writes happen concurrently, as would be the case in real-life. By small block we mean that IO blocks read/written from/to disk were 4, 8, 16 or 32 kB with equal probability; by large block we mean that the IO blocks were 256kB, 512k, 1MB or 4MB in size. We ensured that memory caching effects were minimized by creating test files equal to the size of system RAM and full of random data.

| Sl. no        | Test type           | Notes  |
| ------------- |:-------------:| -----:|
| 1      | random small block read heavy, [fio job file](https://github.com/bigbitbus/fio-formula/blob/test_diskchecker/fio/files/Rnd-SBRH.job) | 80:20 read/write ratio; block sizes are uniformly distributed in {4k, 8k, 16k, 32k}. A relational database with more reads than writes, a blog server running off a MySQL database. |
| 2      | random small block write heavy, [fio job file](https://github.com/bigbitbus/fio-formula/blob/test_diskchecker/fio/files/Rnd-SBWH.job)      |  20:80 read/write  block sizes are uniformly distributed in {256k, 512k, 1M, 4M}. A data sink server where sensors send small data packets that are stored in a database or as small files |
| 3      | random large block read heavy, [fio job file](https://github.com/bigbitbus/fio-formula/blob/test_diskchecker/fio/files/Rnd-LBRH.job)     |    80:20 read/write ratio; block sizes are uniformly distributed in {256k, 512k, 1M, 4M}. Hadoop or Cassandra data nodes  |
| 4      | random large block write heavy, [fio job file](https://github.com/bigbitbus/fio-formula/blob/test_diskchecker/fio/files/Rnd-LBWH.job) | 20:80 read/write  block sizes are uniformly distributed in {256k, 512k, 1M, 4M}. Hadoop or Cassandra data nodes |
| 5      | sequential large block read heavy, [fio job file](https://github.com/bigbitbus/fio-formula/blob/test_diskchecker/fio/files/Seq-LBRH.job)      |   80:20 read/write ratio; block sizes are uniformly distributed in {256k, 512k, 1M, 4M}. Hadoop or Cassandra data nodes |
| 6      | sequential large block write heavy, [fio job file](https://github.com/bigbitbus/fio-formula/blob/test_diskchecker/fio/files/Rnd-LBWH.job)     |  20:80 read/write  block sizes are uniformly distributed in {4k, 8k, 16k, 32k}. Hadoop or Cassandra data nodes |

Our aim is not to come up with the largest IOPs and throughput or the lowest latency number possible for block storage devices. We want to emulate pseudo-realistic workloads for measuring raw block device performance for comparison purposes. After you have narrowed down on the block storage devices that may fit your application needs via our comparison tool we encourage you to run application specific tests, with the corresponding cache and file system optimizations, in order to get the real performance numbers for your use case.

An important aspect of any storage test is the queue depth set by the application. The queue depth is the maximum number of un-acknowledged or in-flight IOs. In our tool the queue depth was set to *4*. High queue depth numbers can result in higher IO throughput (MB/s), at the expense of higher latency (ms).

The comparison tool is simple to use; simply choose the storage type and the type of workload for two different block devices and the corresponding performance numbers will be displayed in a table.

## FAQs

* _Why are other cloud providers/disk types not included in the comparison?_
We are working toward integrating more providers' data into our system; please check back as we expand our provider coverage or contact us if you would like a specific cloud provider/disk tested and compared. If you represent a cloud provider then please contact us so we can work on accelerating on-boarding your offerings into our tools.


* _How accurate is the tool?_
Generic IO benchmarks, such as the ones that form the basis of this tool, are rarely representative of actual production workloads' performance. The tool's data is a basis for comparing different storage types and gives us a "rule-of-thumb" or "back-of-envelope" comparison between different persistent block IO types. We encourage users to investigate short-listed persistent disks thoroughly against their custom workloads with their custom testing for validation.

* _I found an inconsistency/bug in the tool. How to report it/get it fixed?_
Fantastic! The tool is new and in beta testing, please help us by emailing any bugs, ideas, comments, or concerns to _contact@bigbitbus.com_.

* _Which cloud provider datacenters/regions were used in our testing?_
We primarily used eastern US cloud regions to perform all performance testing. Giving users the ability to select specific data-centers in different cloud provider regions is on our product road map.


Back to the [BigBitBus persistent disk IO comparison tool](https://tools.bigbitbus.com/iocomparer/)

_BigBitBus is on a mission to bring greater transparency in public cloud and managed big data and analytics services._