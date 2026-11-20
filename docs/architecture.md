# System Architecture

Plaka Göz follows **Onion Architecture** on the backend (C# / .NET 8) to keep core domain rules decoupled from databases, object storage, messaging, and video infrastructure.

![Architecture Overview](../assets/architecture-overview.svg)

---

## High-Level Layers

| Layer | Responsibility | Key Technologies |
|---|---|---|
| **Domain** | Core entities (plates, vehicles, camera feeds, alarm events, categories), enums (`VIP`, `Blacklisted`, `Personnel`), domain rules | C#, .NET 8 |
| **Application** | Use cases, commands/queries, validation, DTOs, repository & service abstractions | MediatR, CQRS |
| **Persistence** | Relational data access, migrations, incremental filtering queries, refresh token storage | Entity Framework Core, PostgreSQL |
| **Infrastructure** | Background job execution, object storage presigned URLs, identity & JWT generation | Hangfire, MinIO SDK, ASP.NET Core Identity |
| **Presentation / Real-Time** | REST endpoints, SignalR hub broadcasting, live HLS playback & event dashboard | ASP.NET Core, SignalR, Angular |

---

## Core Architectural Principles

1. **Dependency Inversion:** Outer layers (Persistence, Infrastructure, API) depend on interfaces defined in the Application/Domain core—never the reverse.
2. **CQRS with MediatR:** Read operations (incremental event history, status/date filtering, reporting) are separated from write operations (plate classification, category updates, event ingestion).
3. **Decoupled Pipelines:** Live video streaming ([Video Pipeline](video-pipeline.md)) and plate recognition event handling ([Alarm Event Pipeline](alarm-event-pipeline.md)) operate independently so heavy video I/O never stalls alarm delivery.

