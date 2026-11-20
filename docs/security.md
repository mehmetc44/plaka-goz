# Security Architecture

Because Plaka Göz operates inside an institutional campus security network and handles surveillance imagery, access control and data protection are enforced at multiple layers.

---

## 1. Authentication & Role-Based Access Control (RBAC)
- Built on **ASP.NET Core Identity** with role-based authorization policies across controllers and SignalR hubs.
- Operators, supervisors, and administrators are restricted to actions permitted by their assigned roles (e.g., viewing live feeds vs. modifying blacklisted plates or generating reports).

## 2. JWT & Revocable Refresh Tokens
- **Short-lived JWT Access Tokens:** Minimize exposure window if an access token is intercepted.
- **Database-Controlled Refresh Tokens:** Refresh tokens are persisted in PostgreSQL, enabling immediate server-side revocation to terminate compromised or expired operator sessions.

## 3. Surveillance Media Protection (MinIO)
- Camera snapshots (vehicle, plate, full-scene) are stored in private MinIO buckets.
- Images are never served from public URLs; the backend issues **time-limited presigned URLs** only to authenticated requests.

## 4. Network & Repository Isolation
- Deployed strictly inside the institutional network; source code and real operational data remain private.

