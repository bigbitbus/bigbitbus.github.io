---
layout: post
title: Cloud Catalog Search Tool
---

_In this article we describe the BigBitBus cloud catalog search tool, available [here](https://tools.bigbitbus.com/search/)._

We are very excited to release the cloud catalog search tool. The tool can search over 100,000 different sku's across Amazon AWS and Google GCP, spanning infrastructure, PaaS, big data and supporting network, storage and licensed software offerings. Although this information is published by cloud providers via APIs ([aws](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/using-ppslong.html), [gcp](https://cloud.google.com/billing/v1/how-tos/catalog-api)) that let you download voluminous JSON files, it has never been available for multiple cloud providers and in an easy searchable format. We hope it will make it easier for cloud users to understand the services on offer and improve transparency and comparison ease between cloud provider offerings.

## How it works

The search tool presents pretty-JSON data from the product information published by cloud providers (AWS and GCP currently); this information is frequently updated by AWS and Google cloud. The data is not vetted by BigBitBus and is presented almost untouched (save for some re-organizing of a few deeply nested JSON keys for easier handling). Skus and prices (if applicable) are also included in the search results.

The tool presents you with a simple search box; use it much like you would use any search engine. Words are case-insensitive; pre-pend a keyword with `--` (no spaces) if you want to avoid search results with that key word.

Example searches typed into the BigBitBus [search box](https://tools.bigbitbus.com/search/):

* aws amazonec2 t3.small tokyo suse `--`windows `--`sql
* bigquery streaming insert
* gcp coldline storage singapore
* kubernetes `--`gcp US-West-1
* kubernetes `--`aws

The search will return up to 8 results; if its not what you are looking for please play around with the key words.


Back to the [BigBitBus cloud catalog tool](https://tools.bigbitbus.com/search/)

_Disclaimer: For the latest and most authoritative information regarding cloud services, prices and availability please contact your cloud provider. While BigBitBus will frequently download and index cloud provider offerings, we cannot guarantee any information provided via the search tool._

_BigBitBus is on a mission to bring greater transparency in public cloud and managed big data and analytics services._