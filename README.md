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
        Nginx[Nginx Reverse Proxy<br/>SSL termination, rate limit, WAF]
    end

    subgraph Gateway ["API Gateway / Auth (опционально)"]
        APIGW[API Gateway<br/>JWT/OAuth, routing, logging]
    end

    subgraph Backend ["Backend Services"]
        Laravel[Laravel (PHP-FPM)<br/>API + Web]
        Python[Python Service<br/>ML/scraper/worker]
    end

    subgraph Queue ["Очереди и Workers"]
        Redis[(Redis<br/>Cache + Queue + Session)]
        RabbitMQ[(RabbitMQ<br/>Queue)]
        Horizon[Laravel Horizon<br/>Queue Worker]
        Celery[Celery<br/>Python Worker]
    end

    subgraph DataLayer ["Слой данных"]
        MySQL[(MySQL / MariaDB)]
        Postgres[(PostgreSQL)]
    end

    subgraph Storage ["Storage"]
        S3[S3 / MinIO<br/>Files, Backups]
    end

    subgraph Proxy ["Proxy (исходящие запросы)"]
        ProxyHTTP[HTTP/SOCKS5 Proxy<br/>Outgoing traffic]
    end

    subgraph Observability ["Observability"]
        Prometheus[Prometheus<br/>Metrics]
        Grafana[Grafana<br/>Dashboards]
        ELK[ELK / EFK / Loki<br/>Logs]
        Jaeger[Jaeger / Tempo<br/>Tracing]
    end

    subgraph CI_CD ["CI/CD"]
        GitHub[GitHub Actions / GitLab CI<br/>Build, Test, Deploy]
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
