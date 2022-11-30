<img alt="title" height="80" src="title.png"/>

# Scalable Wildcard Subscriptions

Resolution sequence diagram:

![dia-seq-subscription-resolution](dia-seq-subscription-resolution.png)

The above diagram is simplified, it hides:
1. message metadata values segmentation and matches against every lexeme
2. 4 different matchers services - includes/excludes x complete/partial

```mermaid
sequenceDiagram
    ->>Client: msg
    activate Client
    Client->>Resolver: msg (id, md)
    activate Resolver
    loop md(k, v)
        
    end
    Resolver-->>Client: done 
    Client-->>: done
```