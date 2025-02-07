# Physical Architecture Diagram

```mermaid
graph TB
    subgraph "External Zone"
        CLB[Cloud Load Balancer]
        WAF[Web Application Firewall]
        Client[Client Applications] --> WAF
        WAF --> CLB
    end

    subgraph "Application Zone"
        ALB[Application Load Balancer]
        CLB --> ALB
        
        subgraph "API Layer"
            API1[API Server 1]
            API2[API Server 2]
            ALB --> API1
            ALB --> API2
        end
        
        subgraph "Model Serving"
            GPU1[GPU Instance 1]
            GPU2[GPU Instance 2]
            API1 --> GPU1
            API1 --> GPU2
            API2 --> GPU1
            API2 --> GPU2
        end
    end

    subgraph "Data Zone"
        subgraph "Caching Layer"
            Redis1[(Redis Primary)]
            Redis2[(Redis Replica)]
            API1 --> Redis1
            API2 --> Redis1
            Redis1 -.-> Redis2
        end
        
        subgraph "Storage Layer"
            Doc[(Document Store)]
            Vector[(Vector Store)]
            Object[(Object Storage)]
            
            API1 --> Doc
            API1 --> Vector
            API1 --> Object
            API2 --> Doc
            API2 --> Vector
            API2 --> Object
        end
    end

    subgraph "Monitoring Zone"
        Prom[Prometheus]
        Graf[Grafana]
        Log[Log Aggregator]
        
        API1 --> Prom
        API2 --> Prom
        GPU1 --> Prom
        GPU2 --> Prom
        Prom --> Graf
        
        API1 --> Log
        API2 --> Log
        GPU1 --> Log
        GPU2 --> Log
    end

    classDef external fill:#2374ab,stroke:#2374ab,color:#fff
    classDef app fill:#57a0d3,stroke:#57a0d3,color:#fff
    classDef data fill:#7bc1f7,stroke:#7bc1f7,color:#fff
    classDef monitor fill:#b5d8f7,stroke:#b5d8f7,color:#000

    class Client,WAF,CLB external
    class ALB,API1,API2,GPU1,GPU2 app
    class Redis1,Redis2,Doc,Vector,Object data
    class Prom,Graf,Log monitor
```

This physical diagram details:

1. Infrastructure components and their relationships
2. Network zones and security boundaries
3. High-availability configurations
4. Monitoring and logging setup