# System Architecture

Plaka Göz follows **Onion Architecture** on the backend (C# / .NET 8) to keep core domain rules decoupled from databases, object storage, media servers, and real-time messaging infrastructure.

![Architecture Overview](../assets/architecture-overview.svg)

---

## End-to-End System Flow

```mermaid
flowchart TB
    subgraph VIDEO["Video Pipeline"]
        S["Settings UI\n(IP & Password)"] -. "Connect" .-> CAM_V["IP Cameras (RTSP)"]
        CAM_V -- "RTSP Stream" --> FFM["FFmpeg\n(Read / Transcode / HLS)"]
        FFM --> MTX["MediaMTX\n(HLS / WebRTC / RTSP)"]
    end

    subgraph ALARM["Alarm Event Pipeline"]
        CAM_A["Hikvision ISAPI\n(XML + Images)"] --> AES["AlarmEventService"]
        AES --> VAL["Validator\n(Integrity Check)"]
        VAL --> PAR["XML Parser\n(Data & Images)"]
        PAR --> HF["Hangfire Background Job"]
        HF --> MIN[("MinIO\nExpiring Image URLs")]
        HF --> PG[("PostgreSQL\nSave & Check Blacklist / VIP")]
        HF --> SIG["SignalR Hub"]
    end

    MTX -- "HLS Live Feeds (1 / 4 / 12)" --> UI["Angular UI"]
    SIG -- "Real-Time Alerts (Red Alert if Blacklisted)" --> UI
```

---

## Onion Architecture Layers

| Layer | Responsibility | Key Components & Technologies |
|---|---|---|
| **Domain** | Core entities (`Plate`, `Vehicle`, `Camera`, `AlarmEvent`), classification enums (`VIP`, `Blacklisted`, `Personnel`), domain rules | C#, .NET 8 |
| **Application** | Use cases, commands/queries, validation rules, repository & service interfaces | MediatR, CQRS |
| **Persistence** | Relational data access, EF Core mappings, Blacklist/VIP lookups, incremental filtering queries, refresh token storage | Entity Framework Core, PostgreSQL |
| **Infrastructure** | `AlarmEventService` (Hikvision ISAPI listener), `Validator`, `XML Parser`, Hangfire job orchestration, MinIO presigned URLs, Identity & JWT | Hangfire, MinIO SDK, ASP.NET Core Identity |
| **Presentation / Real-Time** | REST API endpoints, SignalR hub broadcasting, Settings & live monitoring dashboard | ASP.NET Core, SignalR, Angular |

---

## Core Architectural Principles

1. **Dependency Inversion:** Outer layers (Persistence, Infrastructure, Presentation) depend on interfaces defined in the Application/Domain core—never the reverse.
2. **CQRS with MediatR:** Read operations (incremental event history, status/date filtering, reporting) are separated from write operations (event ingestion, automatic unseen-plate registration, one-click category updates).
3. **Decoupled Pipelines:** Live video streaming ([Video Pipeline](video-pipeline.md)) and Hikvision ISAPI alarm processing ([Alarm Event Pipeline](alarm-event-pipeline.md)) operate independently so heavy video I/O never stalls real-time plate alerts.
