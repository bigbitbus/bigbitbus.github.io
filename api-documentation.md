# The BigBitBus API Documentation<!-- omit in toc -->

Revision v5.0

Date 19/02/2020

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
    - [1.2.9. Deleting multiple Applications](#129-deleting-multiple-applications)
  - [1.3. Prices](#13-prices)
    - [1.3.1. Creating a new price for a service (POST)](#131-creating-a-new-price-for-a-service-post)
    - [1.3.2. Get price details about a pre-existing price (GET)](#132-get-price-details-about-a-pre-existing-price-get)
    - [1.3.3. Updating the costperunit for a pre-existing price (PUT)](#133-updating-the-costperunit-for-a-pre-existing-price-put)
    - [1.3.4. Deleting a price (DELETE)](#134-deleting-a-price-delete)
    - [1.3.5. Deleting multiple prices (POST)](#135-deleting-multiple-prices-post)
    - [1.3.6. Filtering Price data](#136-filtering-price-data)
      - [1.3.6.1. Getting all prices for a service type](#1361-getting-all-prices-for-a-service-type)
      - [1.3.6.2 Get the price of a ServiceType (specified by its service_pk) of a certain pricetype](#1362-get-the-price-of-a-servicetype-specified-by-its-service_pk-of-a-certain-pricetype)
      - [1.3.6.3. Getting all prices of a certain pricetype](#1363-getting-all-prices-of-a-certain-pricetype)
      - [1.3.6.4. Getting all prices](#1364-getting-all-prices)
  - [1.4. ServiceTypes](#14-servicetypes)
    - [1.4.1. Create a new service type (POST)](#141-create-a-new-service-type-post)
    - [1.4.2. Retrieve a service type (GET)](#142-retrieve-a-service-type-get)
    - [1.4.3. Deleting a service type (DELETE)](#143-deleting-a-service-type-delete)
    - [1.4.4. Deleting multiple service types (POST)](#144-deleting-multiple-service-types-post)
    - [1.4.5. Finding service types](#145-finding-service-types)
      - [1.4.5.1. Service type keyword search](#1451-service-type-keyword-search)
      - [1.4.5.2. Service type name autocomplete](#1452-service-type-name-autocomplete)
      - [1.4.5.3. Getting service type info](#1453-getting-service-type-info)
      - [1.4.5.4. List all service types](#1454-list-all-service-types)
  - [1.5. ProviderDiscounts](#15-providerdiscounts)
    - [1.5.1. Create a new discount (POST)](#151-create-a-new-discount-post)
    - [1.5.2. Retrieve a discount (GET)](#152-retrieve-a-discount-get)
    - [1.5.3. Updating the discount amount for a pre-existing discount (PUT)](#153-updating-the-discount-amount-for-a-pre-existing-discount-put)
    - [1.5.4. Deleting a discount (DELETE)](#154-deleting-a-discount-delete)
    - [1.5.5. Listing all discounts](#155-listing-all-discounts)
  - [1.6. Providers](#16-providers)

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

![Applications and Service Instances](applications.png)

In Fig.1 a user defines an application, which may comprise of multiple service instances. Each service instance is composed of one or more service type instance (quantity), along with properties like which price type is being used to pay for these services. The italicized text illustrates these concepts via examples. Users define applications via the API.

![Providers and Servicetypes](providers.png)

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
    "description": "Finance department application",
    "metadata": "this can be any string up to 1024 characters long, for example, fookey=barvalue",
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
The `metadata` and `description` keys are optional strings (each up to 1024 characters long). If not set, they default to empty string `""`.

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

which yields

```json
{
  "count": 2,
  "next": null,
  "previous": null,
  "results": [
    {
      "app_name": "app1",
      "date_created": "2019-11-14T16:18:18.730938Z",
      "pk_hash": "373d10dfd18ae01daca0dcad1448d6b62eca0b7ad66b3c439ba869fa",
      "services": [
        {
          "attributes": [],
          "service_type": "aze-basic_a0",
          "version_date": "2019-11-03T20:22:09.047029Z",
          "pk_hash": "6baf5d6af555b4c4046d6f0a00b0ea6d50ed8629fa97515e70f3fce8",
          "location": "brazilsouth",
          "price_type": "onDemandPrice",
          "provider": "aze"
        }
      ],
      "description": ""
    },
    {
      "app_name": "app2",
      "date_created": "2019-11-20T17:15:42.879412Z",
      "pk_hash": "be78422202906efd425a84bdbb796ef38d91cd068ddb9773cb1aea1d",
      "services": [
        {
          "attributes": [],
          "service_type": "aze-basic_a0",
          "version_date": "2019-11-03T20:22:10.225264Z",
          "pk_hash": "6baf5d6af555b4c4046d6f0a00b0ea6d50ed8629fa97515e70f3fce8",
          "location": "us-east-2",
          "price_type": "reserved1yearPrice",
          "provider": "aze"
        }
      ],
      "description": ""
    }
  ]
}
```

The list results are paginated, and by default shows at most the first 10 applications. The response also contains URLs corresponding to the next and previous 'pages' if applicable, and a count of the total number of applications in the database.

The limit and offset of the results can be customized like so:

```bash
curl -X GET \
  'http://127.0.0.1:8000/api/v1.0/calc/applications/?limit=1&offset=0' \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkyODkzMDAsImVtYWlsIjoiIn0.ESU0xx9I8MVuIhvN4eD02szSzz3bhi5D3CpCiKQQoFk' \
  -H 'Content-Type: application/json'


```

which would yield:

```json
{
  "count": 2,
  "next": "http://127.0.0.1:8000/api/v1.0/calc/applications/?limit=1&offset=1",
  "previous": null,
  "results": [
    {
      "app_name": "app1",
      "date_created": "2019-11-14T16:18:18.730938Z",
      "pk_hash": "373d10dfd18ae01daca0dcad1448d6b62eca0b7ad66b3c439ba869fa",
      "services": [
        {
          "attributes": [],
          "service_type": "aze-basic_a0",
          "version_date": "2019-11-03T20:22:09.047029Z",
          "pk_hash": "6baf5d6af555b4c4046d6f0a00b0ea6d50ed8629fa97515e70f3fce8",
          "location": "brazilsouth",
          "price_type": "onDemandPrice",
          "provider": "aze"
        }
      ],
      "description": ""
    }
  ]
}
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
    "description": "Finance department application",
    "metadata": "this can be any string up to 1024 characters long, for example, fookey=barvalue"
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
#### 1.2.4.1 Querying Applications using ApplicationList
The application may have specific attributes based on which user want to query. Following Filters can be applied to API Call.  

##### Filters  

| Filter  | Description  |
|---|---|
| name_contains   | Filter if application name contains the entered string   |
| metadata_contains   | Filter if application metadata contains entered string  |
| min_time   | Filter if application creation date is more recent than min_time |
| max_time   | Filter if application creation date is earlier than  max_time  |

Note: Times have to be specified in following format:  
```
 YYYY-MM-DD HH:MM[:ss[.uuuuuu]][TZ]  
 # Some Valid DateTime Examples:  
 2019-12-15  
 2020-01-01  
 2019-12-12 9:55:57  
 )
```

We can now filter applications, like so


```bash
curl --location --request GET '127.0.0.1:8000/api/v1.0/calc/applications/?name_contains=CI%20database&metadata_contains=Testing' \
--header 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjozLCJ1c2VybmFtZSI6Im1hY2ludG9zaF93aG9sZXNhbGUiLCJleHAiOjE1ODE3MTQzNzUsImVtYWlsIjoiYWRtaW5AbWFjaGludG9zaC5jb20ifQ.a5bimnHAnfv-rSU5xrt11zX7VqLTy0hefOqyYyaPj1s' \
--header 'Content-Type: application/json'
```
It returns following Output:
```json
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
            "app_name": "CI database",
            "date_created": "2019-12-11T03:27:27.842342Z",
            "pk_hash": "14ee6590b4e342b91be4294e7dd64b39525ed8f0ae6fbbe23ddaad5b",
            "services": [
                {
                    "attributes": [
                        {
                            "attr_id": "unit",
                            "value": "instance"
                        },
                        {
                            "attr_id": "util",
                            "value": "15"
                        },
                        {
                            "attr_id": "max_util",
                            "value": "80"
                        }
                    ],
                    "service_type": "aws-t3.2xlarge",
                    "version_date": "2019-11-03T20:24:34.333836Z",
                    "pk_hash": "4bd0129bf078e5d6d1ad0966f4c2481194041e175cdcebc49673a988",
                    "location": "ca-central-1",
                    "price_type": "onDemandPrice",
                    "provider": "aws",
                    "quantity": 10,
                    "description": ""
                },
                {
                    "attributes": [],
                    "service_type": "Custom_Organization_IT_HR_cost",
                    "version_date": "2019-12-11T03:27:26Z",
                    "pk_hash": "4d87230cddb963c11a1c1a7a0989d5c89c6d5eb113463f811ce18e7c",
                    "location": "anywhere",
                    "price_type": "hr_overhead_cost",
                    "provider": "org",
                    "quantity": 8,
                    "description": ""
                },
                {
                    "attributes": [],
                    "service_type": "Custom_Organization_amortized_IT_cost",
                    "version_date": "2019-12-11T03:27:26Z",
                    "pk_hash": "09c6fe490b524143121e5b22130a91c2ca01f7dab43c749ad508dd1b",
                    "location": "anywhere",
                    "price_type": "hr_overhead_cost",
                    "provider": "org",
                    "quantity": 8,
                    "description": ""
                }
            ],
            "description": "Database for all IT CI numbers in the organization",
            "metadata": "Testing It"
        }
    ]
}

```

##### Sort Criteria

The returned application list can (optionally) be pre-sorted based on the following values for the `sort` parameter in the url.

| sort Parameter value | Meaning |
| --- | --- |
| date | Sort according to date created (ascending order) |
| -date | Sort according to date created (descending order) |
| name | Sort according to application name (ascending order) |
| -name | Sort according to application name (descending order) |
| service_instance | Sort according to date created (ascending order) |
| -service_instance | Sort according to date created (descending order) |


For example: 

```bash
curl --location --request GET '127.0.0.1:8000/api/v1.0/calc/applications/?metadata_contains=Test&sort=-name' \
--header 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjozLCJ1c2VybmFtZSI6Im1hY2ludG9zaF93aG9sZXNhbGUiLCJleHAiOjE1ODIxMzM5MjcsImVtYWlsIjoiYWRtaW5AbWFjaGludG9zaC5jb20ifQ.RMO1U6ELAHN8ha3raVRrbCZHG1OC3dLarkCYwRX4mII' \
--header 'Content-Type: application/json'
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
    "description": "Finance department application",
    "metadata": "this can be any string up to 1024 characters long, for example, fookey=barvalue",
  },
  "alternative_option": {
    "components": [
      {
        "servicetype": "aze-standard_a4_v2",
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
        "servicetype": "aze-standard_f8s_v2",
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
    "description": "Finance department application",
    "metadata": "this can be any string up to 1024 characters long, for example, fookey=barvalue",
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
        "service_type": "aws-c5.xlarge",
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
        "service_type": "aws-c5.2xlarge",
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
        "servicetype": "aws-t3.xlarge",
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
        "servicetype": "aws-t3.xlarge",
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
    "description": "Finance department application",
    "metadata": "this can be any string up to 1024 characters long, for example, fookey=barvalue",
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
        "service_type": "aws-c5.xlarge",
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
        "service_type": "aws-c5.2xlarge",
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
        "servicetype": "aze-standard_b4ms",
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
        "servicetype": "aze-standard_f4",
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
    "description": "Finance department application",
    "metadata": "this can be any string up to 1024 characters long, for example, fookey=barvalue",
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
        "service_type": "aws-c5.xlarge",
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
        "service_type": "aws-c5.2xlarge",
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
        "servicetype": "aws-t3.xlarge",
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
        "servicetype": "aws-t3.xlarge",
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
    "description": "HR department application",
    "metadata": "Updated the metadata!"
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
    "description": "HR department application",
    "metadata": "Updated the metadata!"
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

### 1.2.9 Deleting multiple applications

Multiple applications may be deleted in one request.

```bash
curl -X POST \
  http://127.0.0.1:8000/api/v1.0/calc/applications/delete/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json' \
  -d '[
    "Enterprise_HR_SAAS_Application",
    "Enterprise_HR_SAAS_Application2"
  ]'
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

```bash
curl -X DELETE \
  http://127.0.0.1:8000/api/v1.0/calc/price/service_pk/45d7822c23fe69282e5640092775050834c1b7ec987d92d2a206c3a8/pricetype/examplecorp-aws-valued-customer-price/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json' \
```

...or using the price `pk`

```bash
curl -X DELETE \
  http://127.0.0.1:8000/api/v1.0/calc/price/pk/dc0c96587629b22e441020309b37227cb885632e2b09caba40cee5be/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json' \
```

A `204 No Content` is returned.

### 1.3.5 Deleting multiple prices (POST)

Multiple prices may be deleted in one request using the `pk`s of the prices.

```bash
curl -X POST \
  http://127.0.0.1:8000/api/v1.0/calc/price/delete/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json' \
  -d '[
    "dc0c96587629b22e441020309b37227cb885632e2b09caba40cee5be",
    "7cb885632e2b09caba40cee5bedc0c96587629b22e441020309b3722",
  ]'
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
{
  "count": 100,
  "next": null,
  "previous": null,
  "results": [
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
}
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

this returns a paginated list of prices

```json
{
  "count": 2,
  "next": null,
  "previous": null,
  "results": [
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
}
```

By default, this returns at most the first 10 prices. The response also contains URLs corresponding to the next and previous 'pages' if applicable, and a count of the total number of prices in the database.

The pagination scheme can be altered with the limit and offset parameters, like so:

```bash
curl -X GET \
  'http://127.0.0.1:8000/api/v1.0/calc/price/pricetype/examplecorp-aws-valued-customer-reduced-price/?limit=1&offset=0' \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json'
```

which would yield

```json
{
  "count": 2,
  "next": "http://127.0.0.1:8000/api/v1.0/calc/price/?limit=1&offset=1",
  "previous": null,
  "results": [
    {
      "service_name": "aws-t3.small",
      "pk": "789a89b8bcc4ce13552cfdd882cee4b130bbf1db0ad3d83b2a1600a9",
      "name": "examplecorp-aws-valued-customer-reduced-price",
      "service_pk": "45d7822c23fe69282e5640092775050834c1b7ec987d92d2a206c3a8",
      "costperunit": 0.001,
      "location": "us-east-2"
    }
  ]
}
```

In both cases, a `200 OK` response code is returned.

#### 1.3.5.4. Getting all prices

To get details of all prices stored in the database.

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

### 1.4.4 Deleting multiple service types (POST)

A user can delete multiple service types if they are all user-defined.

```bash
curl -X POST \
  http:127.0.0.1:8000/api/v1.0/calc/servicetype/delete/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json' \
  -d '[
    "e72e11be451152e0758d18248d700e05c36e0b0a4a1929f612fc6ea6",
    "b971a4a221ad29cfb3b0558e37689dfb460d590f108f45f7ea7dc751"
  ]'
```

### 1.4.5. Finding service types

#### 1.4.5.1. Service type keyword search

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

#### 1.4.5.2. Service type name autocomplete

The `serviceautocomplete` endpoint can be used for to autocomplete a partial name, demonstrated as follows:

Search for services prefixed by "aws-t3"

```bash
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/serviceautocomplete/aws-t3/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkyODkzMDAsImVtYWlsIjoiIn0.ESU0xx9I8MVuIhvN4eD02szSzz3bhi5D3CpCiKQQoFk' \
  -H 'Content-Type: application/json'
```

which returns

```json
[
  "aws-t3.2xlarge",
  "aws-t3a.2xlarge",
  "aws-t3a.large",
  "aws-t3a.medium",
  "aws-t3a.micro",
  "aws-t3a.nano",
  "aws-t3a.small",
  "aws-t3a.xlarge",
  "aws-t3.large",
  "aws-t3.medium",
  "aws-t3.micro",
  "aws-t3.nano",
  "aws-t3.small",
  "aws-t3.xlarge"
]
```

#### 1.4.5.3 Getting service type info

The `servicetypeinfo` endpoint can be used for to find locations, prices and providers for a service type

Search for information on "aws-t3.2xlarge"

```bash
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/servicetypeinfo/aws-t3%2E2xlarge/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkyODkzMDAsImVtYWlsIjoiIn0.ESU0xx9I8MVuIhvN4eD02szSzz3bhi5D3CpCiKQQoFk' \
  -H 'Content-Type: application/json'
```

which returns

```json
{
  "locations": [
    "ap-east-1",
    "ap-northeast-1",
    "ap-northeast-2",
    "ap-south-1",
    "ap-southeast-1",
    "ap-southeast-2",
    "ca-central-1",
    "eu-central-1",
    "eu-north-1",
    "eu-west-1",
    "me-south-1",
    "sa-east-1",
    "us-east-1",
    "us-east-2",
    "us-west-1"
  ],
  "prices": ["onDemandPrice", "reserved1yearPrice", "reserved3yearPrice"],
  "providers": ["aws"]
}
```

#### 1.4.5.4. List all service types

This API call returns a paginated list of all the service types form.

```bash
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/servicetype/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkyODkzMDAsImVtYWlsIjoiIn0.ESU0xx9I8MVuIhvN4eD02szSzz3bhi5D3CpCiKQQoFk' \
  -H 'Content-Type: application/json' \

```

yields

```json
{
  "count": 2,
  "next": null,
  "previous": null,
  "results": [
    {
      "name": "aze-standard_b2ms",
      "provider": "aze",
      "location": "canadacentral",
      "type": "compute",
      "unit": "",
      "attributes": [
        {
          "attr_id": "numcpus",
          "value": "2"
        },
        {
          "attr_id": "memory",
          "value": "8"
        }
      ],
      "pk_hash": "cbd3aac72de81ae1925f09c86c1b7f77bf6ee4c934fd41c35a8f77fc",
      "description": "Standard_B2ms General purpose",
      "version_date": "2019-11-03T20:21:41.185615Z",
      "user_defined": false
    },
    {
      "name": "aze-standard_f8",
      "provider": "aze",
      "location": "canadacentral",
      "type": "compute",
      "unit": "",
      "attributes": [
        {
          "attr_id": "numcpus",
          "value": "8"
        },
        {
          "attr_id": "memory",
          "value": "16"
        }
      ],
      "pk_hash": "1c51c76bd7e00a3bc32fe80b5937542abf22bd1e2c2d3ad4ab54b8e6",
      "description": "Standard_F8 Compute optimized",
      "version_date": "2019-11-03T20:21:41.697401Z",
      "user_defined": false
    }
  ]
}
```

By default, this returns at most the first 10 service types. The response also contains URLs corresponding to the next and previous 'pages' if applicable, and a count of the total number of service types in the database.

The pagination scheme can be altered with the limit and offset parameters, like so:

```bash
curl -X GET \
  'http://127.0.0.1:8000/api/v1.0/calc/servicetype/?limit=1&offset=0' \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkyODkzMDAsImVtYWlsIjoiIn0.ESU0xx9I8MVuIhvN4eD02szSzz3bhi5D3CpCiKQQoFk' \
  -H 'Content-Type: application/json' \
```

which would yield

```json
{
  "count": 2,
  "next": "http://127.0.0.1:8000/api/v1.0/calc/servicetype/?limit=1&offset=1",
  "previous": null,
  "results": [
    {
      "name": "aze-standard_b2ms",
      "provider": "aze",
      "location": "canadacentral",
      "type": "compute",
      "unit": "",
      "attributes": [
        {
          "attr_id": "numcpus",
          "value": "2"
        },
        {
          "attr_id": "memory",
          "value": "8"
        }
      ],
      "pk_hash": "cbd3aac72de81ae1925f09c86c1b7f77bf6ee4c934fd41c35a8f77fc",
      "description": "Standard_B2ms General purpose",
      "version_date": "2019-11-03T20:21:41.185615Z",
      "user_defined": false
    }
  ]
}
```

In both cases, a `200 OK` code is returned.

<div style="page-break-after: always;"></div>

## 1.5. ProviderDiscounts

Users may add discounts they receive from using certain providers. For example, a 30% price discount for machines that use amazon web services (AWS)

### 1.5.1. Create a new discount (POST)

Create new discount like so

```bash
curl -X POST \
  http://127.0.0.1:8000/api/v1.0/calc/discount/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json' \
  -d '{
    "provider": "aws",
    "discount": "30"
  }'
```

The new discount is returned as a JSON string.

```json
{
  "pk_hash": "b42385356271f4a9e3945abe2c3daff54d41af4ad0977394805437ec",
  "provider": {
    "name": "aws",
    "owner": "generic"
  },
  "discount": "30"
}
```

Note that discount percentages must be expressed as an integer between 0 and 100, and further that only one discount may exist for each provider. In the case that there is both a generic-owned and user-owned provider of the same name, the user-owned one will be used.

A `201 Created` code is returned.

### 1.5.2. Retrieve a discount (GET)

We can use the pk to get details of a discount.

```bash
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/discount/pk/e72e11be451152e0758d18248d700e05c36e0b0a4a1929f612fc6ea6/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json' \

```

yields

```json
{
  "pk_hash": "b42385356271f4a9e3945abe2c3daff54d41af4ad0977394805437ec",
  "provider": {
    "name": "aws",
    "owner": "generic"
  },
  "discount": "30"
}
```

A `200 OK` code is returned.

### 1.5.3. Updating the discount amount for a pre-existing discount (PUT)

the `discount` value can be updated for any provider discount(as long as it is "owned" - originally created - by the user).
The `pk` of the discount can be used as in the `get` request above.

```bash
curl -x put \
  http://127.0.0.1:8000/api/v1.0/calc/discount/pk/b42385356271f4a9e3945abe2c3daff54d41af4ad0977394805437ec/ \
  -h 'authorization: jwt eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -h 'content-type: application/json' \
  -d '{
    "provider": "aws",
    "discount": "40"
}'
```

the modified discount is returned

```json
{
  "pk_hash": "b42385356271f4a9e3945abe2c3daff54d41af4ad0977394805437ec",
  "provider": {
    "name": "aws",
    "owner": "generic"
  },
  "discount": "30"
}
```

a `200 ok` is returned on success.

### 1.5.4. Deleting a discount (DELETE)

A user can delete a discount for any provider.

```bash
curl -X DELETE \
  http://127.0.0.1:8000/api/v1.0/calc/discount/pk/e72e11be451152e0758d18248d700e05c36e0b0a4a1929f612fc6ea6/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkxMTAyOTUsImVtYWlsIjoiIn0.GvZaozS0NQ3e3B4MrGDzNIIP32nGXCpGBRNendUy2Kw' \
  -H 'Content-Type: application/json'

```

A `204 No Content` is returned.

### 1.5.5. Listing all discounts

This API call returns all discounts belonging to a user.

```bash
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/discount/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkyODkzMDAsImVtYWlsIjoiIn0.ESU0xx9I8MVuIhvN4eD02szSzz3bhi5D3CpCiKQQoFk' \
  -H 'Content-Type: application/json' \

```

Limit the size of the response using pagination, like so:

```bash
curl -X GET \
  'http://127.0.0.1:8000/api/v1.0/calc/discount/?limit=5&offset=4' \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImdlbmVyaWMiLCJleHAiOjE1NjkyODkzMDAsImVtYWlsIjoiIn0.ESU0xx9I8MVuIhvN4eD02szSzz3bhi5D3CpCiKQQoFk' \
  -H 'Content-Type: application/json' \
```

<div style="page-break-after: always;"></div>

## 1.6. Providers

Users can create custom providers. This may be useful in order to add servetypes corresponding to a private datacenter or cloud and adding customized costs to their applications. For example, a user may create a "provider" called `overheads` and add "servicetypes" for `hr_admin_overhead` and `network_cost_per_server`. Lets work through an example below.

First, create the provider

```bash
curl -X POST \
  http://127.0.0.1:8000/api/v1.0/calc/provider/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoyLCJ1c2VybmFtZSI6ImJic2VydmljZWFjY291bnQiLCJleHAiOjE1NzQ5NjIyMzIsImVtYWlsIjoiIn0.6ZmIczIXvers4SFRsOowZqBtMqpD9rHwzChH_shOUlE' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "overheads",
    "description": "Provider to capture custom overheads for any application"
}'
```

We can check that this `overheads` provider has been correctly created

```bash
curl -X GET \
  http://127.0.0.1:8000/api/v1.0/calc/provider/name/ove \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoyLCJ1c2VybmFtZSI6ImJic2VydmljZWFjY291bnQiLCJleHAiOjE1NzQ5NjIyMzIsImVtYWlsIjoiIn0.6ZmIczIXvers4SFRsOowZqBtMqpD9rHwzChH_shOUlE' \
  -H 'Content-Type: application/json'
```

```json
{
  "name": "overheads",
  "owner": "bbserviceaccount",
  "description": "Provider to capture custom overheads for any application",
  "short_name": "ove"
}
```

We note that the backend auto-assigns a 3-letter `short_name` to the new provider corresponding to the first 3 letters of the `name` (lowercased if needed). If we were to create a second provider with the same prefix (e.g. `overtheheads`) then the backend would choose a random 3-letter `short_name` and assign it to the second provider.

Optionally, a short_name may be explicitly specified as follows:

```bash
curl -X POST \
  http://127.0.0.1:8000/api/v1.0/calc/provider/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoyLCJ1c2VybmFtZSI6ImJic2VydmljZWFjY291bnQiLCJleHAiOjE1NzQ5NjIyMzIsImVtYWlsIjoiIn0.6ZmIczIXvers4SFRsOowZqBtMqpD9rHwzChH_shOUlE' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "overheads",
    "short_name": "ove",
    "description": "Provider to capture custom overheads for any application"
}'
```

Yielding the same successful response as above if the name is not already taken.

Next, lets create a service type called `hr_admin_overhead`

```bash
curl -X POST \
  http://127.0.0.1:8000/api/v1.0/calc/servicetype/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoyLCJ1c2VybmFtZSI6ImJic2VydmljZWFjY291bnQiLCJleHAiOjE1NzQ5NjIyMzIsImVtYWlsIjoiIn0.6ZmIczIXvers4SFRsOowZqBtMqpD9rHwzChH_shOUlE' \
  -H 'Content-Type: application/json' \
  -d '{
    "attributes": [
    ],
    "name": "hr_admin_overhead",
    "location": "anywhere",
    "type": "other",
    "unit": "per server",
    "provider": "ove",
    "description": "This servicetype embodies the human resource admin cost overhead per server"
}'
```

The POST request returns

```json
{
  "name": "hr_admin_overhead",
  "provider": "overheads",
  "location": "anywhere",
  "type": "other",
  "unit": "per server",
  "attributes": [],
  "pk_hash": "512c4c227fc31be9291f0c8e392dc930f1b1f9e0b20776df81e5a986",
  "description": "This servicetype embodies the human resource admin cost overhead per server",
  "version_date": "2019-11-28T16:12:07.011819Z",
  "user_defined": true
}
```

Next, lets add a custom price to the `hr_admin_overhead`. In this example the organization runs 1,000 servers with an IT team comprising of 15 people. Each team-member costs \$100,000 on average per year. So the HR admin overhead per server, per hour is

`(100000*15)/1000 = $1500` per server, per year; or about \$0.17 per server, per hour.

So lets post a custom price to the `hr_admin_overhead` service.

```bash
curl -X POST \
  http://127.0.0.1:8000/api/v1.0/calc/price/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoyLCJ1c2VybmFtZSI6ImJic2VydmljZWFjY291bnQiLCJleHAiOjE1NzQ5NjIyMzIsImVtYWlsIjoiIn0.6ZmIczIXvers4SFRsOowZqBtMqpD9rHwzChH_shOUlE' \
  -H 'Content-Type: application/json' \
  -d '{
    "service_name": "hr_admin_overhead",
    "name": "hr_overhead_cost",
    "service_pk": "512c4c227fc31be9291f0c8e392dc930f1b1f9e0b20776df81e5a986",
    "costperunit": 0.17,
    "location": "anywhere"
}'
```

Now we are ready to add the `hr_admin_overhead` to an application. In this case lets assume the application is running in AWS. The VMs will be selected from the Azure `aze` provider and the `hr_admin_overhead` will be selected from the `overheads` provider we just created. Like so

```bash
curl -X POST \
  http://127.0.0.1:8000/api/v1.0/calc/applications/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoyLCJ1c2VybmFtZSI6ImJic2VydmljZWFjY291bnQiLCJleHAiOjE1NzUwNDkxNTcsImVtYWlsIjoiIn0.PhWO2M7kcA280dCM26NjsGYGDOK4dnGFLY01KSz-SyM' \
  -H 'Content-Type: application/json' \
  -d '{
  "app_name": "User-application-in-AZURE, with overhead added",
  "description": "This is an application the customer currently runs in AZURE public cloud and has hr admin overheads",
  "services": [
    {
      "service_type": "aze-standard_f2",
      "location": "canadacentral",
      "price_type": "onDemandPrice",
      "provider": "aze",
      "quantity": 8,
      "attributes": [
        {
          "attr_id": "quantity",
          "value": 8
        }
      ]
    },
    {
      "service_type": "aze-standard_b8ms",
      "location": "eastus2",
      "price_type": "onDemandPrice",
      "provider": "aze",
      "quantity": 9,
      "attributes": [
        {
          "attr_id": "quantity",
          "value": 9
        }
      ]
    },
    {
      "service_type": "hr_admin_overhead",
      "location": "anywhere",
      "price_type": "hr_overhead_cost",
      "provider": "ove",
      "quantity": 17,
      "attributes": [
        {
          "attr_id": "quantity",
          "value": 17
        }
      ]
    }
  ]
}
'
```

Note the quantity has been set to `0.17` - since we calculated earlier that each server has a \$0.17 hr admin overhead per hour.

Next, lets got a cost report on the application

```bash
curl -X GET \
  'http://127.0.0.1:8000/api/v1.0/calc/reports/applications/app_name/User-application-in-AZURE,%20with%20overhead%20added/' \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoyLCJ1c2VybmFtZSI6ImJic2VydmljZWFjY291bnQiLCJleHAiOjE1NzUwNDkxNTcsImVtYWlsIjoiIn0.PhWO2M7kcA280dCM26NjsGYGDOK4dnGFLY01KSz-SyM' \
  -H 'Content-Type: application/json' \
```

This returns the following json

```json
{
  "application": {
    "app_name": "User-application-in-AZURE, with overhead added",
    "date_created": "2019-11-28T17:50:33.122670Z",
    "pk_hash": "9a2564fe4d32cec895cb0f1b5ae87c0550165baf40a088e25054d4cd",
    "services": [
      {
        "attributes": [
          {
            "attr_id": "quantity",
            "value": "8"
          }
        ],
        "service_type": "aze-standard_f2",
        "version_date": "2019-11-03T20:21:41.645718Z",
        "pk_hash": "f39943de1852e7746abfcd03dc8fec23cdc3666096b114cda98fcaf7",
        "location": "canadacentral",
        "price_type": "onDemandPrice",
        "provider": "aze"
      },
      {
        "attributes": [
          {
            "attr_id": "quantity",
            "value": "9"
          }
        ],
        "service_type": "aze-standard_b8ms",
        "version_date": "2019-11-03T20:22:29.255322Z",
        "pk_hash": "ed44b2fc3989778c09aa8dbe171708bb4a9bbadd4afa3c4e90644881",
        "location": "eastus2",
        "price_type": "onDemandPrice",
        "provider": "aze"
      },
      {
        "attributes": [
          {
            "attr_id": "quantity",
            "value": "17"
          }
        ],
        "service_type": "hr_admin_overhead",
        "version_date": "2019-11-28T17:50:23.796307Z",
        "pk_hash": "72456047a3bac812aa2269a352aaab0699a6fa3be61dc0331a1f8a49",
        "location": "anywhere",
        "price_type": "hr_overhead_cost",
        "provider": "overheads"
      }
    ],
    "description": "This is an application the customer currently runs in AZURE public cloud and has hr admin overheads"
  },
  "cost_components": [
    {
      "numcpus": "2",
      "memory": "4",
      "currentGen": "False",
      "quantity": "8",
      "service": "aze-standard_f2",
      "unit_cost": 0.104,
      "discount": 0,
      "unit": "",
      "location": "canadacentral",
      "service_cost": 0.83,
      "price_type": "onDemandPrice",
      "price_version_date": "2019-11-04T15:44:28.703193Z"
    },
    {
      "numcpus": "8",
      "memory": "32",
      "currentGen": "False",
      "quantity": "9",
      "service": "aze-standard_b8ms",
      "unit_cost": 0.333,
      "discount": 0,
      "unit": "",
      "location": "eastus2",
      "service_cost": 3.0,
      "price_type": "onDemandPrice",
      "price_version_date": "2019-11-03T20:22:29.258705Z"
    },
    {
      "quantity": "17",
      "service": "hr_admin_overhead",
      "unit_cost": 0.17,
      "discount": 0,
      "unit": "per server",
      "location": "anywhere",
      "service_cost": 2.89,
      "price_type": "hr_overhead_cost",
      "price_version_date": "2019-11-28T17:50:26.170275Z"
    }
  ],
  "total_cost": 6.720000000000001
}
```

The servicetype `hr_admin_overhead`, with quantity 17, is now a component of the total cost in the report.

We can also run the optimization or matching to another cloud provider as usual. In this case the admin cost is carried over as it is constant per server, no matter where the server is hosted.

For example, here we query porting the application to AWS.

```bash
curl -X GET \
  'http://127.0.0.1:8000/api/v1.0/calc/applications/app_name/User-application-in-AZURE,%20with%20overhead%20added/provider/aws/pricetype/reserved1yearPrice' \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoyLCJ1c2VybmFtZSI6ImJic2VydmljZWFjY291bnQiLCJleHAiOjE1NzUwNDkxNTcsImVtYWlsIjoiIn0.PhWO2M7kcA280dCM26NjsGYGDOK4dnGFLY01KSz-SyM' \
  -H 'Content-Type: application/json' \
```

And this yields

```json
{
  "application": {
    "app_name": "User-application-in-AZURE, with overhead added",
    "date_created": "2019-11-28T17:50:33.122670Z",
    "pk_hash": "9a2564fe4d32cec895cb0f1b5ae87c0550165baf40a088e25054d4cd",
    "services": [
      {
        "attributes": [
          {
            "attr_id": "quantity",
            "value": "8"
          }
        ],
        "service_type": "aze-standard_f2",
        "version_date": "2019-11-03T20:21:41.645718Z",
        "pk_hash": "f39943de1852e7746abfcd03dc8fec23cdc3666096b114cda98fcaf7",
        "location": "canadacentral",
        "price_type": "onDemandPrice",
        "provider": "aze"
      },
      {
        "attributes": [
          {
            "attr_id": "quantity",
            "value": "9"
          }
        ],
        "service_type": "aze-standard_b8ms",
        "version_date": "2019-11-03T20:22:29.255322Z",
        "pk_hash": "ed44b2fc3989778c09aa8dbe171708bb4a9bbadd4afa3c4e90644881",
        "location": "eastus2",
        "price_type": "onDemandPrice",
        "provider": "aze"
      },
      {
        "attributes": [
          {
            "attr_id": "quantity",
            "value": "17"
          }
        ],
        "service_type": "hr_admin_overhead",
        "version_date": "2019-11-28T17:50:23.796307Z",
        "pk_hash": "72456047a3bac812aa2269a352aaab0699a6fa3be61dc0331a1f8a49",
        "location": "anywhere",
        "price_type": "hr_overhead_cost",
        "provider": "overheads"
      }
    ],
    "description": "This is an application the customer currently runs in AZURE public cloud and has hr admin overheads"
  },
  "current_application": {
    "components": [
      {
        "numcpus": "2",
        "memory": "4",
        "currentGen": "False",
        "quantity": "8",
        "service_type": "aze-standard_f2",
        "unit_cost": 0.104,
        "discount": 0,
        "unit": "",
        "location": "canadacentral",
        "service_cost": 0.83,
        "price_type": "onDemandPrice",
        "price_version_date": "2019-11-04T15:44:28.703193Z"
      },
      {
        "numcpus": "8",
        "memory": "32",
        "currentGen": "False",
        "quantity": "9",
        "service_type": "aze-standard_b8ms",
        "unit_cost": 0.333,
        "discount": 0,
        "unit": "",
        "location": "eastus2",
        "service_cost": 3.0,
        "price_type": "onDemandPrice",
        "price_version_date": "2019-11-03T20:22:29.258705Z"
      },
      {
        "quantity": "17",
        "service": "hr_admin_overhead",
        "unit_cost": 0.17,
        "discount": 0,
        "unit": "per server",
        "location": "anywhere",
        "service_cost": 2.89,
        "price_type": "hr_overhead_cost",
        "price_version_date": "2019-11-28T17:50:26.170275Z"
      }
    ],
    "total_cost": 6.720000000000001
  },
  "alternative_option": {
    "components": [
      {
        "costph": 0.0263,
        "discount": 0,
        "quantity": "8",
        "servicetype": "aws-t3a.medium",
        "price_type": "reserved1yearPrice",
        "location": "ca-central-1",
        "service_cost": 0.21,
        "gpus": "0",
        "numcpus": "2",
        "memory": "4",
        "ntwPerf": "Low to Moderate",
        "instanceTypeCategory": "General purpose",
        "currentGen": "True",
        "predbogo": "N/A",
        "predcpu": "N/A",
        "tgzfilename": "N/A",
        "epoch-secs": "N/A",
        "minbogo": "N/A",
        "maxbogo": "N/A",
        "warning": "alternative for service aze-standard_f2 in canadacentral with reserved1yearPrice could not be found based on performance, response may be unchanged or contain alternative based on attributes instead."
      },
      {
        "costph": 0.1886,
        "discount": 0,
        "quantity": "9",
        "servicetype": "aws-t3a.2xlarge",
        "price_type": "reserved1yearPrice",
        "location": "us-east-2",
        "service_cost": 1.7,
        "gpus": "0",
        "numcpus": "8",
        "memory": "32",
        "ntwPerf": "Moderate",
        "instanceTypeCategory": "General purpose",
        "currentGen": "True",
        "predbogo": "N/A",
        "predcpu": "N/A",
        "tgzfilename": "N/A",
        "epoch-secs": "N/A",
        "minbogo": "N/A",
        "maxbogo": "N/A",
        "warning": "alternative for service aze-standard_b8ms in eastus2 with reserved1yearPrice could not be found based on performance, response may be unchanged or contain alternative based on attributes instead."
      },
      {
        "costph": 0.17,
        "error": "alternative for service hr_admin_overhead in anywhere with reserved1yearPrice not found, partial data/costs returned",
        "discount": 0,
        "quantity": "17",
        "servicetype": "hr_admin_overhead",
        "price_type": "reserved1yearPrice",
        "location": "anywhere",
        "service_cost": 2.89,
        "warning": "alternative for service hr_admin_overhead in anywhere with reserved1yearPrice could not be found based on performance, response may be unchanged or contain alternative based on attributes instead."
      }
    ],
    "total_cost": 4.8
  }
}
```

Custom Providers, such as the `overheads` provider we created above, can be can be deleted too, but only after all applications with servicetypes from the provider have been deleted. Otherwise the API will not allow custom provider deletion.

```bash
curl -X DELETE \
  http://127.0.0.1:8000/api/v1.0/calc/provider/name/ove/ \
  -H 'Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoyLCJ1c2VybmFtZSI6ImJic2VydmljZWFjY291bnQiLCJleHAiOjE1NzUwNDkxNTcsImVtYWlsIjoiIn0.PhWO2M7kcA280dCM26NjsGYGDOK4dnGFLY01KSz-SyM' \
  -H 'Content-Type: application/json' \
```

This DELETE request returns `204 No Content`.

<div style="page-break-after: always;"></div>

**Table of Contents**

- [BigBitBus API Version 1](#bigbitbus-api-version-1)
  - [1.1. Authorization](#11-authorization)
  - [1.2. Applications](#12-applications)
    - [1.2.1. Create an application](#121-create-an-application)
    - [1.2.2. Retrieving Applications](#122-retrieving-applications)
    - [1.2.3. Obtaining a report on an Application](#123-obtaining-a-report-on-an-application)
    - [1.2.4. Querying Applications](#124-querying-applications)
      - [1.2.4.1 Querying Applications using ApplicationList](#1241-querying-applications-using-applicationlist)
        - [Filters](#filters)
        - [Sort Criteria](#sort-criteria)
    - [1.2.5. Finding Application Alternatives on Another Provider](#125-finding-application-alternatives-on-another-provider)
    - [1.2.6. VM Optimization](#126-vm-optimization)
    - [1.2.7. Updating an Application](#127-updating-an-application)
    - [1.2.8. Deleting an Application](#128-deleting-an-application)
    - [1.2.9 Deleting multiple applications](#129-deleting-multiple-applications)
  - [1.3. Prices](#13-prices)
    - [1.3.1. Creating a new price for a service (POST)](#131-creating-a-new-price-for-a-service-post)
    - [1.3.2. Get price details about a pre-existing price (GET)](#132-get-price-details-about-a-pre-existing-price-get)
    - [1.3.3. Updating the costperunit for a pre-existing price (PUT)](#133-updating-the-costperunit-for-a-pre-existing-price-put)
    - [1.3.4. Deleting a price (DELETE)](#134-deleting-a-price-delete)
    - [1.3.5 Deleting multiple prices (POST)](#135-deleting-multiple-prices-post)
    - [1.3.5. Filtering Price data](#135-filtering-price-data)
      - [1.3.5.1. Getting all prices for a service type](#1351-getting-all-prices-for-a-service-type)
      - [1.3.5.2 Get the price of a ServiceType (specified by its service_pk) of a certain pricetype](#1352-get-the-price-of-a-servicetype-specified-by-its-servicepk-of-a-certain-pricetype)
      - [1.3.5.3. Getting all prices of a certain pricetype](#1353-getting-all-prices-of-a-certain-pricetype)
      - [1.3.5.4. Getting all prices](#1354-getting-all-prices)
  - [1.4. ServiceTypes](#14-servicetypes)
    - [1.4.1. Create a new service type (POST)](#141-create-a-new-service-type-post)
    - [1.4.2. Retrieve a service type (GET)](#142-retrieve-a-service-type-get)
    - [1.4.3. Deleting a service type (DELETE)](#143-deleting-a-service-type-delete)
    - [1.4.4 Deleting multiple service types (POST)](#144-deleting-multiple-service-types-post)
    - [1.4.5. Finding service types](#145-finding-service-types)
      - [1.4.5.1. Service type keyword search](#1451-service-type-keyword-search)
      - [1.4.5.2. Service type name autocomplete](#1452-service-type-name-autocomplete)
      - [1.4.5.3 Getting service type info](#1453-getting-service-type-info)
      - [1.4.5.4. List all service types](#1454-list-all-service-types)
  - [1.5. ProviderDiscounts](#15-providerdiscounts)
    - [1.5.1. Create a new discount (POST)](#151-create-a-new-discount-post)
    - [1.5.2. Retrieve a discount (GET)](#152-retrieve-a-discount-get)
    - [1.5.3. Updating the discount amount for a pre-existing discount (PUT)](#153-updating-the-discount-amount-for-a-pre-existing-discount-put)
    - [1.5.4. Deleting a discount (DELETE)](#154-deleting-a-discount-delete)
    - [1.5.5. Listing all discounts](#155-listing-all-discounts)
  - [1.6. Providers](#16-providers)
