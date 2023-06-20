# Contents

1. [Overview](#1-overview)<br/>
   1.1. [Comparison](#11-comparison)<br/>
   1.2. [Purpose](#12-purpose)<br/>
   1.3. [Definitions](#13-definitions)<br/>
2. [Deployment](#2-deployment)<br/>
3. [Usage](#3-usage)<br/>
4. [Configuration](#4-configuration)<br/>
5. [Design](#5-design)<br/>
6. [Additional Infromation](#6-additional-information)<br/>

# 1. Overview

Awakari is a messaging pub/sub service.

In the modern world of information streams its getting harder to find what is important timely. Conventional search engines, social networks and messengers doesn't solve this problem.

Awakari offers comprehensive subscriptions to catch the relevant messages. It allows users to publish their messages to the common stream. Other users decide, what they want to receive.

Awakari works with:
* Streams of [Cloud Events](https://cloudevents.io/).
* Subscriptions with sets of matching conditions.

## 1.1. Comparison

The key differences from other messaging solutions are:
<table>
    <thead>
        <tr>
            <td rowspan="2" colspan="2"><b>Criteria</b></td>
            <td colspan="4" align="center"><b>Solution</b></td>
        </tr>
        <tr>
            <td align="center" valign="top"><b>Awakari</b></td>
            <td align="center" valign="top"><b>Kafka</b></td>
            <td align="center" valign="top"><b>Nats<br/>(JetStream)</b></td>
            <td align="center" valign="top"><b>AWS<br/>SNS/SQS</b></td>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="2">Matching Patterns</td>
            <td>Horizontal Scalability</td>
            <td align="center"><img width="16px" src="icon-yes.svg" title=""/></td>
            <td align="center"><img width="16px" src="icon-no.svg" title="consumer-side topic matching"/></td>
            <td align="center"><img width="16px" src="icon-yes.svg" title=""/></td>
            <td align="center"><img width="16px" src="icon-no.svg" title="not applicable"/></td>
        </tr>
        <tr>
            <td>Matching Time Complexity</td>
            <td align="center"><img width="16px" src="icon-yes.svg" title="O(log(N)) for kiwi-tree subscriptions"/></td>
            <td align="center"><img width="16px" src="icon-no.svg" title="O(N)"/></td>
            <td align="center"><img width="16px" src="icon-no.svg" title="O(N)"/></td>
            <td align="center"><img width="16px" src="icon-no.svg" title="not applicable"/></td>
        </tr>
        <tr>
            <td rowspan="2">Matching Criteria</td> 
            <td>Arbitrary Attributes</td>
            <td align="center"><img width="16px" src="icon-yes.svg" title="Any metadata (key/value)"/></td>
            <td align="center"><img width="16px" src="icon-no.svg" title="Topic only"/></td>
            <td align="center"><img width="16px" src="icon-no.svg" title="Subject only"/></td>
            <td align="center"><img width="16px" src="icon-no.svg" title="Topic only"/></td>
        </tr>
        <tr>
            <td>Grouping and Logic</td>
            <td align="center"><img width="16px" src="icon-yes.svg" title="nested arbitrary groups + logic and/or/xor"/></td>
            <td align="center"><img width="16px" src="icon-no.svg" title=""/></td>
            <td align="center"><img width="16px" src="icon-no.svg" title=""/></td>
            <td align="center"><img width="16px" src="icon-no.svg" title="the only option available is to subscribe queue to multiple topics"/></td>
        </tr>
        <tr>
            <td rowspan="2">Hosting</td> 
            <td>Cloud</td>
            <td align="center"><img width="16px" src="icon-yes.svg" /></td>
            <td align="center"><img width="16px" src="icon-yes.svg" /></td>
            <td align="center"><img width="16px" src="icon-no.svg" /></td>
            <td align="center"><img width="16px" src="icon-yes.svg" /></td>
        </tr>
        <tr>
            <td>Self-Hosted</td>
            <td align="center"><img width="16px" src="icon-yes.svg"/></td>
            <td align="center"><img width="16px" src="icon-yes.svg" title=""/></td>
            <td align="center"><img width="16px" src="icon-yes.svg" title=""/></td>
            <td align="center"><img width="16px" src="icon-no.svg" title=""/></td>
        </tr>
    </tbody>
</table>

## 1.2. Purpose

### 1.2.1. Get Notified 

Subscribe for important info and receive the relevant information in the real time.
Latest news and updates from any source.

### 1.2.2. Advertise

Publish own information and everyone interested will receive it.

### 1.2.3. Integrate

Publish and receive messages using RSS feeds, Chat Bots or message queues.
Use Machine Learning algorithms to classify and label the image, audio or video data when publish.

## 1.3. Definitions

Awarkari works with messages and subscriptions.

### 1.3.1. Matching Condition

A condition represents a message matching criteria. [Learn more](https://github.com/awakari/subscriptions#121-condition)

### 1.3.2. Subscription

Subscriptions is a set of matching conditions associated with a specific user.
[Learn more](https://github.com/awakari/subscriptions#122-subscription)

### 1.3.3. Message

The entity being routed and delivered by Awakari. The accepted format is [Cloud Events](https://cloudevents.io)

# 2. Deployment

There are two deployment options:
1. [Self-Hosted](https://github.com/awakari/core#3-deployment)
2. Cloud

![components](components.png)

# 3. Additional Information

* [Roadmap](ROADMAP.md)
* [Contributing](CONTRIBUTING.md)
