# Contents

1. [Overview](#1-overview)<br/>
2. [Concepts](#2-concepts)<br/>
3. [Access](#3-access)<br/>
4. [Usage](#4-usage)<br/>
5. [Design](#5-design)<br/>
6. [Additional Information](#6-additional-information)<br/>

# 1. Overview

Awakari: real-time search results alerting service.

# 2. Concepts

## 2.1. Condition

An alert condition represents an event matching criteria.
Currently, Awakari supports the following condition types: 
* text matching
* numeric comparison (<, ≤, =, ≥, >)
* groups of nested condtions with logic (And, Or, Xor)

Any condition may be negative ("Not").
The matching is structured by design: text and numeric conditions may be used to match a certain attribute of an event.

## 2.2. Subscription

Subscription is a set of matching conditions associated with a specific user (i.e. alert).

## 2.3. Event

The entity being routed and delivered by Awakari. 
The accepted format is [Cloud Events](https://cloudevents.io).
There are multiple source integrations, producing events from: 
* Web feeds (like RSS)
* Fediverse publishers
* Telegram channels

# 3. Access

Using the Awakari cloud API requires mutual TLS authentication and encryption to secure all client data.
Hence, to access the cloud API it's necessary to have a client certificate.

Prepare own client certificate request:
```shell
openssl req -new -newkey rsa:4096 -nodes \
  -keyout client.key \
  -out client.csr \
  -addext "subjectAltName=DNS:api.awakari.com" \
  -subj '/CN=group0.company1.com'
```

> **Warning**
> Never specify additional certificate attributes like "O", "OU", etc.
> The resulting DN should not contain commas.

Then request the client certificate (currently by [email](mailto:awakari@awakari.com)).

# 4. Usage

> [!IMPORTANT]
> The documentation below is applicable to the [Serverless](https://awakari.com/#solutions-hosting) hosting option.
> For the Hybrid option usage refer to [Core Usage](https://github.com/awakari/core#4-usage).

## 4.1. Client SDK

Refer to [Client SDK Usage](https://github.com/awakari/client-sdk-go#3-usage).

## 4.2. API

Preparation steps:

1. Install [grpcurl](https://github.com/fullstorydev/grpcurl)
2. Download the necessary proto files and save to the current directory:
    1. [Cloud Events](https://awakari.com/proto/cloudevent.proto)
    2. [Reader](https://awakari.com/proto/reader.proto)
    3. [Resolver](https://awakari.com/proto/resolver.proto)

### 4.2.1. Usage 

#### 4.2.1.2. Limits

Usage limit represents the successful API call count limit. The limit is identified per:
* group id
* user id (optional)
* subject

There are the group-level limits where user id is not specified. All users from the group share the group limit in this
case.

Usage subject may be one of:
* Subscriptions (code 1)
* Publish Events (code 2)

Check current subscriptions limits (`"subj": 1`):
```shell
grpcurl \
  -key test0.client0.key \
  -cert test0.client0.crt \
  -cacert ca.crt \
  -H 'X-Awakari-User-Id: john.doe@company1.com' \
  -d '{"subj": 1}' \
  api.awakari.com:443 \
  awakari.api.limits.Service/Get
```

A successfull response looks like:
```json
{
  "count": "1000"
}
```

> [!NOTE]
> The empty `userId` attribute in the response means the usage limit is group-level limit.

#### 4.2.1.2. Permits

Usage permits represents the current usage statistics (counters) by the subject. Similar to usage limit, the counters
represent the group-level usage when the user id is empty.

User-specific events publishing permits checking command example:
```shell
grpcurl \
  -key test0.client0.key \
  -cert test0.client0.crt \
  -cacert ca.crt \
  -H 'X-Awakari-User-Id: john.doe@company1.com' \
  -d '{"subj": 2}' \
  api.awakari.com:443 \
  awakari.api.permits.Service/GetUsage
```

> [!NOTE]
> Skip the "X-Awakari-User-Id" header to get the group-level permits.

### 4.2.2. Subscriptions

Create:
```shell
grpcurl \
  -key test0.client0.key \
  -cert test0.client0.crt \
  -cacert ca.crt \
  -H 'X-Awakari-User-Id: john.doe@company1.com' \
  -d @ \
  api.awakari.com:443 \
  awakari.subscriptions.proxy.Service/Create
```

Example payload:
```json
{
   "description": "Tesla model S updates",
   "enabled": true,
   "cond": {
      "not": false,
      "tc": {
        "key": "",
        "term": "Tesla Model S",
        "exact": false
      }
    }
}
```

A successful response contains the created subscription id:
```json
{
  "id": "547857e3-adfc-48a5-a49e-110cfdedbaab"
}
```

Note the created subscription id and use it further to read the events.
Learn more about the [Subscriptions API](https://github.com/awakari/client-sdk-go/blob/master/api/grpc/subscriptions/service.proto).

### 4.2.3. Events

#### 4.2.3.1. Read

```shell
grpcurl \
  -key test0.client0.key \
  -cert test0.client0.crt \
  -cacert ca.crt \
  -proto reader.proto \
  -max-time 86400 \
  -H 'X-Awakari-User-Id: john.doe@company1.com' \
  -d @ \
  api.awakari.com:443 \
  awakari.reader.Service/Read
```

Specify the subscription id in the payload:
```json
{"start": {"batchSize": 1, "subId": "547857e3-adfc-48a5-a49e-110cfdedbaab"}}
```

This starts a reader stream. A new event appear in the response once system receives anything matching the subscription.
Leave this shell/window open and switch to another. Later switch back and check for new events received.

It's necessary to acknowledge every received event:
```json
{"ack": { "count": 1}}
```

#### 4.2.3.2. Write

```shell
grpcurl \
  -key test0.client0.key \
  -cert test0.client0.crt \
  -cacert ca.crt \
  -proto writer.proto \
  -H 'X-Awakari-User-Id: john.doe@company1.com' \
  -d @ \
  api.awakari.com:443 \
  awakari.resolver.Service/SubmitMessages
```

Specify the events to write in the payload:
```json
{
   "msgs": [
      {
         "id": "3426d090-1b8a-4a09-ac9c-41f2de24d5ac",
         "type": "example.type",
         "source": "example/uri",
         "spec_version": "1.0",
         "attributes": {
            "subject": {
               "ce_string": "Tesla price updates"
            },
            "time": {
               "ce_timestamp": "2023-07-03T23:20:50.52Z"
            }
         },
         "text_data": "Tesla model S is now available at lower price"
      }
   ]
}
```

A successful response looks like:
```json
{
  "ackCount": 1
}
```

After this it's possible to submit more events.

When finished, close the writer stream by pressing ^C or leave it open to publish any other events later.

# 5. Design

## 5.1. Data Schema

| Attribute             | Type       | Description                         | Known Usage                                                                 |                  
|-----------------------|------------|-------------------------------------|-----------------------------------------------------------------------------|
| `author`              | string     | 1st author info, name and email     | set by source-feeds                                                         |
| `awakarigroupid`      | string     | group id                            | set by resolver from the "X-Awakari-Group-Id" gRPC header                   |
| `awakarimatchfound`   | bool       |                                     | internal, set and used by resolver                                          |
| `awakariregistered`   | bool       |                                     | internal, set and used by resolver                                          |
| `awakarispanid`       | string     | tracing span id                     | set and used by core components when tracing is enabled                     |
| `awakaritraceid`      | string     | trace id                            | set and used by core components when tracing is enabled                     |
| `awakariuserid`       | string     | user id                             | set by resolver from the "X-Awakari-User-Id" gRPC header, removed by reader |
| `categories`          | string     | space-separated item categories     | set by source-feeds, pub wizard                                             |
| `contact`             | string     | Contact info to reply               | set by pub wizard                                                           |
| `currency`            | string     | Currency, `USD`, `EUR`, ...         | set by pub wizard                                                           |
| `data`                | string     | event payload, mostly text          | set by any source, may be empty                                             |  
| `feedcategories`      | string     | space-separated feed categories     | set by source-feeds                                                         |
| `feeddescription`     | string     |                                     | set by source-feeds                                                         |
| `feedimagetitle`      | string     |                                     | set by source-feeds                                                         |
| `feedimageurl`        | uri        |                                     | set by source-feeds                                                         |
| `feedtitle`           | string     |                                     | set by source-feeds                                                         |
| `feedurl`             | uri        |                                     | set by source-feeds                                                         |
| `id`                  | string     | event id                            | set by any source to a unique value                                         |
| `imagetitle`          | string     |                                     | set by source-feeds                                                         |
| `imageurl`            | uri        |                                     | set by source-feeds                                                         |
| `language`            | string     |                                     | set by source-feeds                                                         |
| `latitude`            | string     | Location Latitude                   | set by various sources                                                      |
| `longitude`           | string     | Location Longitude                  | set by various sources                                                      |
| `pricemax`            | string     | Max total price                     | set by pub wizard                                                           |
| `pricemin`            | string     | Min total price                     | set by pub wizard                                                           |
| `quantitymax`         | string     | Max quantity                        | set by pub wizard                                                           |
| `quantitymin`         | string     | Min quantity                        | set by pub wizard                                                           |
| `quantityunit`        | string     | Quantity units, may be empty        | set by pub wizard                                                           |
| `source`              | string     |                                     | bot-telegram: "@AwakariBot", others: source link URL                        |
| `specversion`         | string     | meaningless, always "1.0"?          | set by any source                                                           |
| `subject`             | string     |                                     | source-feeds: RSS item guid, source-sites: site URL                         |
| `summary`             | string     |                                     | set by source-feeds                                                         |
| `tags`                | string     | space separated tags                | set by pub wizard                                                           |
| `tgfileid`            | string     | telegram file id                    | set and used by bot-telegram                                                |
| `tgfileimgheight`     | int32      | telegram image height               | set and used by bot-telegram                                                |
| `tgfileimgwidth`      | int32      | telegram image width                | set and used by bot-telegram                                                |
| `tgfilemediaduration` | int32      | telegram media duration             | set and used by bot-telegram                                                |
| `tgfiletype`          | int32      | telegram file type                  | set and used by bot-telegram                                                | 
| `tgfileuniqueid`      | string     | telegram file unique id             | set and used by bot-telegram                                                |
| `tgmessageid`         | string     | telegram message id                 | internal, set by bot-telegram                                               |
| `time`                | timestamp  |                                     | set by any source                                                           |
| `title`               | string     |                                     | set by source-feeds, pub wizard                                             |
| `type`                | string     | various values                      | set by any source, pub wizard                                               |

# 6. Additional Information

* [Roadmap](ROADMAP.md)
* [Contributing](CONTRIBUTING.md)
