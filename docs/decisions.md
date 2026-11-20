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

## 6. Adaptive Stream Resolution for 4 and 12-Camera Layouts
- **Context:** Decoding and rendering 12 concurrent high-resolution HLS video streams in a single browser tab overwhelms client CPU/GPU resources, memory, and bandwidth, leading to UI freezes or browser crashes.
- **Decision:** Serve lower-resolution streams when operators switch to **4-camera** or **12-camera** grid views, and reserve full-resolution playback for the **1-camera** focus layout.
- **Benefit:** Enables smooth, crash-free simultaneous monitoring across all 12 camera feeds in standard browsers while preserving full visual detail whenever an operator zooms into a single camera.

## 7. MediaMTX as the Media Relay & Protocol Bridge
- **Context:** Multiple operators watching 12 cameras simultaneously would overload IP cameras if each browser opened its own RTSP connection, and routing video streams through the .NET backend would degrade API and alarm responsiveness.
- **Decision:** Introduce **MediaMTX** as a lightweight, dedicated media server that ingests a single stream per camera (via FFmpeg) and fans it out to all clients while supporting HLS, WebRTC, and RTSP from one place.
- **Benefit:** Keeps camera load constant, isolates heavy video I/O from the .NET 8 application server, and allows dynamic camera onboarding from the Settings UI.

## 8. HLS Over WebRTC for Multi-Camera Browser Playback
- **Context:** Opening 12 concurrent WebRTC `RTCPeerConnection` sessions (with ICE/DTLS/UDP ports) in a single browser tab strains client media threads and frequently conflicts with strict institutional firewalls and NAT policies.
- **Decision:** Use **HLS** over standard HTTP/HTTPS for 1, 4, and 12-camera playback, while relying on the **Hikvision ISAPI → SignalR** pipeline for sub-second real-time plate alerts.
- **Benefit:** Delivers firewall-friendly, stable multi-hour playback across 12 concurrent feeds without plugin or STUN/TURN overhead, while still keeping WebRTC available via MediaMTX if ultra-low-latency single-camera mode is ever needed.

