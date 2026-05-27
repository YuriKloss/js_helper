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

    %% Traffic flows
    Browser -->|HTTPS| LB
    CLI -->|HTTPS| LB
    LB --> Nginx
    Nginx -->|Route to Laravel| APIGW
    Nginx -->|Route to Python| APIGW
    APIGW --> Laravel
    APIGW --> Python

    %% Laravel flows
    Laravel -->|Cache/Session| Redis
    Laravel -->|Queue| Redis
    Laravel -->|DB| MySQL
    Laravel -->|Files| S3
    Laravel --> Horizon
    Horizon -->|Process jobs| Redis

    %% Python flows
    Python -->|Cache| Redis
    Python -->|Queue| RabbitMQ
    Python -->|DB| Postgres
    Python --> Celery
    Celery -->|Process jobs| RabbitMQ

    %% External requests through proxy
    Laravel -->|External HTTP| ProxyHTTP
    Python -->|External HTTP| ProxyHTTP
    ProxyHTTP -->|Outgoing| ExtExternal((External APIs))

    %% Static files
    Laravel -.->|Serve static| Static
    Browser -.->|Static assets| Static

    %% Observability
    Laravel -->|Metrics| Prometheus
    Python -->|Metrics| Prometheus
    Nginx -->|Logs| ELK
    Laravel -->|Logs| ELK
    Python -->|Logs| ELK
    Prometheus --> Grafana
    Laravel -->|Traces| Jaeger
    Python -->|Traces| Jaeger

    %% CI/CD
    GitHub -->|Build & Deploy| Laravel
    GitHub -->|Build & Deploy| Python
    GitHub -->|Deploy| Nginx

    %% Styling
    style Clients fill:#e1f5ff,stroke:#01579b
    style CDN fill:#fff3e0,stroke:#e65100
    style Edge fill:#e8f5e9,stroke:#1b5e20
    style Gateway fill:#f3e5f5,stroke:#4a148c
    style Backend fill:#ffebee,stroke:#b71c1c
    style Queue fill:#e0f2f1,stroke:#004d40
    style DataLayer fill:#fff8e1,stroke:#ff6f00
    style Storage fill:#f3e5f5,stroke:#4a148c
    style Proxy fill:#fce4ec,stroke:#880e4f
    style Observability fill:#eceff1,stroke:#263238
    style CI_CD fill:#c8e6c9,stroke:#1b5e20
