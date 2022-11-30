<img alt="title" height="80" src="title.png"/>

# Scalable Wildcard Subscriptions

Resolution sequence diagram:

![dia-seq-subscription-resolution](dia-seq-subscription-resolution.png)

The above diagram is simplified, it hides:
1. message metadata values segmentation and matches against every lexeme
2. 4 different matchers services - includes/excludes x complete/partial

```mermaid
sequenceDiagram
    actor Message Bus
    participant Adapter
    participant Resolver
    participant Matchers
    participant Subscriptions
    Message Bus->>Adapter: msg
    activate Adapter
    Adapter->>Resolver: msg (id, md) 
    Resolver-->>Adapter: done
    Adapter-->>Message Bus: done
    deactivate Adapter
```