## Enfoque Arquitectónico

| Elemento | Descripción aplicada al Marketplace |
| --- | --- |
| Patrón / enfoque arquitectónico | Clean Architecture (Arquitectura Limpia). |
| Objetivo | Separar responsabilidades y controlar las dependencias hacia el dominio.|
| ¿Qué problema resuelve?| Evita el acoplamiento entre la interfaz Angular, las reglas del negocio y las tecnologías externas, como bases de datos, API y servicios de pago.|
| Capas definidas | Presentación, Aplicación, Dominio e Infraestructura. |
| Beneficios| • Facilita el mantenimiento y las pruebas unitarias.<br>• Permite cambiar implementaciones técnicas sin modificar innecesariamente las reglas del negocio.<br>• Mejora la organización y separación de responsabilidades del código.|


### Diagrama de arquitectura

```mermaid
flowchart LR
    %% USUARIO
    USER["Usuario<br/>(Cliente)"]

    %% APLICACIÓN ANGULAR
    subgraph WEB["Marketplace Web — Angular 18 + TypeScript"]
        direction TB

        subgraph FRAMEWORK["ADAPTADORES Y FRAMEWORKS — Angular, HttpClient, RxJS"]
            direction LR

            %% PRESENTACIÓN
            subgraph PRESENTACION["PRESENTACIÓN — src/app/presentacion/"]
                direction TB
                CATALOGO["CatálogoComponent<br/>Lista y filtra productos"]
                ESTADO["EstadoCarrito<br/>Signals, sin reglas de negocio"]
                CARRITO["CarritoComponent<br/>Resumen y confirmación"]
                APP["AppComponent<br/>Shell de la aplicación"]
            end

            %% APLICACIÓN
            subgraph APLICACION["APLICACIÓN — Casos de uso"]
                direction TB
                CONSULTAR["ConsultarCatalogoCasoUso<br/>ejecutar()"]
                AGREGAR["AgregarAlCarritoCasoUso<br/>ejecutar()"]
                REGISTRAR["RegistrarCompraCasoUso<br/>ejecutar()"]
            end

            %% DOMINIO
            subgraph DOMINIO["DOMINIO — Núcleo de negocio — TypeScript puro"]
                direction TB

                subgraph MODELOS["Modelos — Entidades y reglas"]
                    direction TB
                    PRODUCTO["Producto<br/>Stock, categoría, precio"]
                    CARRITO_ENT["Carrito<br/>Inmutable, subtotal, total"]
                    PEDIDO["Pedido<br/>Estados y cancelación"]
                    PRECIOS["precios.ts<br/>Comisión 10 % · IGV 18 %"]
                end

                subgraph CONTRATOS["Contratos — Puertos"]
                    direction TB
                    RP["RepositorioProductos<br/>Interface"]
                    RPE["RepositorioPedidos<br/>Interface"]
                    PP["ProcesadorPagos<br/>Interface"]
                    NC["NotificadorCliente<br/>Interface"]
                end
            end

            %% INFRAESTRUCTURA
            subgraph INFRA["INFRAESTRUCTURA — Adaptadores"]
                direction TB
                RPM["RepositorioProductosMemoria<br/>RepositorioProductosHttp"]
                RPEM["RepositorioPedidosMemoria"]
                PPS["ProcesadorPagosSimulado<br/>ProcesadorPagosNiubiz"]
                NCS["NotificadorConsola<br/>NotificadorWhatsApp"]
                TOKENS["tokens.ts<br/>Angular DI — InjectionToken"]
            end

            %% COMPOSICIÓN
            CONFIG["app.config.ts<br/>Raíz de composición<br/>useFactory + InjectionToken"]
        end
    end

    %% SISTEMA EXTERNO
    API["Marketplace API REST<br/>Backend Node.js — Monolito modular<br/><br/>/api/productos<br/>/api/pedidos<br/>/api/authorization<br/>/api/mensajes"]

    %% INTERACCIONES
    USER -->|"Navegador"| APP

    %% PRESENTACIÓN INVOCA CASOS DE USO
    CATALOGO --> CONSULTAR
    CARRITO --> AGREGAR
    CARRITO --> REGISTRAR

    %% ESTADO DE CARRITO
    CARRITO -.-> ESTADO

    %% CASOS DE USO DEPENDEN DEL DOMINIO
    CONSULTAR -.-> RP
    AGREGAR -.-> CARRITO_ENT
    REGISTRAR -.-> RPE
    REGISTRAR -.-> PP
    REGISTRAR -.-> NC

    %% RELACIÓN ENTRE ENTIDADES
    CARRITO_ENT -.-> PRODUCTO
    PEDIDO -.-> PRODUCTO
    PEDIDO -.-> PRECIOS

    %% ADAPTADORES IMPLEMENTAN CONTRATOS
    RPM -.->|"Implementa"| RP
    RPEM -.->|"Implementa"| RPE
    PPS -.->|"Implementa"| PP
    NCS -.->|"Implementa"| NC

    %% CONFIGURACIÓN DE DEPENDENCIAS
    CONFIG -.->|"Registra"| TOKENS
    CONFIG -.-> RPM
    CONFIG -.-> RPEM
    CONFIG -.-> PPS
    CONFIG -.-> NCS

    %% SISTEMA EXTERNO
    RPM -->|"HTTP / JSON"| API
    PPS -->|"API de pagos"| API
    NCS -->|"Notificaciones"| API

    %% ESTILOS
    classDef usuario fill:#ffffff,stroke:#777,color:#222
    classDef presentacion fill:#e4edfa,stroke:#86a6d5,color:#222
    classDef aplicacion fill:#e9f4e4,stroke:#9bc587,color:#222
    classDef dominio fill:#fff4d6,stroke:#d9b44a,color:#222
    classDef infraestructura fill:#f0e5f7,stroke:#b297c9,color:#222
    classDef configuracion fill:#eeeeee,stroke:#888,color:#222
    classDef externo fill:#eeeeee,stroke:#777,color:#222

    class USER usuario
    class CATALOGO,ESTADO,CARRITO,APP presentacion
    class CONSULTAR,AGREGAR,REGISTRAR aplicacion
    class PRODUCTO,CARRITO_ENT,PEDIDO,PRECIOS,RP,RPE,PP,NC dominio
    class RPM,RPEM,PPS,NCS,TOKENS infraestructura
    class CONFIG configuracion
    class API externo

    style WEB fill:#ffffff,stroke:#666,stroke-width:2px,stroke-dasharray:7 5
    style FRAMEWORK fill:#f8f8f8,stroke:#aaaaaa
    style PRESENTACION fill:#e4edfa,stroke:#86a6d5
    style APLICACION fill:#e9f4e4,stroke:#9bc587
    style DOMINIO fill:#fff4d6,stroke:#d9b44a,stroke-width:2px
    style INFRA fill:#f0e5f7,stroke:#b297c9
    style MODELOS fill:#fff9eb,stroke:#dfc778
    style CONTRATOS fill:#fff9eb,stroke:#dfc778
```

