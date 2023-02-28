# Scalable wildcard message routing

Awakari is a message pub/sub routing system. It's designed to be used ahead of a reliable message queue like Kafka to 
bring additional subscription-based routing flexibility. Awakari subscriptions have the following additional features:
1. Scalability to handle billions of subscriptions
2. Rich routing conditions in subscriptions

Comparison to other message pub/sub solutions:
<table align="center">
    <thead>
        <tr>
            <td rowspan="2"><b>Criteria</b></td>
            <td colspan="4" align="center"><b>Solution</b></td>
        </tr>
        <tr>
            <td align="center" valign="top"><b>Nats<br/>(JetStream)</b></td>
            <td align="center" valign="top"><b>Kafka</b></td>
            <td align="center" valign="top"><b>Awakari</b></td>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Horizontal Scalability</td>
            <td><img width="16px" src="icon-yes.svg" title=""/></td>
            <td><img width="16px" src="icon-no.svg" title="consumer-side topic matching"/></td>
            <td><img width="16px" src="icon-yes.svg" title=""/></td>
        </tr>
        <tr>
            <td>Persistence</td>
            <td><img width="16px" src="icon-yes.svg" title="in the JetStream mode"/></td>
            <td><img width="16px" src="icon-yes.svg" title=""/></td>
            <td><img width="16px" src="icon-no.svg" title=""/></td>
        </tr>
        <tr>
            <td>Delivery Guarantee</td>
            <td><img width="16px" src="icon-yes.svg" title="Exactly once (JetStream)"/></td>
            <td><img width="16px" src="icon-yes.svg" title="Exactly once"/></td>
            <td><img width="16px" src="icon-no.svg" title="At most once"/></td>
        </tr>
        <tr>
            <td>Full Pattern Syntax</td>
            <td><img width="16px" src="icon-no.svg" title="Limited"/></td>
            <td><img width="16px" src="icon-yes.svg" title="Complete"/></td>
            <td><img width="16px" src="icon-yes.svg" title="Complete for kiwi-bird subscriptions"/></td>
        </tr>
        <tr>
            <td>Matching Time (N subscriptions)</td>
            <td><img width="16px" src="icon-no.svg" title="O(N)"/></td>
            <td><img width="16px" src="icon-no.svg" title="O(N)"/></td>
            <td><img width="16px" src="icon-yes.svg" title="O(log(N)) for kiwi-tree subscriptions"/></td>
        </tr>
        <tr>
            <td>Matching Criteria Attributes</td>
            <td><img width="16px" src="icon-no.svg" title="Subject only"/></td>
            <td><img width="16px" src="icon-no.svg" title="Topic only"/></td>
            <td><img width="16px" src="icon-yes.svg" title="Any metadata (key/value)"/></td>
        </tr>
        <tr>
            <td>Matching Criteria Groups/Logic</td>
            <td><img width="16px" src="icon-no.svg" title=""/></td>
            <td><img width="16px" src="icon-no.svg" title=""/></td>
            <td><img width="16px" src="icon-yes.svg" title="nested groups + logic and/or/xor"/></td>
        </tr>
    </tbody>
</table>

Basically, Awakari consist of 3 storages (conditions, subscriptions, matches) and 2 stateless functions (resolver, 
aggregator). The high-level processing sequence follows: 

```mermaid
%%{init: {'theme': 'neutral' } }%%
sequenceDiagram

    autonumber

    actor Source
    participant Resolver
    participant Conditions
    participant Subscriptions
    participant Matches
    participant Aggregator
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
        
    Resolver-)Aggregator: message
    deactivate Resolver

    activate Aggregator
    loop each matching subscriptions page
        
        Aggregator->>Matches: message id
        activate Matches
        Matches->>Aggregator: next subscriptions page
        deactivate Matches
        
        loop each subscription in page
            Aggregator-)Destination: message, route
            deactivate Aggregator
        end
        
    end
```