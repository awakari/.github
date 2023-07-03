# Contents

1. [Overview](#1-overview)<br/>
   1.1. [Comparison](#11-comparison)<br/>
   1.2. [Purpose](#12-purpose)<br/>
   1.3. [Concepts](#13-concepts)<br/>
2. [Deployment](#2-deployment)<br/>
3. [Usage](#3-usage)<br/>
4. [Configuration](#4-configuration)<br/>
5. [Design](#5-design)<br/>
6. [Additional Infromation](#6-additional-information)<br/>

# 1. Overview

Awakari is an event-driven publish/subscribe system.

Publish events to the common stream.
Define comprehensive search conditions as a subscription.
Awakari will push the relevant events as notifications.

Awakari works with:
* Streams of [Cloud Events](https://cloudevents.io/).
* Subscriptions with sets of matching conditions.

## 1.1. Comparison

The closest Awakari analogue is a [Percolate Query in Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-percolate-query.html).

The key difference is that Awakari pushes the new matching events when user is online through a gRPC stream.

## 1.2. Purpose

### 1.2.1. Manage Subscriptions

Define custom message matching conditions:

#### 1.2.1.1. Arbitrary Attributes

Specify custom keys to match: 
* category
* location
* language
* ...

#### 1.2.1.2. Full-Text Search

Specify text search terms to match

#### 1.2.1.3. Composite Conditions

Groups of nested conditions and groups

#### 1.2.1.4. Logic Functions

* And
* Or
* Xor
* Not


### 1.2.2. Receive Push Notifications 

Receive new messages timely and exactly once.

### 1.2.3. Broadcast

Publish info, everyone interested in it will receive it.

### 1.2.4. Integrate

Publish and receive messages using RSS feeds, Chat Bots or message queues.
Leverage Machine Learning algorithms to classify and label the image, audio or video data when publishing.

## 1.3. Concepts

Awarkari works with events and subscriptions.

### 1.3.1. Matching Condition

A condition represents a message matching criteria. [Learn more](https://github.com/awakari/subscriptions#121-condition)

### 1.3.2. Subscription

Subscriptions is a set of matching conditions associated with a specific user.
[Learn more](https://github.com/awakari/subscriptions#122-subscription)

### 1.3.3. Event

The entity being routed and delivered by Awakari. The accepted format is [Cloud Events](https://cloudevents.io)

# 2. Deployment

There are two deployment options:
1. [Self-Hosted Core](https://github.com/awakari/core#3-deployment)
2. Cloud SaaS:
   1. `demo.awakari.cloud:443`

![components](components.png)

# 3. Additional Information

* [Roadmap](ROADMAP.md)
* [Contributing](CONTRIBUTING.md)
