<img alt="title" height="80" src="title.png"/>

# Scalable Wildcard Subscriptions

```mermaid
%%{init: {'theme': 'neutral' } }%%
sequenceDiagram

    actor Source
    participant Input Adapter
    participant Resolver
    participant Matchers
    participant Subscriptions
    participant Matches
    participant Aggregator
    participant Output Adapter
    actor Destination

    Source-)Input Adapter: message
    activate Input Adapter
        
    Input Adapter-)Resolver: message
    deactivate Input Adapter
    activate Resolver
    
    loop message metadata (k, v)
    
        Resolver->>Matchers: resolve by next (k, v)
        activate Matchers
        Matchers->>Resolver: next matchers page
        deactivate Matchers
        
        activate Resolver
        loop each matcher in page
            
            Resolver->>Subscriptions: resolve by next matcher
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
        
        Aggregator->>Matches: resolve matches by message id
        activate Matches
        Matches->>Aggregator: next subscriptions page
        deactivate Matches
        
        loop each subscription in page
            Aggregator-)Output Adapter: message, route
            deactivate Aggregator
            activate Output Adapter
            Output Adapter-)Destination: message, route
            deactivate Output Adapter
        end
        
    end
```