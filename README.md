# Event-Driven Architecture
%% Event-Driven API / Daten­pipeline  (LR = Left-to-Right)
graph LR
  %% Knoten -----------------------------------------------------------------
  E[Event Source<br/>(Event Grid Topic)]
  F[Azure Function<br/>(Trigger&nbsp;& Orchestration)]
  API[External REST API]
  RAW[ADLS Gen2<br/>Raw Zone]
  DBX[Azure Databricks<br/>Notebook (PySpark)]
  DL[Delta Lake<br/>Bronze / Silver / Gold]
  CONS[Downstream Consumers<br/>(Power BI, Synapse)]
  MON(Azure Monitor<br/>+ Application Insights)
  DLQ(Service Bus<br/>Dead-Letter Queue)

  %% Kanten -----------------------------------------------------------------
  E --> |event| F
  F --  |"call<br/>(retry, auth)"| API
  API --|response| F
  F --> |"write raw<br/>(JSON)"| RAW
  F -.-> |"on failure"| DLQ           %% gestrichelt
  RAW -->|"Databricks Job<br/>(webhook)"| DBX
  DBX -->|"clean &amp; transform"| DL
  DL --> |"query / report"| CONS

  %% Monitoring (gestrichelte Hilfslinien) ----------------------------------
  F -.-> MON
  DBX -.-> MON
  API -.-> MON

  %% Optionale Layout-Tweaks (Abstände, Farben) -----------------------------
  classDef dashed stroke-dasharray: 5 5;
  linkStyle 4 class:dashed;  %% DLQ-Edge
  linkStyle 8,9,10 class:dashed;  %% Monitoring-Edges
