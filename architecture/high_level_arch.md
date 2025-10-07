# Arquitectura (n-layer con comunicación HTTP y futura integración MCP)

La solución se inspira en una arquitectura por capas (n-layer) pero todos los módulos se comunican entre sí mediante HTTP (REST/JSON por defecto). En una fase futura, ciertos clientes técnicos podrán acceder directamente a la Domain Layer vía MCP (Model Context Protocol) para invocar capacidades del dominio de forma segura y trazable, saltando la serialización HTTP cuando corresponda (p. ej., herramientas/agents).

## Diagrama general
```mermaid 
%% Fuerza layout horizontal de los módulos con enlaces invisibles
flowchart TB

    %% ===== Presentation =====
    subgraph Presentation_Layer [Presentation Layer]
        direction LR
        Microfront1[Microfront1]
        Microfront2[Microfront2]
        Microfront3[Microfront3]

        %% Enlaces invisibles para forzar horizontal
        Microfront1 --- Microfront2
        Microfront2 --- Microfront3
    end

    %% ===== App =====
    subgraph App_Layer [App Layer]
        direction LR
        Pulse[Pulse]
        Echo[Echo]
        SoundBoard[SoundBoard]

        %% Enlaces invisibles para forzar horizontal
        Pulse --- Echo
        Echo --- SoundBoard
    end

    %% ===== Auth =====
    subgraph Auth_Layer [Auth Layer]
        direction LR
        AUTH[AUTH]
    end

    %% ===== Domain / Logic =====
    subgraph Domain_Layer [Domain / Logic Layer]
        direction LR
        InsightsExtractor[Insights Extractor]
        FileNormalizer[FileNormalizer]
        DBCRUD[DBCRUD]
        Cache[Cache]

        %% Enlaces invisibles para forzar horizontal
        InsightsExtractor --- FileNormalizer
        FileNormalizer --- DBCRUD
        DBCRUD --- Cache
    end

    %% ===== Data / Services =====
    subgraph Data_Layer [Data / Services Layer]
        direction LR
        LLMs[LLMs]
        VisionModels[VisionModels]
        DBAccess[DB Access]
        MLPipelines[MachineLearningPipelines]

        %% Enlaces invisibles para forzar horizontal
        LLMs --- VisionModels
        VisionModels --- DBAccess
        DBAccess --- MLPipelines
    end

    %% ===== Relaciones entre capas (vertical) =====
    Presentation_Layer --> App_Layer
    App_Layer --> Auth_Layer
    Auth_Layer --> Domain_Layer
    Domain_Layer --> Data_Layer

    %% ===== Estilos: ocultar enlaces internos (0..9) =====
    %% (Ajustar índices si agregás/quitas líneas de enlace antes)
    linkStyle 0,1,2,3,4,5,6,7,8,9 stroke:transparent,stroke-width:0px

```

## Responsabilidades por módulo (conciso)
### Auth Layer

- AUTH: seguridad y autenticación. Emite/valida credenciales y autoriza llamadas HTTP entre capas.

### Domain / Logic Layer

- Insights Extractor: extrae insights a partir de texto ya normalizado; devuelve estructuras listas para usar por las apps.

- FileNormalizer: recibe audio, video, PDF e imágenes y produce texto comprensible por un LLM (incluye transcripción/OCR/limpieza).

- DBCRUD: expone operaciones CRUD del dominio vía HTTP y delegan el acceso físico a DB Access.

- Cache: provee caché de respuestas/datos intermedios para reducir latencia y costo (invalida por claves/reglas simples).

### Data / Services Layer

- LLMs: hoy actúa como wrapper de OpenAI; a futuro permitirá enrutar a múltiples LLMs (selección por tarea/costo/latencia).

- Vision (VisionModels): hoy wrapper básico; a futuro multi-proveedor de modelos de visión (clasificación, OCR avanzado, VLM).

- DB Access: recibe peticiones de DBCRUD y las rutea a la base correspondiente (p. ej., por esquema, tenant o tipo de dato).

- MachineLearning (MLPipelines): encapsula la lógica y pipelines de ML propios del producto (entrenamiento/inferencia básica por ahora).

### Notas operativas

Contrato de comunicación: interfaces HTTP claras entre capas (recursos, verbos y códigos de estado).

Evolución MCP: cuando esté habilitado, MCP permitirá a tools/agents invocar funciones del Domain Layer sin pasar por endpoints HTTP, manteniendo auditoría y policies equivalentes a AUTH.

Separación de responsabilidades: Presentation y App no acceden a Data directamente; todo pasa por AUTH → Domain, preservando el modelo de dominio y las políticas.



