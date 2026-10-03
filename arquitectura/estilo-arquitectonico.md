## Estilo Arquitectónico

```mermaid
flowchart TB
    %% ACTORES
    subgraph ACTORES["Usuarios del sistema"]
        direction LR
        CLI["Cliente"]
        SEL["Seller"]
        ADM["Administrador"]
    end

    WEB["Cliente Web<br/>HTML / CSS / JavaScript"]

    CLI --> WEB
    SEL --> WEB
    ADM --> WEB

    %% BACKEND MONOLITICO
    subgraph MONO["Marketplace Backend - Node.js + Express"]
        direction TB

        MW["Middlewares Express<br/>CORS | JSON | JWT | Validación | Errores | Logger"]

        %% PRESENTACION
        subgraph PRESENTACION["1. CAPA DE PRESENTACIÓN"]
            direction LR

            subgraph USR["Módulo usuarios"]
                UR["usuarios.routes.js"]
                UC["usuarios.controller.js"]
                UR --> UC
            end

            subgraph SELL["Módulo sellers"]
                SR["sellers.routes.js"]
                SC["sellers.controller.js"]
                SR --> SC
            end

            subgraph CAT["Módulo catálogo"]
                CR["catalogo.routes.js"]
                CC["catalogo.controller.js"]
                CR --> CC
            end

            subgraph CART["Módulo carrito"]
                CAR["carrito.routes.js"]
                CAC["carrito.controller.js"]
                CAR --> CAC
            end

            subgraph ORD["Módulo pedidos"]
                PR["pedidos.routes.js"]
                PC["pedidos.controller.js"]
                PR --> PC
            end
        end

        %% LOGICA DE NEGOCIO
        subgraph LOGICA["2. CAPA DE LÓGICA DE NEGOCIO"]
            direction LR
            US["usuarios.service.js<br/>Registro, login y roles"]
            SS["sellers.service.js<br/>Tiendas y validación"]
            CS["catalogo.service.js<br/>Productos, categorías y stock"]
            CAS["carrito.service.js<br/>Ítems y totales"]
            PS["pedidos.service.js<br/>Checkout, estados, pago y envío"]
        end

        %% DATOS
        subgraph DATOS["3. CAPA DE DATOS"]
            direction TB

            subgraph REPOS["Repositorios"]
                direction LR
                URepo["usuarios.repository.js"]
                SRepo["sellers.repository.js"]
                CRepo["catalogo.repository.js"]
                CARepo["carrito.repository.js"]
                PRepo["pedidos.repository.js"]
            end

            ORM["Acceso a datos compartido<br/>Sequelize ORM | Modelos | Pool de conexiones"]
        end

        %% MIDDLEWARES
        MW --> UR
        MW --> SR
        MW --> CR
        MW --> CAR
        MW --> PR

        %% CONTROLADORES A SERVICIOS
        UC --> US
        SC --> SS
        CC --> CS
        CAC --> CAS
        PC --> PS

        %% SERVICIOS A REPOSITORIOS
        US --> URepo
        SS --> SRepo
        CS --> CRepo
        CAS --> CARepo
        PS --> PRepo

        %% REPOSITORIOS A ORM
        URepo --> ORM
        SRepo --> ORM
        CRepo --> ORM
        CARepo --> ORM
        PRepo --> ORM

        %% COMUNICACION ENTRE MODULOS
        PS -.-> US
        PS -.-> CS
        PS -.-> CAS
        CS -.-> SS
    end

    %% BASE DE DATOS
    DB[("PostgreSQL<br/>marketplace_db")]

    %% SERVICIOS EXTERNOS
    PAGOS["Pasarela de pagos<br/>Culqi / Niubiz"]
    ENVIOS["Servicio de envíos<br/>API del courier"]

    %% CONEXIONES EXTERNAS
    WEB -->|"HTTPS / JSON"| MW
    ORM -->|"SQL / TCP 5432"| DB
    PS -->|"HTTPS / REST"| PAGOS
    PS -->|"HTTPS / REST"| ENVIOS

    %% ESTILOS
    classDef actor fill:#ffffff,stroke:#777,color:#222
    classDef middleware fill:#dae5fb,stroke:#91aad4,color:#222
    classDef presentation fill:#edf3ff,stroke:#8ca8d4,color:#222
    classDef service fill:#deedcf,stroke:#a5c68c,color:#222
    classDef repository fill:#fff9e9,stroke:#dbba65,color:#222
    classDef orm fill:#f8e8cd,stroke:#d5ac65,color:#222
    classDef external fill:#eeeeee,stroke:#888,color:#222

    class CLI,SEL,ADM actor
    class MW middleware
    class UR,UC,SR,SC,CR,CC,CAR,CAC,PR,PC presentation
    class US,SS,CS,CAS,PS service
    class URepo,SRepo,CRepo,CARepo,PRepo repository
    class ORM orm
    class PAGOS,ENVIOS external

    style MONO fill:#ffffff,stroke:#555,stroke-width:2px,stroke-dasharray:7 5
    style PRESENTACION fill:#e7effb,stroke:#a5b8d4
    style LOGICA fill:#edf5e7,stroke:#a3bf91
    style DATOS fill:#fff4dd,stroke:#d6bb83
```

