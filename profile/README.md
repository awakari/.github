# Contents

1. [Overview](#1-overview)<br/>
2. [Concepts](#2-concepts)<br/>
3. [Access](#3-access)<br/>
4. [Usage](#4-usage)<br/>
5. [Additional Information](#5-additional-information)<br/>

# 1. Overview

Awakari is a service that combines publish/subscribe capability with real-time search.

# 2. Concepts

Awarkari works with events and subscriptions.

## 2.1. Matching Condition

A condition represents a message matching criteria. 
Currently, Awakari supports the following condition types: 
* text matching
* numeric comparison (<, ≤, =, ≥, >)
* groups of nested condtions with logic (And, Or, Xor)

## 2.2. Subscription

Subscriptions is a set of matching conditions associated with a specific user.

## 2.3. Event

The entity being routed and delivered by Awakari. The accepted format is [Cloud Events](https://cloudevents.io).

# 3. Access

Using the Awakari cloud API requires mutual TLS authentication and encryption to secure all client data.
Hence, to access the cloud API it's necessary to have a client certificate.

For the demo purposes, there is the cloud instance `demo.awakari.cloud` available.
Ready-to-use demo client certificates are in the [certs/demo](certs/demo) directory.

For production usage, prepare own client certificate request:
```shell
openssl req -new -newkey rsa:4096 -nodes \
  -keyout client.key \
  -out client.csr \
  -addext "subjectAltName=DNS:awakari.cloud" \
  -subj '/CN=group0.company1.com'
```

> **Warning**
> Never specify additional certificate attributes like "O", "OU", etc.
> The resulting DN should not contain commas.

Then request the client certificate (currently by email).

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
  demo.awakari.cloud:443 \
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
  demo.awakari.cloud:443 \
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
  demo.awakari.cloud:443 \
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
  demo.awakari.cloud:443 \
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
  demo.awakari.cloud:443 \
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

# 5. Additional Information

* [Roadmap](ROADMAP.md)
* [Contributing](CONTRIBUTING.md)
