# Scalable wildcard message routing

<table>
    <thead>
        <tr>
            <td></td>
            <td colspan="5" align="center"><b>Solution</b></td>
        </tr>
        <tr>
            <td rowspan="2"></td>
            <td rowspan="2" align="center"><b>Redis</b></td>
            <td rowspan="2" align="center"><b>Nats</b></td>
            <td rowspan="2" align="center"><b>Kafka</b></td>
            <td colspan="2" align="center"><b>Awakari</b></td>
        </tr>
        <tr>
            <td>kiwi-tree</td>
            <td>kiwi-bird</td>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><b>Pattern Syntax</b></td>
            <td><img width="16px" src="icon-no.svg"/> Limited</td>
            <td><img width="16px" src="icon-no.svg"/> Limited</td>
            <td><img width="16px" src="icon-yes.svg"/> Full</td>
            <td><img width="16px" src="icon-no.svg"/> Limited</td>
            <td><img width="16px" src="icon-yes.svg"/></img> Full</td>
        </tr>
        <tr>
            <td><b>Matching Time</b><br/>(N subscriptions)</td>
            <td><img width="16px" src="icon-no.svg"/> O(N)</td>
            <td><img width="16px" src="icon-no.svg"/> O(N)</td>
            <td><img width="16px" src="icon-no.svg"/> O(N)</td>
            <td><img width="16px" src="icon-yes.svg"/> O(log(N))</td>
            <td><img width="16px" src="icon-no.svg"/> O(N)</td>
        </tr>
        <tr>
            <td><b>Horizontal Scalability</b></td>
            <td><img width="16px" src="icon-yes.svg"/></td>
            <td><img width="16px" src="icon-yes.svg"/></td>
            <td><img width="16px" src="icon-no.svg"/> Consumer matches</td>
            <td><img width="16px" src="icon-yes.svg"/></td>
            <td><img width="16px" src="icon-yes.svg"/></td>
        </tr>
        <tr>
            <td><b>Matching Criteria</b></td>
            <td><img width="16px" src="icon-no.svg"/>Channel only</td>
            <td><img width="16px" src="icon-no.svg"/>Subject only</td>
            <td><img width="16px" src="icon-no.svg"/>Topic only</td>
            <td colspan="2"><img width="16px" src="icon-yes.svg"/>Any metadata (key-value)</td>
        </tr>
    </tbody>
</table>

| Solution                          | Redis                                       | Nats                                        | Kafka                                        | Awakari<br>kiwi-tree                        | Awakari<br/>kiwi-bird                       |
|-----------------------------------|---------------------------------------------|---------------------------------------------|----------------------------------------------|---------------------------------------------|---------------------------------------------|
| Subscription Pattern Syntax       | <span style="color:red">Limited</span>      | <span style="color:red">Limited</span>      | <span style="color:green">Full</span>        | <span style="color:red">Limited</span>      | <span style="color:green">Full</span>       |
| Matching time for N subscriptions | <span style="color:red">O(N)</span>         | <span style="color:red">O(N)</span>         | <span style="color:red">O(N)</span>          | <span style="color:green">O(log(N))</span>  | <span style="color:red">O(N)</span>         |
| Matching side                     | <span style="color:green">Broker</span>     | <span style="color:green">Broker</span>     | <span style="color:red">Consumer</span>      | <span style="color:green">Broker</span>     | <span style="color:green">Broker</span>     |
| Scalability                       | <span style="color:green">Horizontal</span> | <span style="color:green">Horizontal</span> | <span style="color:red">Vertical only</span> | <span style="color:green">Horizontal</span> | <span style="color:green">Horizontal</span> |

Existing messaging solutions offer wildcard subscriptions:
* [Redis](https://redis.io/commands/psubscribe/)
* [NATS](https://docs.nats.io/nats-concepts/subjects#wildcards)
* [Kafka](https://kafka.apache.org/32/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#subscribe(java.util.regex.Pattern,org.apache.kafka.clients.consumer.ConsumerRebalanceListener))

These solutions are able to resolve subscriptions in O(N) time, where N is the number of wildcard subscriptions.
This is not efficient because the time to resolve such subscriptions will grow with increasing number of subscriptions.
Awakari offers a more scalable solution for this.

Nevertheless, Awakari is not a messaging delivery system but a primitive.
The purpose is only to resolve subscriptions by an input message.
So it should be used as a component of a more sophisticated system.
For example, it can be used as a part of the system working ahead of Redis/NATS/Kafka to solve their scalability issues.

The heart of Awakari is [Kiwi](https://github.com/awakari/kiwi) providing the wildcards search by a sample in a O(log N)
time. 

```mermaid
%%{init: {'theme': 'neutral' } }%%
sequenceDiagram

    actor Source
    participant Resolver
    participant Kiwi
    participant Subscriptions
    participant Matches
    participant Aggregator
    actor Destination

    Source-)Resolver: message
    activate Resolver
    
    loop message metadata (k, v)
    
        Resolver->>Kiwi: resolve by next (k, v)
        activate Kiwi
        Kiwi->>Resolver: next patterns page
        deactivate Kiwi
        
        activate Resolver
        loop each pattern in page
            
            Resolver->>Subscriptions: resolve by next (k, pattern) pair
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