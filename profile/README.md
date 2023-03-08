# Contents

1. [Overview](#1-overview)<br/>
   1.1. [Comparison](#11-comparison)<br/>
   1.2. [Purpose](#12-purpose)<br/>
   1.3. [Definitions](#13-definitions)<br/>
2. [Deployment](#2-deployment)<br/>
3. [Usage](#3-usage)<br/>
4. [Configuration](#4-configuration)<br/>
5. [Design](#5-design)<br/>
6. [Roadmap](#6-roadmap)<br/>
7. [Contributing](#7-contributing)<br/>

# 1. Overview

Awakari is a flexible and scalable message pub/sub routing system. Awakari brings additional features to the 
subscriptions:
* Scalability to handle any big number of subscriptions
* Rich subscription conditions: 
  * wildcards
  * arbitrary grouping and nested conditions 
  * logic functions

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

The core of Awakari consist of 3 storages (conditions, subscriptions, matches) and 2 stateless services (resolver, 
router). The high-level message processing sequence follows: 

```mermaid
%%{init: {'theme': 'neutral' } }%%
sequenceDiagram

    autonumber

    actor Source
    participant Resolver
    participant Conditions
    participant Subscriptions
    participant Matches
    participant Router
    actor Destination

    Source-)Resolver: message
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
            Router-)Destination: message, route
            deactivate Router
        end
        
    end
```

Components:

```mermaid
%%{init: {'theme': 'neutral' } }%%
flowchart LR
    subgraph pipeline
        direction TB
        producer[[Producer]]
        resolverQueue([Resolver Queue])
        resolver[[Resolver]]
        routerQueue([Router Queue])
        router[[Router]]
        consumerQueue([Consumer Queue])
        consumer[[Consumer]]
    end
    conditions[(Conditions)]
    subscriptions[(Subscriptions)]
    matches[(Matches)]
    frontend(Frontend) -->|create, read, delete| subscriptions
    producer -->|submit| resolverQueue
    resolver -->|search by value| conditions
    resolver --> |search by condition| subscriptions
    resolver --> |register| matches
    subscriptions --> |create, delete| conditions
    router --> |search| matches
    resolverQueue --> resolver -->|submit| routerQueue --> router -->|submit| consumerQueue --> consumer -->|notify| frontend
```

# 6. Roadmap

Refer to the [page](ROADMAP.md)

# 7. Contributing

Refer to the [page](CONTRIBUTING.md)
