# Design Decisions

Key architectural decisions made during the design and production rollout of Plaka Göz.

---

## 1. Onion Architecture + CQRS (MediatR)
- **Context:** The system integrates multiple external concerns (RTSP/HLS media, MinIO object storage, Hangfire, SignalR, PostgreSQL).
- **Decision:** Adopt Onion Architecture with CQRS/MediatR handlers.
- **Trade-off:** Slightly more boilerplate than a simple CRUD service, in exchange for clean separation of business rules, easy testability, and zero infrastructure leakage into domain logic.

## 2. Hangfire for Asynchronous Event Processing
- **Context:** 12 cameras can emit recognition events concurrently with high-resolution image payloads.
- **Decision:** Offload classification, DB persistence, MinIO uploads, and SignalR dispatch to Hangfire background jobs.
- **Benefit:** Camera webhook endpoints stay non-blocking, preventing dropped events during peak campus traffic.

## 3. HLS via MediaMTX Instead of Direct RTSP in Browser
- **Context:** Modern browsers do not natively play raw RTSP streams without plugins or heavy WebRTC signaling overhead.
- **Decision:** Package RTSP streams into HLS using FFmpeg + MediaMTX.
- **Benefit:** Reliable multi-camera playback (1, 4, and 12-grid layouts) across standard browsers with minimal client complexity.

## 4. MinIO with Time-Limited Presigned URLs
- **Context:** Storing surveillance images directly on web-server disk or public URLs poses security and scaling risks.
- **Decision:** Store images in private MinIO buckets and generate expiring access links per request.
- **Benefit:** Sensitive campus surveillance imagery remains protected even if a URL is shared or cached.

## 5. Database-Backed Revocable Refresh Tokens
- **Context:** Purely stateless JWTs cannot be invalidated immediately when an operator leaves or a session must be terminated.
- **Decision:** Pair short-lived JWT access tokens with refresh tokens tracked in PostgreSQL.
- **Benefit:** Enables instant session revocation from the server side.

