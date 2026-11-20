# Deployment

Plaka Göz is deployed in production on a Linux server using Docker, with an additional PowerShell automation path for Dockerless Windows Server environments.

---

## 1. Production Linux Deployment (Docker)

All core components run as isolated containerized services:

| Service | Role |
|---|---|
| **Backend API (.NET 8)** | REST API, CQRS handlers, Hangfire worker, SignalR Hub |
| **Frontend (Angular)** | Operator monitoring & reporting SPA |
| **PostgreSQL** | Plate registry, event logs, Identity & refresh tokens |
| **MinIO** | S3-compatible object storage for event snapshots |
| **MediaMTX (+ FFmpeg)** | RTSP ingest and HLS stream packaging |

---

## 2. Windows Server Fallback (PowerShell)

For institutional Windows Server environments where Docker is not available, a dedicated **PowerShell installation script** automates:
- Installing required runtime and media dependencies.
- Configuring MediaMTX, MinIO, and the .NET application.
- Registering components as **automatically starting Windows services** to ensure recovery after server reboots.

