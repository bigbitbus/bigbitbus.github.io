---
title: BigBitBus API Documentation
author: BigBitBus Inc.
---
# The BigBitBus API Documentation<!-- omit in toc -->

Revision v1.01

Date 30/09/2019

License: Copyright BigBitBus Inc. All rights reserved.

<div style="page-break-after: always;"></div>

**Table of Contents**

- [BigBitBus API Version 1](#bigbitbus-api-version-1)
  - [1.1. Authorization](#11-authorization)
  - [1.2. Applications](#12-applications)
    - [1.2.1. Create an application](#121-create-an-application)
    - [1.2.2. Retrieving Applications](#122-retrieving-applications)
    - [1.2.3. Obtaining a report on an Application](#123-obtaining-a-report-on-an-application)
    - [1.2.4. Querying Applications](#124-querying-applications)
    - [1.2.5. Finding Application Alternatives on Another Provider](#125-finding-application-alternatives-on-another-provider)
    - [1.2.6. VM Optimization](#126-vm-optimization)
    - [1.2.7. Updating an Application](#127-updating-an-application)
    - [1.2.8. Deleting an Application](#128-deleting-an-application)
  - [1.3. Prices](#13-prices)
    - [1.3.1. Creating a new price for a service (POST)](#131-creating-a-new-price-for-a-service-post)
    - [1.3.2. Get price details about a pre-existing price (GET)](#132-get-price-details-about-a-pre-existing-price-get)
    - [1.3.3. Updating the costperunit for a pre-existing price (PUT)](#133-updating-the-costperunit-for-a-pre-existing-price-put)
    - [1.3.4. Deleting a price (DELETE)](#134-deleting-a-price-delete)
    - [1.3.5. Filtering Price data](#135-filtering-price-data)
      - [1.3.5.1. Getting all prices for a service type](#1351-getting-all-prices-for-a-service-type)
      - [1.3.5.2 Get the price of a ServiceType (specified by its service_pk) of a certain pricetype](#1352-get-the-price-of-a-servicetype-specified-by-its-service_pk-of-a-certain-pricetype)
      - [1.3.5.3. Getting all prices of a certain pricetype](#1353-getting-all-prices-of-a-certain-pricetype)
      - [1.3.5.4. Getting all prices](#1354-getting-all-prices)
  - [1.4. ServiceTypes](#14-servicetypes)
    - [1.4.1. Create a new service type (POST)](#141-create-a-new-service-type-post)
    - [1.4.2. Retrieve a service type (GET)](#142-retrieve-a-service-type-get)
    - [1.4.3. Deleting a service type (DELETE)](#143-deleting-a-service-type-delete)
    - [1.4.4. Finding service types](#144-finding-service-types)
      - [1.4.4.1. Service type keyword search](#1441-service-type-keyword-search)
      - [1.4.4.2. List all service types](#1442-list-all-service-types)

<div style="page-break-after: always;"></div>


## Concepts

Cloud providers offer service types to users, who in turn compose applications from one or more service types. For example, a user will create a nosql database, consisting of multiple VMs, spread across different data centers (cloud regions). BigBitBus API allows users to store information about such applications in its database. This information is mapped to the cloud pricing and performance information (about different service types on offer by cloud providers).

Users create applications in the database (using the API). They can then query different BigBitBus API endpoints to get pricing information about the application, which analogous service types can be used to build a similar application on another cloud provider, comparing costs and performance, etc. Before we get into specifics of the API, we define a few terms which will be used throughout the documentation.

## Definitions

- **Provider** A provider is either a public cloud provider, like AWS or Azure, or a in-house on-prem cloud.

- **ServiceType** A servicetype is an offering that a cloud provider offers. For example, AWS provides the "aws-t3.small" VM instance type.

- **Location** Location refers to cloud provider regions. For example, "us-east-2" and "ukwest" are regions where AWS and Azure provide services. ServiceTypes are unique to a location because cloud providers may not offer some servicetypes in some locations.

- **Application** An application describes a cluster or set of cloud services used together - for example, 30 virtual machines, spread across 3 data centers, that comprise a No-SQL database. An application is a set of one more more ServiceInstances. A ServiceInstance comprises of a ServiceType and arbitrary attributes that describe characteristics of how the ServiceType is used in the application. Here is an example of an Application, named "Enterprise_HR_SAAS_Application" that has two ServiceInstances. The first ServiceInstance uses the "aws-c5-xlarge" ServiceType and has several attributes - for example - the "quantity" is set to 8. The second ServiceInstance usese the "aws-c5-2xlarge" ServiceType and has the quantity value 3. You can add any arbitrary <attr_id,value> pair to applications. Some of these attributes (`util`, `max_util`, and `quantity`) have special meaning and are used by the BigBitBus comparison and recommendation engine.

```json
{
  "app_name": "Enterprise_HR_SAAS_Application",
  "services": [
    {
      "service_type": "aws-c5.xlarge",
      "location": "us-west-2",
      "price_type": "onDemandPrice",
      "provider": "aws",
      "attributes": [
        {
          "attr_id": "unit",
          "value": "instance"
        },
        {
          "attr_id": "util",
          "value": "40"
        },
        {
          "attr_id": "max_util",
          "value": "80"
        },
        {
          "attr_id": "quantity",
          "value": "8"
        }
      ]
    },
    {
      "service_type": "aws-c5.2xlarge",
      "location": "us-east-2",
      "price_type": "onDemandPrice",
      "provider": "aws",
      "attributes": [
        {
          "attr_id": "unit",
          "value": "instance"
        },
        {
          "attr_id": "quantity",
          "value": "3"
        }
      ]
    }
  ]
}
```

- **PriceType** Cloud providers offer ServiceTypes at different prices to different customers. Users have the ability to create a custom price for any ServiceType or choose among the standard prices offered by the cloud provider - these are usually "onDemandPrice", "reserved1yearPrice", and "reserved3yearPrice".

## Model

Next, lets visualize the data models being used under the hood.

![Applications and Service Instances](/assets/documentation/applications.png)

In Fig.1 a user defines an application, which may comprise of multiple service instances. Each service instance is composed of one or more service type instance (quantity), along with properties like which price type is being used to pay for these services. The italicized text illustrates these concepts via examples. Users define applications via the API.

![Providers and Servicetypes](/assets/documentation/providers.png)

Fig. 2 shows providers that contain multiple service types, which in turn may have multiple price types associated with them. This data is loaded and updated in the BigBitBus database by the administrator.

<div style="page-break-after: always;"></div>

# BigBitBus API Version 1

This Document shows examples of making REST calls to the BigBitBus API version 1 (this is the current and only version as of now).

Many examples are used to illustrate the API below. All examples assume the BigBitBus API server is running at http://127.0.0.1 and at port 8000.

## 1.1. Authorization

Use the authorization credentials (username, password) to get access to the API.

```bash
curl --request POST \
  --url http://127.0.0.1:8000/api-token-auth/login/ \
  --header 'Accept: */*' \
  --header 'content-type: multipart/form-data' \
  --form username=examplecorpuser \
  --form password=securepassword
```

The command will return a Java Web Token (JWT) `token`, its string value is used in subsequent API requests.

```bash
{"token":"eyJ0eXAiOiJKV1QiLCJhbasdas3qJIUzI1NiJ9.eyJ1c2VyX2lkIjoyMywidXNlcm5hbWUiOiJleGFtcGxlY29ycHVzZXIiLCJleHAiOjE1NjkwOTU4NzEsImVtYWlsIjoiIn0.oW2iT6d99Bypdsux5wvzFpR6IqVU"}
```

<div style="page-break-after: always;"></div>

## 1.2. Applications

### 1.2.1. Create an application

Send a `POST` request to create an application in the database.

```bash
curl -X POST \
  http://127.0.0.1:8000/api/v1.0/calc/applications/ \
  -H 'Accept: */*' \
  -H 'Accept-Encoding: gzip, deflate' \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkwOTQ5OTksImVtYWlsIjoiIn0.wLUVsUtZMRwLj6bvRK5dWAFmUTwYPNMbGpuHmmO9mXY' \
  -d '{
    "app_name": "Enterprise_HR_SAAS_Application",
    "decription": "Finance department application",
    "services": [
        {
            "service_type": "aws-c5.xlarge",
            "location": "us-west-2",
            "price_type": "onDemandPrice",
            "provider":"aws",
            "attributes": [
                {
                    "attr_id": "unit",
                    "value": "instance"
                },
                {
                    "attr_id": "util",
                    "value": "40"
                },
                {
                    "attr_id": "max_util",
                    "value": "80"
                },
                {
                    "attr_id": "quantity",
                    "value": "8"
                }
            ]
        },
        {
            "service_type": "aws-c5.2xlarge",
            "location": "us-east-2",
            "price_type": "onDemandPrice",
            "provider":"aws",
            "attributes": [
                {
                    "attr_id": "unit",
                    "value": "instance"
                },
                {
                    "attr_id": "util",
                    "value": "30"
                },
                {
                    "attr_id": "max_util",
                    "value": "80"
                },
                {
                    "attr_id": "quantity",
                    "value": "3"
                }
            ]
        }
    ]
}'
```

Tables for ServiceTypes and PriceTypes are available by navigating to `http://127.0.0.1:8000/data/[services,prices]`.

### 1.2.2. Retrieving Applications

`GET` requests can be used to read created applications:

1. Get a list of all applications belonging to the authorized user.

```bash
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/applications/ \
  -H 'Accept: */*' \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkwOTQ5OTksImVtYWlsIjoiIn0.wLUVsUtZMRwLj6bvRK5dWAFmUTwYPNMbGpuHmmO9mXY' \
  -H 'Content-Type: application/json'
```

In case there are too many applications, pagination can be used, like so

```bash
curl -X GET \
  'http://127.0.0.1:8000/api/v1.0/calc/applications/?limit=1&offset=0' \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkyODkzMDAsImVtYWlsIjoiIn0.ESU0xx9I8MVuIhvN4eD02szSzz3bhi5D3CpCiKQQoFk' \
  -H 'Content-Type: application/json'


```

2. Get the details of a specific application. This can be accomplished by passing the primary key `pk` or the `app_name` in the URL, like so

```bash
# Using the pk of the application
 curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/applications/pk/89a8e4463ed6ab13713d7ad8c82b40e23d6dec023348d49625e74885/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkwOTQ5OTksImVtYWlsIjoiIn0.wLUVsUtZMRwLj6bvRK5dWAFmUTwYPNMbGpuHmmO9mXY' \
  -H 'Content-Type: application/json'
```

```bash
# Using the app_name of the application; this also means that two applications belonging to the same user cannot have identical app_names.
curl -X GET \
http://127.0.0.1:8000/api/v1.0/calc/applications/app_name/Enterprise_HR_SAAS_Application/ \
-H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkwOTQ5OTksImVtYWlsIjoiIn0.wLUVsUtZMRwLj6bvRK5dWAFmUTwYPNMbGpuHmmO9mXY' \
-H 'Content-Type: application/json' \
```

### 1.2.3. Obtaining a report on an Application

A report contains cost data for the application. As before, the application can be specified using a `pk` or the `app_name`.

```bash
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/reports/applications/app_name/Enterprise_HR_SAAS_Application/ \
  -H 'Accept: */*' \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkwOTQ5OTksImVtYWlsIjoiIn0.wLUVsUtZMRwLj6bvRK5dWAFmUTwYPNMbGpuHmmO9mXY' \
  -H 'Content-Type: application/json'
#
# OR
#
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/reports/applications/pk/89a8e4463ed6ab13713d7ad8c82b40e23d6dec023348d49625e74885/ \
  -H 'Accept: */*' \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkwOTQ5OTksImVtYWlsIjoiIn0.wLUVsUtZMRwLj6bvRK5dWAFmUTwYPNMbGpuHmmO9mXY' \
  -H 'Content-Type: application/json'


```

Here is an example of a report returned by the above API calls

```json
{
  "application": {
    "app_name": "Enterprise_HR_SAAS_Application",
    "date_created": "2019-09-20T01:51:24.368595Z",
    "pk_hash": "89a8e4463ed6ab13713d7ad8c82b40e23d6dec023348d49625e74885",
    "services": [
      {
        "attributes": [
          {
            "attr_id": "unit",
            "value": "instance"
          },
          {
            "attr_id": "util",
            "value": "40"
          },
          {
            "attr_id": "max_util",
            "value": "80"
          },
          {
            "attr_id": "quantity",
            "value": "8"
          }
        ],
        "service_type": "aws-c5.xlarge",
        "date_created": "2019-09-20T01:51:24.416991Z",
        "pk_hash": "881a3b377b41ff3c8d6084d79f8e1bfa42d9fc45f9642ef9eead88b8",
        "location": "us-west-2",
        "price_type": "onDemandPrice",
        "provider": "aws"
      },
      {
        "attributes": [
          {
            "attr_id": "unit",
            "value": "instance"
          },
          {
            "attr_id": "util",
            "value": "30"
          },
          {
            "attr_id": "max_util",
            "value": "80"
          },
          {
            "attr_id": "quantity",
            "value": "3"
          }
        ],
        "service_type": "aws-c5.2xlarge",
        "date_created": "2019-09-20T01:51:24.485717Z",
        "pk_hash": "45ddf322bad20dae58a7cf736493e02a7d178448154daec065fa0398",
        "location": "us-east-2",
        "price_type": "onDemandPrice",
        "provider": "aws"
      }
    ],
    "decription": "Finance department application"
  },
  "cost_components": [
    {
      "currentGen": "True",
      "numcpus": "4",
      "memory": "7.45",
      "quantity": "8",
      "util": "40",
      "max_util": "80",
      "service": "aws-c5.xlarge",
      "unit_cost": 0.17,
      "unit": "",
      "location": "us-west-2",
      "service_cost": 1.36,
      "price_type": "onDemandPrice"
    },
    {
      "currentGen": "True",
      "memory": "15.1",
      "numcpus": "8",
      "quantity": "3",
      "util": "30",
      "max_util": "80",
      "service": "aws-c5.2xlarge",
      "unit_cost": 0.34,
      "unit": "",
      "location": "us-east-2",
      "service_cost": 1.02,
      "price_type": "onDemandPrice"
    }
  ],
  "total_cost": 2.38
}
```

### 1.2.4. Querying Applications

The application name and its description may have keywords that the user wants to query. This can be accomplished using an API call like so

```bash
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/applicationsearch/aqw Cassandra\
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkyODkzMDAsImVtYWlsIjoiIn0.ESU0xx9I8MVuIhvN4eD02szSzz3bhi5D3CpCiKQQoFk' \
  -H 'Content-Type: application/json' \
```

### 1.2.5. Finding Application Alternatives on Another Provider

We now use the BigBitBus API matching engine to find comparable services in other cloud providers in case the user wants to migrate to another cloud provider.

In our example we wish to look at moving the application into the Azure provider (aze) from the current AWS provider.

```bash
curl -X GET \
  'http://127.0.0.1:8000/api/v1.0/calc/applications/app_name/Enterprise_HR_SAAS_Application/provider/aze/pricetype/onDemandPrice/?matchtype=attribute' \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json'
```

This yields

```json
{
  "application": {
    "app_name": "Enterprise_HR_SAAS_Application",
    "date_created": "2019-09-20T01:51:24.368595Z",
    "pk_hash": "89a8e4463ed6ab13713d7ad8c82b40e23d6dec023348d49625e74885",
    "services": [
      {
        "attributes": [
          {
            "attr_id": "unit",
            "value": "instance"
          },
          {
            "attr_id": "util",
            "value": "40"
          },
          {
            "attr_id": "max_util",
            "value": "80"
          },
          {
            "attr_id": "quantity",
            "value": "8"
          }
        ],
        "service_type": "aws-c5.xlarge",
        "date_created": "2019-09-20T01:51:24.416991Z",
        "pk_hash": "881a3b377b41ff3c8d6084d79f8e1bfa42d9fc45f9642ef9eead88b8",
        "location": "us-west-2",
        "price_type": "onDemandPrice",
        "provider": "aws"
      },
      {
        "attributes": [
          {
            "attr_id": "unit",
            "value": "instance"
          },
          {
            "attr_id": "util",
            "value": "30"
          },
          {
            "attr_id": "max_util",
            "value": "80"
          },
          {
            "attr_id": "quantity",
            "value": "3"
          }
        ],
        "service_type": "aws-c5.2xlarge",
        "date_created": "2019-09-20T01:51:24.485717Z",
        "pk_hash": "45ddf322bad20dae58a7cf736493e02a7d178448154daec065fa0398",
        "location": "us-east-2",
        "price_type": "onDemandPrice",
        "provider": "aws"
      }
    ],
    "decription": "Finance department application"
  },
  "alternative_option": {
    "components": [
      {
        "machine_name": "aze-standard_a4_v2",
        "quantity": "8",
        "costph": 0.159,
        "numcpus": "4",
        "memory": "7.79",
        "price_type": "onDemandPrice",
        "service_cost": 1.272,
        "note": "You might also want to consider these (similar but higher priced) services: ,aze-standard_a3(0.202) ,aze-standard_f4s(0.199) ,aze-basic_a3(0.176) ,aze-standard_f4s_v2(0.17) ,aze-standard_f4(0.199) ",
        "Instead of": "aws-c5.xlarge in us-west-2"
      },
      {
        "machine_name": "aze-standard_f8s_v2",
        "quantity": "3",
        "costph": 0.34,
        "numcpus": "8",
        "memory": "16",
        "price_type": "onDemandPrice",
        "service_cost": 1.02,
        "note": "You might also want to consider these (similar but higher priced) services: ,aze-standard_a8_v2(0.4) ,aze-standard_f8(0.398) ,aze-basic_a4(0.376) ,aze-standard_f8s(0.398) ,aze-standard_a4(0.48) ",
        "Instead of": "aws-c5.2xlarge in us-east-2"
      }
    ],
    "total_cost": 2.292
  }
}
```

The matching engine has returned similar services in Azure which may be used to compose the same application there, should the user decide to migrate from AWS to Azure. Compare the "total_cost" - $2.292 per hour in Azure with the cost report of Step 4 - $2.38 in AWS. The output may also include some a "note" about other possible matching options and their hourly costs.

The matching engine compares service type attributes (e.g. numcpus and memory in the case of VMs) in order to make comparisons and present similar services from other providers.

### 1.2.6. VM Optimization

If the average CPU utilization of a VM is specified in the application (`util`) then there are a few use cases where the VMs being used by the application can be optimized.

1. Optimize the cost of running the service within the same provider by choosing smaller VMs in case CPU utilization is low:

```bash
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/applications/app_name/Enterprise_HR_SAAS_Application/provider/aws/pricetype/onDemandPrice/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json'
```

This returns the following JSON string

```json
{
  "application": {
    "app_name": "Enterprise_HR_SAAS_Application",
    "date_created": "2019-09-20T01:51:24.368595Z",
    "pk_hash": "89a8e4463ed6ab13713d7ad8c82b40e23d6dec023348d49625e74885",
    "services": [
      {
        "attributes": [
          {
            "attr_id": "unit",
            "value": "instance"
          },
          {
            "attr_id": "util",
            "value": "40"
          },
          {
            "attr_id": "max_util",
            "value": "80"
          },
          {
            "attr_id": "quantity",
            "value": "8"
          }
        ],
        "service_type": "aws-c5.xlarge",
        "date_created": "2019-09-20T01:51:24.416991Z",
        "pk_hash": "881a3b377b41ff3c8d6084d79f8e1bfa42d9fc45f9642ef9eead88b8",
        "location": "us-west-2",
        "price_type": "onDemandPrice",
        "provider": "aws"
      },
      {
        "attributes": [
          {
            "attr_id": "unit",
            "value": "instance"
          },
          {
            "attr_id": "util",
            "value": "30"
          },
          {
            "attr_id": "max_util",
            "value": "80"
          },
          {
            "attr_id": "quantity",
            "value": "3"
          }
        ],
        "service_type": "aws-c5.2xlarge",
        "date_created": "2019-09-20T01:51:24.485717Z",
        "pk_hash": "45ddf322bad20dae58a7cf736493e02a7d178448154daec065fa0398",
        "location": "us-east-2",
        "price_type": "onDemandPrice",
        "provider": "aws"
      }
    ],
    "decription": "Finance department application"
  },
  "current_application": {
    "components": [
      {
        "currentGen": "True",
        "numcpus": "4",
        "memory": "7.45",
        "quantity": "8",
        "util": "40",
        "max_util": "80",
        "service": "aws-c5.xlarge",
        "unit_cost": 0.17,
        "unit": "",
        "location": "us-west-2",
        "service_cost": 1.36,
        "price_type": "onDemandPrice"
      },
      {
        "currentGen": "True",
        "memory": "15.1",
        "numcpus": "8",
        "quantity": "3",
        "util": "30",
        "max_util": "80",
        "service": "aws-c5.2xlarge",
        "unit_cost": 0.34,
        "unit": "",
        "location": "us-east-2",
        "service_cost": 1.02,
        "price_type": "onDemandPrice"
      }
    ],
    "total_cost": 2.38
  },
  "alternative_option": {
    "components": [
      {
        "machine_name": "aws-t3.xlarge",
        "machine_cpu": 41.56805002183218,
        "numcpus": "4",
        "memory": "15.48",
        "costph": 0.166,
        "description": "aws-t3.xlarge",
        "price_type": "onDemandPrice",
        "location": "us-west-2",
        "service_cost": 1.328
      },
      {
        "machine_name": "aws-t3.xlarge",
        "machine_cpu": 65.54483646460177,
        "numcpus": "4",
        "memory": "15.48",
        "costph": 0.166,
        "description": "aws-t3.xlarge",
        "price_type": "onDemandPrice",
        "location": "us-east-2",
        "service_cost": 0.498
      }
    ],
    "total_cost": 1.83
  }
}
```

The BigBitBus API server has performance information about 100s of VMs across different cloud providers. It finds other VMs which may be cheaper than the original service types. Note that the CPU utilization of the suggested VMs reported by the tool is an estimate based on CPU performance and utilization models. The `util` and `max_util` parameters set in the application are used as starting points for the calculation. THe `util` parameter is the average utilization of the VM's CPU whereas the `max_util` is the maximum utilization of the suggested CPU that the user is willing to allow. For example if `max_util` is 80 then the tool will only consider VMs whose performance model indicates that less than 80\% CPU will be utilized.

It is important to note that the utilization is an estimate; users are encouraged to verify the utilization for their specific application on the suggested options before deploying the application for production use.

2. A user may also want to optimize VMs when moving an application from one cloud to another. For example, consider the case when a user is considering moving from AWS to Azure and she wants to optimize costs by choosing smaller VMs, if low CPU utilization allows:

```bash
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/applications/app_name/Enterprise_HR_SAAS_Application/provider/aze/pricetype/onDemandPrice/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json'
```

This returns the following alternate options for the application.

```json
{
  "application": {
    "app_name": "Enterprise_HR_SAAS_Application",
    "date_created": "2019-09-20T01:51:24.368595Z",
    "pk_hash": "89a8e4463ed6ab13713d7ad8c82b40e23d6dec023348d49625e74885",
    "services": [
      {
        "attributes": [
          {
            "attr_id": "unit",
            "value": "instance"
          },
          {
            "attr_id": "util",
            "value": "40"
          },
          {
            "attr_id": "max_util",
            "value": "80"
          },
          {
            "attr_id": "quantity",
            "value": "8"
          }
        ],
        "service_type": "aws-c5.xlarge",
        "date_created": "2019-09-20T01:51:24.416991Z",
        "pk_hash": "881a3b377b41ff3c8d6084d79f8e1bfa42d9fc45f9642ef9eead88b8",
        "location": "us-west-2",
        "price_type": "onDemandPrice",
        "provider": "aws"
      },
      {
        "attributes": [
          {
            "attr_id": "unit",
            "value": "instance"
          },
          {
            "attr_id": "util",
            "value": "30"
          },
          {
            "attr_id": "max_util",
            "value": "80"
          },
          {
            "attr_id": "quantity",
            "value": "3"
          }
        ],
        "service_type": "aws-c5.2xlarge",
        "date_created": "2019-09-20T01:51:24.485717Z",
        "pk_hash": "45ddf322bad20dae58a7cf736493e02a7d178448154daec065fa0398",
        "location": "us-east-2",
        "price_type": "onDemandPrice",
        "provider": "aws"
      }
    ],
    "decription": "Finance department application"
  },
  "current_application": {
    "components": [
      {
        "currentGen": "True",
        "numcpus": "4",
        "memory": "7.45",
        "quantity": "8",
        "util": "40",
        "max_util": "80",
        "service": "aws-c5.xlarge",
        "unit_cost": 0.17,
        "unit": "",
        "location": "us-west-2",
        "service_cost": 1.36,
        "price_type": "onDemandPrice"
      },
      {
        "currentGen": "True",
        "memory": "15.1",
        "numcpus": "8",
        "quantity": "3",
        "util": "30",
        "max_util": "80",
        "service": "aws-c5.2xlarge",
        "unit_cost": 0.34,
        "unit": "",
        "location": "us-east-2",
        "service_cost": 1.02,
        "price_type": "onDemandPrice"
      }
    ],
    "total_cost": 2.38
  },
  "alternative_option": {
    "components": [
      {
        "machine_name": "aze-standard_b4ms",
        "machine_cpu": 75.43015027854008,
        "numcpus": "4",
        "memory": "15.67",
        "costph": 0.166,
        "description": "aze-standard_b4ms",
        "price_type": "onDemandPrice",
        "location": "us-west-2",
        "service_cost": 1.328
      },
      {
        "machine_name": "aze-standard_f4",
        "machine_cpu": 69.46153379604617,
        "numcpus": "4",
        "memory": "7.79",
        "costph": 0.199,
        "description": "aze-standard_f4",
        "price_type": "onDemandPrice",
        "location": "us-east-2",
        "service_cost": 0.597
      }
    ],
    "total_cost": 1.93
  }
}
```

3. A user may want to also optimize by selecting another price_type. For example, in addition to moving to a smaller VM within AWS, a user may want to move from an `onDemandPrice` to `reserved1yearPrice`. Cloud providers offer discounts for reserved instances.

```bash
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/applications/app_name/Enterprise_HR_SAAS_Application/provider/aws/pricetype/reserved1yearPrice/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json'

```

```json
{
  "application": {
    "app_name": "Enterprise_HR_SAAS_Application",
    "date_created": "2019-09-20T01:51:24.368595Z",
    "pk_hash": "89a8e4463ed6ab13713d7ad8c82b40e23d6dec023348d49625e74885",
    "services": [
      {
        "attributes": [
          {
            "attr_id": "unit",
            "value": "instance"
          },
          {
            "attr_id": "util",
            "value": "40"
          },
          {
            "attr_id": "max_util",
            "value": "80"
          },
          {
            "attr_id": "quantity",
            "value": "8"
          }
        ],
        "service_type": "aws-c5.xlarge",
        "date_created": "2019-09-20T01:51:24.416991Z",
        "pk_hash": "881a3b377b41ff3c8d6084d79f8e1bfa42d9fc45f9642ef9eead88b8",
        "location": "us-west-2",
        "price_type": "onDemandPrice",
        "provider": "aws"
      },
      {
        "attributes": [
          {
            "attr_id": "unit",
            "value": "instance"
          },
          {
            "attr_id": "util",
            "value": "30"
          },
          {
            "attr_id": "max_util",
            "value": "80"
          },
          {
            "attr_id": "quantity",
            "value": "3"
          }
        ],
        "service_type": "aws-c5.2xlarge",
        "date_created": "2019-09-20T01:51:24.485717Z",
        "pk_hash": "45ddf322bad20dae58a7cf736493e02a7d178448154daec065fa0398",
        "location": "us-east-2",
        "price_type": "onDemandPrice",
        "provider": "aws"
      }
    ],
    "decription": "Finance department application"
  },
  "current_application": {
    "components": [
      {
        "currentGen": "True",
        "numcpus": "4",
        "memory": "7.45",
        "quantity": "8",
        "util": "40",
        "max_util": "80",
        "service": "aws-c5.xlarge",
        "unit_cost": 0.17,
        "unit": "",
        "location": "us-west-2",
        "service_cost": 1.36,
        "price_type": "onDemandPrice"
      },
      {
        "currentGen": "True",
        "memory": "15.1",
        "numcpus": "8",
        "quantity": "3",
        "util": "30",
        "max_util": "80",
        "service": "aws-c5.2xlarge",
        "unit_cost": 0.34,
        "unit": "",
        "location": "us-east-2",
        "service_cost": 1.02,
        "price_type": "onDemandPrice"
      }
    ],
    "total_cost": 2.38
  },
  "alternative_option": {
    "components": [
      {
        "machine_name": "aws-t3.xlarge",
        "machine_cpu": 41.56805002183218,
        "numcpus": "4",
        "memory": "15.48",
        "costph": 0.104,
        "description": "aws-t3.xlarge",
        "price_type": "reserved1yearPrice",
        "location": "us-west-2",
        "service_cost": 0.832
      },
      {
        "machine_name": "aws-t3.xlarge",
        "machine_cpu": 65.54483646460177,
        "numcpus": "4",
        "memory": "15.48",
        "costph": 0.104,
        "description": "aws-t3.xlarge",
        "price_type": "reserved1yearPrice",
        "location": "us-east-2",
        "service_cost": 0.312
      }
    ],
    "total_cost": 1.14
  }
}
```

### 1.2.7. Updating an Application

Applications can be updated using a `PUT` request. Currently all the service instances of an application need to be necessarily specified even if only one is changed. The name of an application cannot be changed; a new application should instead be created via `POST` request. The description of the application can be updated.

The following PUT request changes some of the service instances of the application. We can alternately use the `pk`, as illustrated in the `GET` method for a specific application above.:

```bash
curl -X PUT \
  http://127.0.0.1:8000/api/v1.0/calc/applications/app_name/Enterprise_HR_SAAS_Application/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json' \
  -d '{
    "app_name": "Enterprise_HR_SAAS_Application",
    "services": [
        {
            "service_type": "aws-c5.2xlarge",
            "quantity": 4,
            "location": "us-west-2",
            "price_type": "reserved1yearPrice",
            "provider":"aws",
            "attributes": [
                {
                    "attr_id": "unit",
                    "value": "instance"
                },
                {
                    "attr_id": "util",
                    "value": "20"
                },
                {
                    "attr_id": "max_util",
                    "value": "30"
                },
                {
                    "attr_id": "quantity",
                    "value": "4"
                }
            ]
        },
        {
            "service_type": "aws-c5.2xlarge",
            "quantity": 3,
            "location": "us-east-2",
            "price_type": "onDemandPrice",
            "provider":"aws",
            "attributes": [
                {
                    "attr_id": "unit",
                    "value": "instance"
                },
                {
                    "attr_id": "util",
                    "value": "30"
                },
                {
                    "attr_id": "max_util",
                    "value": "80"
                },
                {
                    "attr_id": "quantity",
                    "value": "3"
                }
            ]
        }
    ],
    "decription": "HR department application"
}'
```

Returns `200 OK` if the update succeeds.

The updated application is returned.

```bash
{
    "app_name": "Enterprise_HR_SAAS_Application",
    "date_created": "2019-09-20T01:51:24.368595Z",
    "pk_hash": "89a8e4463ed6ab13713d7ad8c82b40e23d6dec023348d49625e74885",
    "services": [
        {
            "attributes": [
                {
                    "attr_id": "unit",
                    "value": "instance"
                },
                {
                    "attr_id": "util",
                    "value": "20"
                },
                {
                    "attr_id": "max_util",
                    "value": "30"
                },
                {
                    "attr_id": "quantity",
                    "value": "4"
                }
            ],
            "service_type": "aws-c5.2xlarge",
            "date_created": "2019-09-21T02:37:21.035897Z",
            "pk_hash": "379fe4b040f72c9fc94d96f99e760b2627f2b2b5c76ecbd3931fb479",
            "location": "us-west-2",
            "price_type": "reserved1yearPrice",
            "provider": "aws"
        },
        {
            "attributes": [
                {
                    "attr_id": "unit",
                    "value": "instance"
                },
                {
                    "attr_id": "util",
                    "value": "30"
                },
                {
                    "attr_id": "max_util",
                    "value": "80"
                },
                {
                    "attr_id": "quantity",
                    "value": "3"
                }
            ],
            "service_type": "aws-c5.2xlarge",
            "date_created": "2019-09-21T02:37:21.154060Z",
            "pk_hash": "45ddf322bad20dae58a7cf736493e02a7d178448154daec065fa0398",
            "location": "us-east-2",
            "price_type": "onDemandPrice",
            "provider": "aws"
        }
    ],
    "decription": "HR department application"
}
```

### 1.2.8. Deleting an Application

Applications may be deleted as follows; as with the `GET` method, `app_name` or `pk` can be used to construct the URL.

```bash
curl -X DELETE \
  http://127.0.0.1:8000/api/v1.0/calc/applications/app_name/Enterprise_HR_SAAS_Application/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json' \

```

This returns `204 No Content` on successful deletion.

## 1.3. Prices

Prices for public cloud services are included in the database; however users can add custom prices.

### 1.3.1. Creating a new price for a service (POST)

Each service type can have multiple prices associated with it. For example, public cloud providers have `onDemandPrice`, `reserved1yearPrice` and `reserved3yearPrice`. Users can define their own price types, for example, if a user obtains special pricing from a provider. All prices are expressed as floating numbers - \$USD per hour.

```bash
curl -X POST \
  http://127.0.0.1:8000/api/v1.0/calc/price/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json' \
  -d '{
	"service_pk": "45d7822c23fe69282e5640092775050834c1b7ec987d92d2a206c3a8",
	"name": "examplecorp-aws-valued-customer-price",
	"costperunit": 0.001
}'
```

The returned output is

```JSON
{
    "service_name": "aws-t3.small",
    "name": "examplecorp-aws-valued-customer-price",
    "service_pk": "45d7822c23fe69282e5640092775050834c1b7ec987d92d2a206c3a8",
    "costperunit": 0.001,
    "location": "us-east-2"
}
```

This returns `201 created` on success.

Note that the `location` key is deciphered from the service location.

### 1.3.2. Get price details about a pre-existing price (GET)

Details about a pre-existing pricetype can be queried either by using the `pk` of the price or by using a combination of the `service_pk` and the `pricetype` (the name of the price like "onDemandPrice").

Using `pk`:

```bash
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/price/pk/789a89b8bcc4ce13552cfdd882cee4b130bbf1db0ad3d83b2a1600a9/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json' \

```

...or

```
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/price/service_pk/45d7822c23fe69282e5640092775050834c1b7ec987d92d2a206c3a8/pricetype/examplecorp-aws-valued-customer-price/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json'
```

Both the approaches yield the price detail

```json
{
  "service_name": "aws-t3.small",
  "pk": "dc0c96587629b22e441020309b37227cb885632e2b09caba40cee5be",
  "name": "examplecorp-aws-valued-customer-price",
  "service_pk": "45d7822c23fe69282e5640092775050834c1b7ec987d92d2a206c3a8",
  "costperunit": 0.001,
  "location": "us-east-2"
}
```

### 1.3.3. Updating the costperunit for a pre-existing price (PUT)

The `costperunit` value can be updated for any price (as long as it is "owned" - originally created - by the user).
Again, the `service_pk` and `pricetype` or the `pk` of the price can be used as in the `GET` request above.

```bash
curl -X PUT \
  http://127.0.0.1:8000/api/v1.0/calc/price/service_pk/45d7822c23fe69282e5640092775050834c1b7ec987d92d2a206c3a8/pricetype/examplecorp-aws-valued-customer-price/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json' \
  -d '{
	"costperunit": 0.005
}'
```

The modified price is returned

```json
{
  "service_name": "aws-t3.small",
  "pk": "dc0c96587629b22e441020309b37227cb885632e2b09caba40cee5be",
  "name": "examplecorp-aws-valued-customer-price",
  "service_pk": "45d7822c23fe69282e5640092775050834c1b7ec987d92d2a206c3a8",
  "costperunit": 0.005,
  "location": "us-east-2"
}
```

A `200 OK` is returned on success.

### 1.3.4. Deleting a price (DELETE)

Similarly, a price can be deleted using the `service_pk` and `pricetype` or the `pk` of the price can be used as in the `GET` request above.

Using `service_pk` and `pricetype`

```
curl -X DELETE \
  http://127.0.0.1:8000/api/v1.0/calc/price/service_pk/45d7822c23fe69282e5640092775050834c1b7ec987d92d2a206c3a8/pricetype/examplecorp-aws-valued-customer-price/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json' \
```

...or using the price `pk`

```
curl -X DELETE \
  http://127.0.0.1:8000/api/v1.0/calc/price/pk/dc0c96587629b22e441020309b37227cb885632e2b09caba40cee5be/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json' \
```

A `204 No Content` is returned.

### 1.3.5. Filtering Price data

#### 1.3.5.1. Getting all prices for a service type

Specify the `service_pk` in the `GET` url.

```bash
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/price/service_pk/ffe6434785a8cca79c3837aec689e8294715c7330f778324d6a1a274/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json' \
```

Returns

```json
[
  {
    "service_name": "aze-standard_f1",
    "pk": "5d63d5ac83ddaade3dce51459c2da9a00fd6d795a6f3495ef29ec1d9",
    "name": "reserved3yearPrice",
    "service_pk": "ffe6434785a8cca79c3837aec689e8294715c7330f778324d6a1a274",
    "costperunit": 0.02842,
    "location": "koreacentral"
  },
  {
    "service_name": "aze-standard_f1",
    "pk": "80838a27745a8adef731df57fcbf1ef0d0cfe2a6aabe233a13caf371",
    "name": "reserved1yearPrice",
    "service_pk": "ffe6434785a8cca79c3837aec689e8294715c7330f778324d6a1a274",
    "costperunit": 0.03927,
    "location": "koreacentral"
  },
  {
    "service_name": "aze-standard_f1",
    "pk": "b17b7b949031819d283f374165e79460058417781c3a2c5c10ce7277",
    "name": "onDemandPrice",
    "service_pk": "ffe6434785a8cca79c3837aec689e8294715c7330f778324d6a1a274",
    "costperunit": 0.051,
    "location": "koreacentral"
  }
]
```

A `200 OK` is returned.

#### 1.3.5.2 Get the price of a ServiceType (specified by its service_pk) of a certain pricetype

```bash
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/price/service_pk/ffc653c1505713081527f1a39894d5fe16cebd8f81593c6622165cc7/pricetype/reserved1yearPrice/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json'
```

Returns the reserved1yearPrice for the service with the service_pk specified in the URL.

```json
{
  "service_name": "aws-i3.large",
  "pk": "105d070f7c14463193739bde119cd4d717483c1b028bc278ac44ba03",
  "name": "reserved1yearPrice",
  "service_pk": "ffc653c1505713081527f1a39894d5fe16cebd8f81593c6622165cc7",
  "costperunit": 0.118,
  "location": "eu-west-1"
}
```

A `200 OK` return code is also sent backby the BigBitBus API server.

#### 1.3.5.3. Getting all prices of a certain pricetype

If we want all the prices of a certain pricetype (name) then

```bash
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/price/pricetype/examplecorp-aws-valued-customer-reduced-price/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json'
```

this returns a list of prices

```json
[
  {
    "service_name": "aws-t3.small",
    "pk": "789a89b8bcc4ce13552cfdd882cee4b130bbf1db0ad3d83b2a1600a9",
    "name": "examplecorp-aws-valued-customer-reduced-price",
    "service_pk": "45d7822c23fe69282e5640092775050834c1b7ec987d92d2a206c3a8",
    "costperunit": 0.001,
    "location": "us-east-2"
  },
  {
    "service_name": "aws-i3.large",
    "pk": "c8ff55168e90749f911a87a5e6ff789e2e2d3f1440827925cb4c631d",
    "name": "examplecorp-aws-valued-customer-reduced-price",
    "service_pk": "ffc653c1505713081527f1a39894d5fe16cebd8f81593c6622165cc7",
    "costperunit": 0.01,
    "location": "eu-west-1"
  }
]
```

A `200 OK` response code is returned.

#### 1.3.5.4. Getting all prices

To get details of all prices stored in th database.

Note the use of limit (number of prices returned) and the offset in order to limit the returned output.

```bash
curl -X GET \
  'http://127.0.0.1:8000/api/v1.0/calc/price/?limit=3&offset=4000' \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkyODkzMDAsImVtYWlsIjoiIn0.ESU0xx9I8MVuIhvN4eD02szSzz3bhi5D3CpCiKQQoFk' \
  -H 'Content-Type: application/json'

```

This returns a `200 OK` status code on success.

## 1.4. ServiceTypes

In addition to the public cloud service types, additional service types can also be added. For example, a service type to capture the "server admin"
component of an application.

### 1.4.1. Create a new service type (POST)

Create new service type like so

```bash
curl -X POST \
  http://127.0.0.1:8000/api/v1.0/calc/servicetype/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json' \
  -d '{
    "attributes": [
        {
            "attr_id": "quantity",
            "value": "60"
        },
        {
            "attr_id": "unit",
            "value": "instance"
        },
        {
            "attr_id": "util",
            "value": "30"
        },
        {
            "attr_id": "max_util",
            "value": "90"
        }
    ],
    "name": "my_misc_service_1",
    "location": "eastus2",
    "type": "compute",
    "unit": "undefined",
    "provider": "alt"
}'
```

The new service type is returned as a JSON string.

```json
{
  "name": "my_misc_service_1",
  "provider": "alt",
  "location": "eastus2",
  "type": "compute",
  "unit": "undefined",
  "attributes": [
    {
      "attr_id": "quantity",
      "value": "60"
    },
    {
      "attr_id": "unit",
      "value": "instance"
    },
    {
      "attr_id": "util",
      "value": "30"
    },
    {
      "attr_id": "max_util",
      "value": "90"
    }
  ],
  "pk_hash": "e72e11be451152e0758d18248d700e05c36e0b0a4a1929f612fc6ea6",
  "user_defined": true
}
```

The `user_defined` field indicates that the servicetype has been created by an end-user (and is not a "standard" servicetype that BigBitBus shipped. This boolean will be useful when deciding if a delete request should be honoured or not.

A `201 Created` code is returned.

### 1.4.2. Retrieve a service type (GET)

We can use the pk or a combination of (provider, service name and location) to get details of a certain servicetype.

```bash
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/servicetype/pk/e72e11be451152e0758d18248d700e05c36e0b0a4a1929f612fc6ea6 \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json' \

```

or

```bash
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/servicetype/provider/alt/name/my_misc_service_1/location/eastus2 \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json'

```

yields

```json
{
  "name": "my_misc_service_1",
  "provider": "alt",
  "location": "eastus2",
  "type": "compute",
  "unit": "undefined",
  "attributes": [
    {
      "attr_id": "quantity",
      "value": "60"
    },
    {
      "attr_id": "unit",
      "value": "instance"
    },
    {
      "attr_id": "util",
      "value": "30"
    },
    {
      "attr_id": "max_util",
      "value": "90"
    }
  ],
  "pk_hash": "e72e11be451152e0758d18248d700e05c36e0b0a4a1929f612fc6ea6",
  "user_defined": true
}
```

A `200 OK` code is returned.

### 1.4.3. Deleting a service type (DELETE)

A user can delete a service type if the service type is user-defined (e.g. created by the user via a POST request earlier).

```bash
curl -X DELETE \
  http://127.0.0.1:8000/api/v1.0/calc/servicetype/pk/e72e11be451152e0758d18248d700e05c36e0b0a4a1929f612fc6ea6/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json'

```

or

```bash
curl -X DELETE \
  http://127.0.0.1:8000/api/v1.0/calc/servicetype/provider/alt/name/my_misc_service_1/location/eastus2/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json'

```

A `204 No Content` is returned.

### 1.4.4. Finding service types

#### 1.4.4.1. Service type keyword search

The `servicesearch` endpoint can be used for keyword search. A list of space-separated terms are sent to the API server, as shown in these examples.

Search for "aws-t3.small" in "us-east"

```bash
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/servicesearch/aws-t3.small%20us-east/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkyODkzMDAsImVtYWlsIjoiIn0.ESU0xx9I8MVuIhvN4eD02szSzz3bhi5D3CpCiKQQoFk' \
  -H 'Content-Type: application/json'
```

Search for "aze" "f4" "korea" for Azure VMs with "f4" in their name in a Korean region.

```bash
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/servicesearch/aze%20f4%20korea/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkyODkzMDAsImVtYWlsIjoiIn0.ESU0xx9I8MVuIhvN4eD02szSzz3bhi5D3CpCiKQQoFk' \
  -H 'Content-Type: application/json'
```

#### 1.4.4.2. List all service types

This API call returns all the service types. This is a usually a lot of data!

```bash
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/servicetype/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkyODkzMDAsImVtYWlsIjoiIn0.ESU0xx9I8MVuIhvN4eD02szSzz3bhi5D3CpCiKQQoFk' \
  -H 'Content-Type: application/json' \

```

Instead, consider using pagination with a limit and offset, like so:

```bash
curl -X GET \
  'http://127.0.0.1:8000/api/v1.0/calc/servicetype/?limit=5&offset=4' \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkyODkzMDAsImVtYWlsIjoiIn0.ESU0xx9I8MVuIhvN4eD02szSzz3bhi5D3CpCiKQQoFk' \
  -H 'Content-Type: application/json' \
```

<div style="page-break-after: always;"></div>


**Table of Contents**

- [BigBitBus API Version 1](#bigbitbus-api-version-1)
  - [1.1. Authorization](#11-authorization)
  - [1.2. Applications](#12-applications)
    - [1.2.1. Create an application](#121-create-an-application)
    - [1.2.2. Retrieving Applications](#122-retrieving-applications)
    - [1.2.3. Obtaining a report on an Application](#123-obtaining-a-report-on-an-application)
    - [1.2.4. Querying Applications](#124-querying-applications)
    - [1.2.5. Finding Application Alternatives on Another Provider](#125-finding-application-alternatives-on-another-provider)
    - [1.2.6. VM Optimization](#126-vm-optimization)
    - [1.2.7. Updating an Application](#127-updating-an-application)
    - [1.2.8. Deleting an Application](#128-deleting-an-application)
  - [1.3. Prices](#13-prices)
    - [1.3.1. Creating a new price for a service (POST)](#131-creating-a-new-price-for-a-service-post)
    - [1.3.2. Get price details about a pre-existing price (GET)](#132-get-price-details-about-a-pre-existing-price-get)
    - [1.3.3. Updating the costperunit for a pre-existing price (PUT)](#133-updating-the-costperunit-for-a-pre-existing-price-put)
    - [1.3.4. Deleting a price (DELETE)](#134-deleting-a-price-delete)
    - [1.3.5. Filtering Price data](#135-filtering-price-data)
      - [1.3.5.1. Getting all prices for a service type](#1351-getting-all-prices-for-a-service-type)
      - [1.3.5.2 Get the price of a ServiceType (specified by its service_pk) of a certain pricetype](#1352-get-the-price-of-a-servicetype-specified-by-its-service_pk-of-a-certain-pricetype)
      - [1.3.5.3. Getting all prices of a certain pricetype](#1353-getting-all-prices-of-a-certain-pricetype)
      - [1.3.5.4. Getting all prices](#1354-getting-all-prices)
  - [1.4. ServiceTypes](#14-servicetypes)
    - [1.4.1. Create a new service type (POST)](#141-create-a-new-service-type-post)
    - [1.4.2. Retrieve a service type (GET)](#142-retrieve-a-service-type-get)
    - [1.4.3. Deleting a service type (DELETE)](#143-deleting-a-service-type-delete)
    - [1.4.4. Finding service types](#144-finding-service-types)
      - [1.4.4.1. Service type keyword search](#1441-service-type-keyword-search)
      - [1.4.4.2. List all service types](#1442-list-all-service-types)
