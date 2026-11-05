flowchart TD
    %% Definición de Carriles (Swimlanes)
    subgraph Usuario ["Usuario (Capturista ERP)"]
        direction TB
        E1((Inicio:\nCrear/Editar\nArtículo)) --> T1[Capturar Identidad:\nSKU, Descripción y Grupo]
        T1 --> T2[Asignar Perfil Operativo:\nVenta, Compra, Inventario]
        T2 --> T3[Configurar UoM, \nReglas Impositivas y BOM]
        
        E2((Inicio:\nEliminar\nArtículo)) --> T4[Solicitar Baja\nde Registro]
    end

    subgraph Sistema ["Sistema SAP (Reglas y Validaciones)"]
        direction TB
        T3 --> G1{¿Datos obligatorios\ncompletos?}
        G1 -- No --> Err1([Bloqueo: Faltan\ndatos requeridos])
        
        G1 -- Sí --> G2{¿SKU es único y\nsin símbolos\nprohibidos?}
        G2 -- No --> Err2([Bloqueo: SKU\ninválido o duplicado])
        
        G2 -- Sí --> G3{¿Integridad\nReferencial OK?}
        G3 -- No --> Err3([Bloqueo: Atributo\nInexistente])
        
        G3 -- Sí --> T5[Validar Determinación\nde Cuentas]
        
        T4 --> G4{¿Cuenta con\nhistorial\ntransaccional?}
        G4 -- Sí --> T6[Bloquear borrado físico:\nCambiar a 'Inactivo']
        G4 -- No --> T7[Ejecutar Borrado\nFísico]
    end

    subgraph BD ["Base de Datos ERP y Auditoría"]
        direction TB
        T5 --> T8[(Guardar Registro\nMaestro)]
        T6 --> T8
        T7 --> T8
        
        T8 --> T9[Generar Log Inmutable\n(Rastreo de Auditoría)]
        T9 --> F1(((Fin:\nProceso\nCompletado)))
    end

    %% Estilos BPMN 
    classDef startEvent fill:#d4edda,stroke:#28a745,stroke-width:2px;
    classDef endEvent fill:#f8d7da,stroke:#dc3545,stroke-width:3px;
    classDef task fill:#cce5ff,stroke:#004085,stroke-width:1px;
    classDef gateway fill:#fff3cd,stroke:#856404,stroke-width:2px,shape:diamond;
    classDef error fill:#f8d7da,stroke:#721c24,stroke-width:1px;

    %% Asignación de estilos
    class E1,E2 startEvent;
    class F1 endEvent;
    class T1,T2,T3,T4,T5,T6,T7,T8,T9 task;
    class G1,G2,G3,G4 gateway;
    class Err1,Err2,Err3 error;
