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
    participant Input Adapter
    participant Resolver
    participant Matchers
    participant Subscriptions
    participant Output Adapter
    actor Destination

    Source-)Input Adapter: message
    activate Input Adapter
    
    Input Adapter-)Resolver: message id, metadata
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
                Resolver->>Resolver: register next match for message id, subscription
            end
            deactivate Resolver
            
        end
        deactivate Resolver
    end
    
    
    
    Resolver->>Input Adapter: message id
    deactivate Resolver

    activate Input Adapter
    loop each matching subscriptions page
        Input Adapter->>Resolver: resolve matches by message id
        activate Resolver
        Resolver->>Input Adapter: next subscriptions page
        deactivate Resolver
        activate Input Adapter
        
        loop each subscription in page
            Input Adapter-)Output Adapter: message, matching subscription
            deactivate Input Adapter
            activate Output Adapter
            Output Adapter-)Destination: message, route
            deactivate Output Adapter
        end
        
    end
    deactivate Input Adapter
```