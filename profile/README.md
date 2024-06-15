# Contents

1. [Overview](#1-overview)<br/>
2. [Concepts](#2-concepts)<br/>
3. [Access](#3-access)<br/>
4. [Usage](#4-usage)<br/>
5. [Design](#5-design)<br/>
6. [Additional Information](#6-additional-information)<br/>

# 1. Overview

Awakari is an events filter.
The purpose is to allow users to follow their interests instead of subjective sources. 

# 2. Concepts

As a publish/subscribe service it works with events and interest subscriptions.
Awakari performs a continuous prospective search capturing relevant events in the events stream on the fly.

## 2.1. Matching Condition

A condition represents a message matching criteria. 
Currently, Awakari supports the following condition types: 
* text matching
* numeric comparison (<, ≤, =, ≥, >)
* groups of nested condtions with logic (And, Or, Xor)

Any condition may be negative ("Not").
The matching is structured by design: text and numeric conditions may be used to match a certain attribute of an event.

## 2.2. Interest

Interest is a set of matching conditions associated with a specific user.
Internally interest is named subscription.

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

TODO

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

* [Opt-Out Methods](../OPT-OUT.md)
* [Roadmap](../ROADMAP.md)
* [Contributing](../CONTRIBUTING.md)
