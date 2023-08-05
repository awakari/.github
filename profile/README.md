# Contents

1. [Overview](#1-overview)<br/>
2. [Concepts](#2-concepts)<br/>
3. [Additional Information](#3-additional-information)<br/>

# 1. Overview

Awakari is a service that combines publish/subscribe capability with real-time search.

# 2. Concepts

Awarkari works with events and subscriptions.

## 2.1. Matching Condition

A condition represents a message matching criteria.

## 2.2. Subscription

Subscriptions is a set of matching conditions associated with a specific user.

## 2.3. Event

The entity being routed and delivered by Awakari. The accepted format is [Cloud Events](https://cloudevents.io)

# 3. Additional Information

* [Roadmap](ROADMAP.md)
* [Contributing](CONTRIBUTING.md)

# 4. Usage

> [!IMPORTANT]
> The documentation below is applicable to the [Serverless](https://awakari.com/#solutions-hosting) hosting option.
> For the Hybrid option usage refer to [Core Usage](https://github.com/awakari/core#4-usage)

## 4.1. Client SDK

Refer to [Client SDK Usage](https://github.com/awakari/client-sdk-go#3-usage).

## 4.2. API

### 4.2.1. Preparation

1. Install [grpcurl](https://github.com/fullstorydev/grpcurl)
2. Download the necessary proto files and save to the current directory:
    1. [Cloud Events](https://awakari.com/proto/cloudevent.proto)
    2. [Reader](https://awakari.com/proto/reader.proto)
    3. [Resolver](https://awakari.com/proto/resolver.proto)
