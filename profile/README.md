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

Awakari is a message pub/sub routing system that comes to solve this problem.
It brings additional features:
* Rich subscription matching conditions: 
  * Wildcards
  * Grouping:
    * Nested conditions and groups 
    * Logic functions
* Scalability:
  * Handle any big number of subscriptions
  * Sustain any large message throughput without latency impact

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
  * Producer
  * Resolver
  * Router
  * Consumer

![components](components.png)

The high-level message processing sequence follows: 

```mermaid
%%{init: {'theme': 'neutral' } }%%
sequenceDiagram

    autonumber

    actor Producer
    participant Resolver
    participant Conditions
    participant Subscriptions
    participant Matches
    participant Router
    actor Consumer

    Producer-)Resolver: message
    activate Resolver
    
    loop message attributes
    
        Resolver->>Conditions: resolve by next attribute 
        activate Conditions
        Conditions->>Resolver: next conditions page
        deactivate Conditions
        
        activate Resolver
        loop each condition in page
            
            Resolver->>Subscriptions: resolve by next condition
            activate Subscriptions
            Subscriptions->>Resolver: next subscriptions page
            deactivate Subscriptions
            
            activate Resolver
            loop each subscription in page
                Resolver->>Matches: register next match for message id, subscription
                activate Matches
                Matches-->>Resolver: ack
                deactivate Matches
            end
            deactivate Resolver
            
        end
        deactivate Resolver
    end
        
    Resolver-)Router: message
    deactivate Resolver

    activate Router
    loop each matching subscriptions page
        
        Router->>Matches: message id
        activate Matches
        Matches->>Router: next subscriptions page
        deactivate Matches
        
        loop each subscription in page
            Router-)Consumer: message, route
            deactivate Router
        end
        
    end
```

# 6. Additional Information

* [Roadmap](ROADMAP.md)
* [Contributing](CONTRIBUTING.md)
