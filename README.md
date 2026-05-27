```mermaid
flowchart TB
    subgraph Clients ["Клиенты"]
        Browser[Браузер / Мобильное приложение]
        CLI[CLI клиенты]
    end

    subgraph CDN ["CDN / Статика (опционально)"]
        Static[Объектное хранилище S3/MinIO + CDN]
    end

    subgraph Edge ["Edge / Ingress (PUBLIC)"]
        LB[External Load Balancer / Cloud LB]
        Nginx[Nginx Reverse Proxy, SSL termination, rate limit, WAF]
    end

    subgraph Gateway ["API Gateway / Auth (опционально)"]
        APIGW[API Gateway, JWT/OAuth, routing, logging]
    end

    subgraph Backend ["Backend Services"]
        Laravel[Laravel (PHP-FPM), API + Web]
        Python[Python Service, ML/scraper/worker]
    end

    subgraph Queue ["Очереди и Workers"]
        Redis[(Redis, Cache + Queue + Session)]
        RabbitMQ[(RabbitMQ, Queue)]
        Horizon[Laravel Horizon, Queue Worker]
        Celery[Celery, Python Worker]
    end

    subgraph DataLayer ["Слой данных"]
        MySQL[(MySQL / MariaDB)]
        Postgres[(PostgreSQL)]
    end

    subgraph Storage ["Storage"]
        S3[S3 / MinIO, Files, Backups]
    end

    subgraph Proxy ["Proxy (исходящие запросы)"]
        ProxyHTTP[HTTP/SOCKS5 Proxy, Outgoing traffic]
    end

    subgraph Observability ["Observability"]
        Prometheus[Prometheus, Metrics]
        Grafana[Grafana, Dashboards]
        ELK[ELK / EFK / Loki, Logs]
        Jaeger[Jaeger / Tempo, Tracing]
    end

    subgraph CI_CD ["CI/CD"]
        GitHub[GitHub Actions / GitLab CI, Build, Test, Deploy]
    end

    Browser -->|HTTPS| LB
    CLI -->|HTTPS| LB
    LB --> Nginx
    Nginx -->|Route to Laravel| APIGW
    Nginx -->|Route to Python| APIGW
    APIGW --> Laravel
    APIGW --> Python

    Laravel -->|Cache/Session| Redis
    Laravel -->|Queue| Redis
    Laravel -->|DB| MySQL
    Laravel -->|Files| S3
    Laravel --> Horizon
    Horizon -->|Process jobs| Redis

    Python -->|Cache| Redis
    Python -->|Queue| RabbitMQ
    Python -->|DB| Postgres
    Python --> Celery
    Celery -->|Process jobs| RabbitMQ

    Laravel -->|External HTTP| ProxyHTTP
    Python -->|External HTTP| ProxyHTTP
    ProxyHTTP -->|Outgoing| ExtExternal((External APIs))

    Laravel -.->|Serve static| Static
    Browser -.->|Static assets| Static

    Laravel -->|Metrics| Prometheus
    Python -->|Metrics| Prometheus
    Nginx -->|Logs| ELK
    Laravel -->|Logs| ELK
    Python -->|Logs| ELK
    Prometheus --> Grafana
    Laravel -->|Traces| Jaeger
    Python -->|Traces| Jaeger

    GitHub -->|Build & Deploy| Laravel
    GitHub -->|Build & Deploy| Python
    GitHub -->|Deploy| Nginx
```
