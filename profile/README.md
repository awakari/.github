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

Conventional search engines are used to find information by a query. 
The problem is reverse in the modern world of real-time data streams.
It's necessary to filter and consume the data matching a query on the fly.
Existing stream processing solutions offer only the iteration way to find matching queries.
For example, a system may have billions of user queries registered.
Then every incoming message causes the iteration over the billions queries to find all the matches.
So this doesn't scale efficiently when number of queries grows.

Awakari is a message pub/sub system that comes to solve this problem.
It brings additional features:
* Rich subscription matching conditions: 
  * Wildcards
  * Grouping:
    * Nested conditions and groups 
    * Logic functions
* Scalability:
  * Handle any big number of subscriptions
  * Sustain any large message throughput without a latency impact

## 1.1. Comparison

Awakari may be used together with another reliable message queue system and inherit the persistence and delivery 
guarantee. The key differences from other message pub/sub solutions are:
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
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="2">Wildcard Subscriptions</td>
            <td>Horizontal Scalability</td>
            <td align="center"><img width="16px" src="icon-yes.svg" title=""/></td>
            <td align="center"><img width="16px" src="icon-no.svg" title="consumer- side topic matching"/></td>
            <td align="center"><img width="16px" src="icon-yes.svg" title=""/></td>
        </tr>
        <tr>
            <td>Matching Time Complexity</td>
            <td align="center"><img width="16px" src="icon-yes.svg" title="O(log(N)) for kiwi-tree subscriptions"/></td>
            <td align="center"><img width="16px" src="icon-no.svg" title="O(N)"/></td>
            <td align="center"><img width="16px" src="icon-no.svg" title="O(N)"/></td>
        </tr>
        <tr>
            <td rowspan="2">Matching Criteria</td> 
            <td>Attributes</td>
            <td align="center"><img width="16px" src="icon-yes.svg" title="Any metadata (key/value)"/></td>
            <td align="center"><img width="16px" src="icon-no.svg" title="Topic only"/></td>
            <td align="center"><img width="16px" src="icon-no.svg" title="Subject only"/></td>
        </tr>
        <tr>
            <td>Grouping and Logic</td>
            <td align="center"><img width="16px" src="icon-yes.svg" title="nested arbitrary groups + logic and/or/xor"/></td>
            <td align="center"><img width="16px" src="icon-no.svg" title=""/></td>
            <td align="center"><img width="16px" src="icon-no.svg" title=""/></td>
        </tr>
    </tbody>
</table>

## 1.2. Purpose

TODO

## 1.3. Definitions

TODO

# 2. Deployment

TODO

# 3. Usage

TODO

# 4. Configuration

TODO

# 5. Design

The core of Awakari consist of: 
* Storage Services
  * Conditions
  * Subscriptions
  * Matches
* Stateless Processing Pipeline
  * Resolver
  * Router
* Queue Service

Additionally, there's:
* Source message producer
* Routed message consumer
* UI

![components](components.png)

Producer flow: 

```mermaid
%%{init: {'theme': 'neutral' } }%%
sequenceDiagram

    autonumber

    actor Producer
    participant Resolver
    participant Conditions
    participant Subscriptions
    participant Matches
    participant Messages

    Producer-)Resolver: Submit Messages
    activate Resolver
    
    loop Message
        loop Message Attributes
        
            Resolver->>Conditions: Search by Key/Value
            activate Conditions
            Conditions->>Resolver: Next Conditions Page
            deactivate Conditions
            
            activate Resolver
            loop Each Condition
                
                Resolver->>Subscriptions: Search by Condition
                activate Subscriptions
                Subscriptions->>Resolver: Next Subscriptions Page
                deactivate Subscriptions
                
                activate Resolver
                loop Each Subscription
                    Resolver->>Matches: Register Match Candidate
                    activate Matches
                    Matches-->>Resolver: Ack
                    deactivate Matches
                end
                deactivate Resolver
                
            end
            deactivate Resolver
        end
    end
        
    Resolver-)Messages: Submit Messages
    deactivate Resolver
```

Consumer flow:

```mermaid
%%{init: {'theme': 'neutral' } }%%
sequenceDiagram

    autonumber

    actor Consumer
    participant Router
    participant Subscriptions
    participant Matches
    participant Messages

    Consumer-)Router: Get Messages by Account
    activate Router
    Router->>Subscriptions: Get Subscriptions by Account
    
    activate Subscriptions
    Subscriptions->>Router: Next Subscriptions Page
    deactivate Subscriptions
    
    loop each Subscription
    activate Router
    
        Router->>Matches: Withdraw by Subscription
        activate Matches
        Matches->>Router: Next Matches Page
        deactivate Matches
        
        loop each Match
        activate Router
            
            Router->>Messages: Search by Ids
            activate Messages
            Messages->>Router: Next Messages Page
            deactivate Messages
            Router-)Consumer: Push Messages
        
        deactivate Router    
        end
        
    deactivate Router  
    end
        
    deactivate Router
```

# 6. Additional Information

* [Roadmap](ROADMAP.md)
* [Contributing](CONTRIBUTING.md)
