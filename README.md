# ProofMint — Certificate Issuance & Verification Platform

**Stack:** React, Redux, TailwindCSS, Ant Design, Node.js, Express, PostgreSQL, Sequelize (no Prisma)

---

## 0) What you are building

A secure system that lets authorised users upload a PDF/image/Doc, place a draggable “stamp” (document ID, optional QR, verification URL), flatten it into a printable PDF, and store it with metadata. A public verification page allows anyone with the credential/ID to check validity and see a preview of the stamped document. Includes JWT authentication, document library, audit trail, optional expiry/revocation.

---

## 1) Deliverables & Definition of Done

* Separate **frontend** and **backend** repositories.
* **README** in each repo with setup, `.env.example`, migration/seed steps, and build/run commands.
* **Database**: PostgreSQL schema via Sequelize migrations + seed data for demo users and documents.
* **Auth**: JWT access + refresh tokens, role-based access control.
* **File handling**: Upload, secure storage, stamped-PDF generation, and previews.
* **Admin UI**: Login, dashboard, issue wizard, stamp designer (drag-and-drop), document library, document detail, settings.
* **Public UI**: `/verify` form → result page (valid/invalid/expired/revoked) + preview thumbnail and metadata.
* **API contract** implemented with request validation and clear error formats.
* **Audit logs** for critical actions (login, create, stamp, render, revoke, verify).
* **Rate limit**: `/verify` must be protected from abuse. Suggestion: max **20 requests per IP per 10 minutes**, adjustable via config.

---

## 2) User roles & access

* **Admin**: Full access to issue, stamp, render, list, revoke, manage templates, manage users.
* **Issuer**: Can issue/stamp/render/list documents but not manage users.
* **Public**: Can only access `/verify` flow.

JWT required for all admin/issuer endpoints. Public endpoints are only `/v/:docId` and `/verify`.

---

## 3) Key user journeys

**Issue & Stamp:** Admin uploads file, enters metadata, drags stamp onto doc, flattens stamped version, saves, issues.
**Verify:** Public user enters credential or scans QR, system checks and shows result + preview.
**Library:** Admin sees list of documents, filters, opens detail, revokes if needed.

---

## 4) Data model (for guidance only)

These Sequelize models are suggested to get started; you can add/remove/modify as per requirements.

**users**

* id (UUID, PK), email, password\_hash, role, is\_active, created\_at, updated\_at

**documents**

* id (UUID, PK), title, recipient\_name/email/phone, description, issuer\_user\_id, status (draft/issued/revoked), issue\_date, expiry\_date, revoke\_reason, created\_at, updated\_at

**document\_files**

* id, document\_id, kind (source/stamped/preview), storage\_path, mime\_type, size\_bytes, page\_count, created\_at

**stamps**

* id, document\_id, page\_number, x\_norm/y\_norm/width\_norm/height\_norm, show\_qr, show\_id\_text, show\_verify\_url, style\_json, created\_at, updated\_at

**audit\_logs**

* id, actor\_user\_id, document\_id, action, meta\_json, created\_at

**verifications**

* id, document\_id, query\_value, result (valid/invalid/expired/revoked), client\_ip, user\_agent, created\_at

---

## 5) API contract (for guidance only)

These endpoints are starting points; you can change/add/remove as needed.

**Auth**

* `POST /auth/login`, `POST /auth/refresh`, `POST /auth/logout`

**Documents**

* `POST /documents`, `POST /documents/:id/upload`, `POST /documents/:id/stamps`, `POST /documents/:id/render`,
* `POST /documents/:id/issue`, `POST /documents/:id/revoke`,
* `GET /documents`, `GET /documents/:id`, `GET /documents/:id/download`, `GET /documents/:id/preview`

**Public verify**

* `GET /v/:docId` → { status, metadata }
* `POST /verify` → { result, meta, preview\_url }

---

## 6) Backend implementation notes

* **Auth**: JWT (access \~15 min, refresh \~7–30 days). Hash passwords with argon2/bcrypt.
* **Validation**: Use any schema validation library.
* **File upload**: Use multer or equivalent. Enforce type/size limits.
* **PDF/image processing**: Can use `pdf-lib`, `sharp`, `qrcode`, `node-canvas`, or any other equivalent.
* **DOCX**: Either reject with a clear message or convert to PDF using any utility.
* **Coordinates**: Store normalised values for consistent rendering.
* **Security**: Never expose internal paths. Public preview should be watermarked.
* **Rate-limit**: `/verify` must enforce realistic limits (e.g., 20 req/10 min/IP).

---

## 7) Frontend implementation notes

* **Pages**: Login, dashboard, documents (list, create, detail), settings, verify.
* **Designer**: Use `react-pdf`/`react-konva`/`fabric.js` or equivalents to allow drag/drop of stamp on preview.
* **Library UI**: AntD Table with filters, status badges.
* **Verification UI**: Simple form, clear valid/invalid/expired/revoked screen with preview.
* **Styling**: Tailwind for layout and typography, AntD for components.

---

## 8) Verification logic & statuses

* **valid**: exists, issued, not expired.
* **expired**: current time > expiry\_date.
* **revoked**: status=revoked (show revoke\_reason).
* **invalid**: not found.

Result page shows status + digest info (doc ID, recipient, issuer, issue date, expiry if any).

---

## 9) Configuration & environment

Provide `.env.example` for both apps.

Here is a sample env for you to get started, add/modify as needed.

**Backend**

```
PORT=4000
DATABASE_URL=postgres://user:pass@localhost:5432/proofmint
JWT_ACCESS_SECRET=…
JWT_REFRESH_SECRET=…
JWT_ACCESS_TTL=15m
JWT_REFRESH_TTL=7d
UPLOAD_DIR=./storage
PUBLIC_BASE_URL=https://your-domain.example
VERIFICATION_PATH=/v
MAX_UPLOAD_MB=15
VERIFY_RATE_LIMIT=20:10m
```

**Frontend**

```
VITE_API_BASE_URL=http://localhost:4000/api
VITE_PUBLIC_VERIFY_URL=http://localhost:5173/verify
```

---

## 10) Optional/bonus features

* Email delivery of stamped certificates.
* Revocation list page.
* Stamp templates with variables.
* Multi-tenant support.
* Webhooks for events.
* API keys for programmatic issuance.
* Digital signatures (cryptographic).
* Document hash & anti-tamper fields.
* Analytics dashboard.

---

**Project name:** **ProofMint**
**Tagline:** Issue, stamp, and verify documents—simply and securely.
