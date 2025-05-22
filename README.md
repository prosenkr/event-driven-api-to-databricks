# Event-gesteuerte API-Triggerung: Architektur Skizze
```mermaid
flowchart LR
    %% ---------- Kernpfad ----------
    E["Event-Quelle<br/>(Event Grid Topic)"]
    F["Azure Function<br/>(Trigger & Orchestration)"]
    API(["External REST API<br/>(Circuit-Breaker)"])
    RAW[(ADLS Gen2<br/>Raw Zone)]
    LOGIC["Logic App<br/>(Trigger, Steuerung, Fehlerhandling)"]
    DBX(["Azure Databricks<br/>(PySpark)"])
    DL[(Delta Lake<br/>Bronze / Silver / Gold)]
    CONS(["Downstream Consumers<br/>(Power BI, Synapse)"])

    E -->|event| F
    F -->|"call (retry + back-off)"| API
    API -->|response| F
    API -.->|on failure| Fallback((Fallback))
    Fallback --> F
    F -->|"write raw (JSON)"| RAW
    RAW -->|"blob event"| LOGIC
    LOGIC -->|"trigger job"| DBX
    DBX -->|"clean & transform"| DL
    DL -->|"query / report"| CONS

    %% ---------- Fehler- & Retry-Pfade ----------
    DLQ((Service Bus<br/>Dead-Letter Queue))
    F -.->|on failure| DLQ

    EGDLQ((Event Grid DLQ))
    E -.->|dead-letter| EGDLQ
    EGDLQ -.->|reprocess| F

    %% ---------- Security / Governance ----------
    KV(["Key Vault +<br/>Managed Identity"])
    F -.->|secrets| KV

    %% ---------- Data Quality ----------
    DQ(["Data Quality<br/>(Great Expectations)"])
    DBX -.-> DQ

    %% ---------- Monitoring & Alerting ----------
    MON((App Insights +<br/>Log Analytics))
    ALERTS((Alert Rules / Dashboards))

    F -.-> MON
    DBX -.-> MON
    API -.-> MON
    MON --> ALERTS

    %% ---------- CI / CD ----------
    CI(["CI/CD via GitHub Actions<br/>(IaC + Unit Tests + Staging/Prod)"])
    CI -.-> F
    CI -.-> DBX
    CI -.-> DL

    %% ---------- Farben / Styles ----------
    style E fill:#cce5ff,stroke:#333,stroke-width:1px
    style F fill:#cce5ff,stroke:#333,stroke-width:1px
    style LOGIC fill:#cce5ff,stroke:#333,stroke-width:1px

    style API fill:#f9e79f,stroke:#333,stroke-width:1px
    style CONS fill:#f9e79f,stroke:#333,stroke-width:1px

    style RAW fill:#d4efdf,stroke:#333,stroke-width:1px
    style DL fill:#d4efdf,stroke:#333,stroke-width:1px

    style DBX fill:#e8daef,stroke:#333,stroke-width:1px

    style MON fill:#d1f2eb,stroke:#333,stroke-width:1px
    style ALERTS fill:#d1f2eb,stroke:#333,stroke-width:1px

    style KV fill:#fcf3cf,stroke:#333,stroke-width:1px

    style DQ fill:#fadbd8,stroke:#333,stroke-width:1px

    style DLQ fill:#f5b7b1,stroke:#333,stroke-width:1px
    style EGDLQ fill:#f5b7b1,stroke:#333,stroke-width:1px
    style Fallback fill:#f5b7b1,stroke:#333,stroke-width:1px

    style CI fill:#d6eaf8,stroke:#333,stroke-width:1px
```
### Legende Farbcodierung

| Farbe        | Kategorie                   |
|--------------|-----------------------------|
| 🟦 Blau | Event / Steuerung           | 
| 🟨 Gelb | Externe Systeme             |
| 🟩 Grün | Speicherung                 |
| 🟪 Lila | Verarbeitung                |
| 🟦 Türkis | Monitoring & Alerting       |
| 🟧 Beige | Security / Governance       | 
| 🟥 Rot | Fehlerpfade / Fallback / DLQ| 
| 🟦 Hellblau | CI / CD                     |
| 🟥 Rosa | Data Quality                |

### Legende Formen

| Form                         | Bedeutung                           |
|------------------------------|--------------------------------------|
| Rechteck                  | Standardprozess / Funktion / Logik   |
| Abgerundetes Rechteck     | Externe Systeme / Dienste            |
| Zylinder                  | Datenspeicher / Datenhaltung         |
| Ellipse                  | Sonderrolle: Monitoring, Fehler, DLQ |
