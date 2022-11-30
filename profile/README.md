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

    actor 3rd Party Message Bus
    participant Adapter
    participant Resolver
    participant Matchers
    participant Subscriptions

    3rd Party Message Bus-)Adapter: message
    activate Adapter
    
    Adapter->>Resolver: message id, metadata
    activate Resolver
    
    loop message metadata (k, v)
        Resolver->>Matchers: resolve matchers by next (k, v)
        activate Matchers
        Matchers->>Resolver: next matchers page
        deactivate Matchers
    end
    
    Resolver-->>Adapter: done
    deactivate Resolver

    activate Adapter
    loop subscriptions
        Adapter->>Resolver: resolve subscriptions matches by the message id
        Resolver-->>Adapter: next subscriptions matches page
    end
    deactivate Adapter
    deactivate Adapter
```