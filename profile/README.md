<img alt="title" height="80" src="title.png"/>

# Scalable Wildcard Subscriptions

Resolution sequence diagram:

![dia-seq-subscription-resolution](dia-seq-subscription-resolution.png)

The above diagram is simplified, it hides:
1. message metadata values segmentation and matches against every lexeme
2. 4 different matchers services - includes/excludes x complete/partial

```mermaid
%%{init: {'theme': 'neutral' } }%%
sequenceDiagram

    actor Source
    participant Adapter
    participant Resolver
    participant Matchers
    participant Subscriptions
    actor Destination

    Source-)Adapter: message
    activate Adapter
    
    Adapter->>Resolver: message id, metadata
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
                Resolver->>Resolver: enroll match
            end
            deactivate Resolver
            
        end
        deactivate Resolver
    end
    
    
    
    Resolver-->>Adapter: done
    deactivate Resolver

    activate Adapter
    loop each subscription page
        Adapter->>Resolver: resolve subscriptions matches by the message id
        activate Resolver
        Resolver->>Adapter: next subscriptions matches page
        deactivate Resolver
        activate Adapter
        Adapter-)Destination: message, next subscriptions batch
        deactivate Adapter
    end
    deactivate Adapter
    deactivate Adapter
```