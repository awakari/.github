# Contents

1. [Overview](#1-overview)<br/>
2. [Concepts](#2-concepts)<br/>
3. [Design](#3-design)<br/>
4. [Additional Information](#4-additional-information)<br/>

# 1. Overview

Awakari is a publish/subscribe based news filter and reader.
On the other hand, Awakari may be treated as a prospective search engine continuously capturing new relevant events.

# 2. Concepts

## 2.1. Publishing

### 2.2.1. Events

Event is an entity being routed and delivered by Awakari.
The accepted format is [Cloud Events](https://cloudevents.io).

[API documentation to publish events](https://pub.awakari.com/swagger/index.html#/Events)

## 2.2.2. Sources

A publishing source, or, sometimes just *source* is an entity that publishes new events.
There are multiple source integrations, producing events from:
* Web feeds (like RSS)
* [Fediverse publishers](https://github.com/awakari/int-activitypub)
* [Bluesky publishers](https://github.com/awakari/int-bluesky)
* [Telegram channels](https://github.com/awakari/source-telegram)
* [Email digests](https://github.com/awakari/int-email)
* Site updates

There are sources that push events to Awakari and sources that are being periodically checked for new events (polling).
Any source also converts an input (feed entry, publisher's post, etc) to the internal format accepted by Awakari.

[API documentation](https://pub.awakari.com/swagger/index.html#/Sources)

## 2.2. Subscriptions

Subscription is a combination of a specific interest and delivery address.

API documentation:
* [Subscriptions](https://reader.awakari.com/swagger/index.html#/Subscriptions)
* [Feeds](https://reader.awakari.com/swagger/index.html#/Feeds)

## 2.3. Interests

Interest is a set of [matching conditions](#231-matching-conditions) linked to a specific user.

[API documentation](https://interests.awakari.com/swagger/index.html)

### 2.3.1. Matching Conditions

A condition represents an event matching criteria. 
Currently, Awakari supports the following condition types: 
* text matching
  * keywords
  * exact
  * semantic
* numeric comparison (<, ≤, =, ≥, >)
* groups of nested condtions with logic (And, Or, Xor)

Any condition may be negative ("Not").
The matching is structured by design: text and numeric conditions may be used to match a certain attribute of an event.

# 3. Design

## 3.1. Data Schema

Some well-known event attributes:

| Attribute             | Type          | Description                     | Known Usage                                                                  |                  
|-----------------------|---------------|---------------------------------|------------------------------------------------------------------------------|
| `action`              | string        | TODO                            | TODO                                                                         |
| `attachmentlength`    | string        | TODO                            | TODO                                                                         |
| `attachmenttype`      | string        | TODO                            | TODO                                                                         |
| `attachmenturl`       | string, uri   | TODO                            | TODO                                                                         |
| `awakarigroupid`      | string        | group id                        | set by resolver from the "X-Awakari-Group-Id" gRPC header                    |
| `awakarimatchfound`   | bool          |                                 | internal, set and used by resolver                                           |
| `awakariregistered`   | bool          |                                 | internal, set and used by resolver                                           |
| `awakarispanid`       | string        | tracing span id                 | set and used by core components when tracing is enabled                      |
| `awakaritraceid`      | string        | trace id                        | set and used by core components when tracing is enabled                      |
| `awakariuserid`       | string        | user id                         | set by resolver from the "X-Awakari-User-Id" gRPC header, removed by reader  |
| `categories`          | string        | space-separated item categories | set by various sources, all social post hashtags go here                     |
| `copyright`           | string        | Copyright info                  | set by various sources                                                       |
| `data`                | string        | event payload, mostly text      | set by any source, may be empty                                              |
| `description`         | string        | description                     | set by any source, may be empty                                              |
| `feedcategories`      | string        | space-separated feed categories | set by source-feeds                                                          |
| `feeddescription`     | string        |                                 | set by source-feeds                                                          |
| `feedimagetitle`      | string        |                                 | set by source-feeds                                                          |
| `feedimageurl`        | uri           |                                 | set by source-feeds                                                          |
| `feedtitle`           | string        |                                 | set by source-feeds                                                          |
| `feedurl`             | uri           |                                 | set by source-feeds                                                          |
| `id`                  | string        | event id                        | set by any source to a unique value                                          |
| `imagetitle`          | string        |                                 | set by source-feeds                                                          |
| `imageurl`            | uri           |                                 | set by source-feeds                                                          |
| `language`            | string        |                                 | set by source-feeds                                                          |
| `latitude`            | string        | Location Latitude               | set by various sources                                                       |
| `longitude`           | string        | Location Longitude              | set by various sources                                                       |
| `magnitude`           | string        | Earthquake magnitude            | set by source-websocket                                                      |
| `offersprice`         | string, int32 | Price                           | set by various sources                                                       |
| `offerspricecurrency` | string        | Price currency                  | set by various sources                                                       |
| `offersurl`           | string        | Offer's URL                     | TODO                                                                         |
| `sentiment`           | int32         | Summary sentiment               | set by pub, rang is from -100 to 100                                         |
| `sentimentnegative`   | int32         | Negative sentiment              | set by pub, rang is from 0 to 100                                            |
| `sentimentneutral`    | int32         | Neutral sentiment               | set by pub, rang is from 0 to 100                                            |
| `sentimentpositive`   | int32         | Positive sentiment              | set by pub, rang is from 0 to 100                                            |
| `source`              | string        |                                 | telegram: "@channel1", others: source link URL                               |
| `specversion`         | string        | always "1.0"                    | set by any source                                                            |
| `snippet`             | string        | summary                         | set by pub, used internally for the sentiment analysis and semantic matching |
| `subject`             | string        | author info                     |                                                                              |
| `summary`             | string        |                                 | set by source-feeds                                                          |
| `tgfileid`            | string        | telegram file id                | set and used by bot-telegram                                                 |
| `tgfileimgheight`     | int32         | telegram image height           | set and used by bot-telegram                                                 |
| `tgfileimgwidth`      | int32         | telegram image width            | set and used by bot-telegram                                                 |
| `tgfilemediaduration` | int32         | telegram media duration         | set and used by bot-telegram                                                 |
| `tgfiletype`          | int32         | telegram file type              | set and used by bot-telegram                                                 | 
| `tgfileuniqueid`      | string        | telegram file unique id         | set and used by bot-telegram                                                 |
| `tgmessageid`         | string        | telegram message id             | internal, set by bot-telegram                                                |
| `time`                | timestamp     |                                 | set by any source                                                            |
| `title`               | string        |                                 | set by source-feeds                                                          |
| `type`                | string        | various values                  | set by any source                                                            |

# 4. Additional Information

* [Source Opt-Out Methods](../OPT-OUT.md)
* [Roadmap](../ROADMAP.md)
* [Contributing](../CONTRIBUTING.md)

## 4.1. API access

Using the Awakari API requires the authentication token (request it by [email](mailto:awakari@awakari.com)).
