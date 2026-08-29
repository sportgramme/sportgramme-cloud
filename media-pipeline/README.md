# Media Pipeline — Capability Briefs

> **Source:** `sportgramme-aws / media-pipeline/`  ·  consolidated from the former standalone repo
> **Audience:** hiring managers, technical due-diligence, investment partners.
> **House style:** every item below is a capability brief — *As a / I want / So that*
> plus a conceptual mermaid diagram. It describes **what the pipeline does and the
> control it gives the business**, not transport, storage, keys, or internal
> system names.

The media pipeline is how accredited contributors get stills, video and audio
**off a camera or phone at a venue and into Sportgramme's governed asset library**,
safely and at volume. It is designed so that:

- nothing leaves the contributor's device until the platform has authorised it,
- every transfer carries only the authority it was granted — no more,
- untrusted contributions are quarantined and moderated before they can be seen,
- and the platform's record of what was delivered is always the source of truth.

---

## Capability map

```mermaid
flowchart TD
    A[Select assets at the venue] --> B[Stage locally + attach match / taxonomy context]
    B --> C[Pre-flight: integrity, format & credit-line checks]
    C --> D[Duplicate check across the contributor's whole history]
    D --> E[Register batch with the platform]
    E --> F{Trust tier<br/>decided by the platform}
    F -->|Probationary| G[Quarantine area]
    F -->|Trusted| H[Published library, foldered by sport]
    E --> I[Per-file just-in-time delivery authorisation]
    I --> J[Direct device-to-cloud transfer<br/>least-authority]
    J --> K[Bounded concurrency + pause / stop / cancel]
    G --> L[Automated moderation gate]
    L -->|approved| H
    J --> M[Completion reconciled against the platform record]
    M --> N[Monitoring & storage visibility]
```

---

## EPIC — Contributors deliver match-day media into the governed library

**As** Sportgramme
**I want** accredited contributors to move large volumes of match-day media from
the field into the governed asset library through one controlled pipeline
**So that** the platform gains a steady supply of first-party imagery and footage
while retaining full control over what is admitted, where it lands, and who may
see it.

```mermaid
flowchart LR
    C[Contributor in the field] --> P[Media pipeline]
    P --> Q[Quarantine + moderation]
    P --> LIB[(Governed asset library)]
    Q --> LIB
    LIB --> SITE[Public site & syndication partners]
    P -.->|every step recorded| AUD[(Audit trail)]
```

*Tracked sub-issues below. Cross-cutting dependencies link to
`SG API Internal Landscape` (#3), `SG AWS Landscape` (#4) and
`SG Authorisations & Security` (#2).*

---

## 1. Stage a batch with match context before anything is sent

**As a** photojournalist at a fixture
**I want** to select a set of assets and attach the match, venue and
classification they belong to before any upload begins
**So that** every asset is correctly catalogued from the outset and I can review
the whole batch locally before committing it.

```mermaid
flowchart TD
    A[Choose assets] --> B[Attach: linked fixture / event / venue / date]
    B --> C[Attach: sport classification & syndication intent]
    C --> D[Local staging area — nothing uploaded yet]
    D --> E[Review, adjust or remove before commit]
```

Match linkage is optional — unattached assets are still accepted and routed to an
"unclassified" holding for later cataloguing.

---

## 2. Pre-flight integrity, format and credit-line checks

**As** the asset library owner
**I want** every staged asset checked for file integrity, an approved aspect
ratio, and a clean credit line before it can be queued
**So that** unusable, malformed or non-compliant assets are stopped at the
contributor's screen rather than consuming delivery capacity or polluting the
library.

```mermaid
flowchart TD
    A[Staged asset] --> B{Readable & not corrupt?}
    B -->|no| R1[Rejected with reason]
    B -->|yes| C{Approved aspect ratio?}
    C -->|no| R2[Rejected with reason]
    C -->|yes| D{Credit line present & policy-clean?}
    D -->|contains links / handles / contact details| R3[Rejected with reason]
    D -->|yes| E[Eligible to queue]
```

The pipeline also protects itself: if a previous session was left in a bad state
by a problem asset, the next load recovers cleanly rather than failing again.

---

## 3. Prevent duplicate assets across the contributor's whole history

**As** the platform
**I want** an asset that has already been processed — in this batch or any past
batch — to be recognised and refused
**So that** the library is not filled with re-uploads and contributors get a clear
pointer to where the original already lives.

```mermaid
flowchart TD
    A[Incoming asset] --> B[Identity = file identity + capture moment]
    B --> C{Seen before?}
    C -->|in the active batch| D[Blocked: already staged]
    C -->|in an earlier batch| E[Blocked: names the earlier batch]
    C -->|no| F[Accepted as unique]
```

---

## 4. Platform-authoritative batch registration and trust tiering

**As** Sportgramme
**I want** the platform — not the contributor's device — to assign the batch its
identity, its trust tier and its destination
**So that** a contributor can never elect to place their own material straight
into the published library, regardless of what their client sends.

```mermaid
flowchart TD
    A[Client requests batch registration] --> B[Platform issues batch identity]
    B --> C[Platform evaluates the contributor's standing]
    C --> D{Trust tier}
    D -->|Probationary| E[Destination = quarantine]
    D -->|Trusted| F[Destination = published library path]
    E & F --> G[Platform records the file manifest]
    G --> H[Client receives only the decisions, never the choice]
```

> **Depends on #3** — Core "register batch / manifest" contract.
> **Depends on #2** — contributor standing / trust grant.

---

## 5. Just-in-time, per-file delivery authorisation

**As** the security architect
**I want** delivery authorisation to be requested one file at a time, at the
moment of transfer, and to be short-lived
**So that** there is never a stockpile of reusable upload permits, and a permit
that leaks is worthless within moments.

```mermaid
sequenceDiagram
    participant Client
    participant Platform
    participant Cloud as Cloud storage
    Client->>Platform: Request permit for this one file, now
    Platform-->>Client: Short-lived, single-purpose permit
    Client->>Cloud: Transfer using the permit
    Note over Client,Cloud: Permit expires shortly after; not reusable
```

> **Depends on #3** — Core "just-in-time authorisation" contract.

---

## 6. Direct device-to-cloud transfer with least authority

**As** the platform
**I want** assets to stream straight from the contributor's device to cloud
storage, with the client sending only exactly what its permit covers
**So that** large files never round-trip through platform servers, and a transfer
cannot smuggle extra data or headers past what was authorised.

```mermaid
flowchart TD
    A[Authorised file] --> B[Open direct transfer to cloud]
    B --> C[Send only the fields the permit authorised]
    C --> D{Anything beyond the permit?}
    D -->|yes| E[Rejected by cloud storage]
    D -->|no| F[Accepted & stored]
    F --> G[Progress streamed back to the contributor]
```

> **Depends on #4** — cloud storage target, authorisation function, metadata contract.

---

## 7. Bounded concurrency, live telemetry and granular control

**As a** contributor uploading a large batch on venue Wi-Fi
**I want** a controlled number of parallel transfers, a live progress read on
each file, and the ability to pause, stop-and-reset or cancel any single file or
the whole batch
**So that** I can manage a slow or unreliable connection without losing the work
already done.

```mermaid
flowchart TD
    Q[Queued files] --> S{Free transfer slot?}
    S -->|yes| T[Start transfer · emit progress]
    S -->|no| Q
    T --> U[Per-file: pause · stop & reset · cancel]
    T --> V[Per-batch: pause · cancel]
    U & V --> W[In-flight transfers abort immediately]
```

---

## 8. Trust-tiered routing and the automated moderation gate

**As** Sportgramme
**I want** probationary contributions held in a quarantine area and passed through
an automated moderation gate before promotion, while trusted contributions land
directly in the library foldered by sport, with a distinct set for syndication
**So that** the published library only ever contains vetted material, and its
structure stays consistent no matter who contributed.

```mermaid
flowchart TD
    A[Delivered asset] --> B{Trust tier}
    B -->|Probationary| C[Quarantine]
    C --> D[Automated moderation gate]
    D -->|approved| E[Promoted to library]
    D -->|rejected| F[Held, not published]
    B -->|Trusted| E
    E --> G[Foldered by sport taxonomy]
    G --> H{Marked for syndication?}
    H -->|yes| I[Syndication set]
    H -->|no| J[Standard set]
```

> **Depends on #4** — quarantine store, moderation function, library layout.

---

## 9. Completion reconciled against the platform record

**As** the platform
**I want** a batch's final state to be settled by the platform's own record of
what was delivered, not by the contributor's device
**So that** a lost network response can never leave a delivered batch looking
failed, or a failed one looking complete.

```mermaid
flowchart TD
    A[All files finished locally] --> B[Ask the platform for its own counts]
    B --> C{Platform record}
    C -->|all delivered| D[Batch = Delivered]
    C -->|some errors| E[Batch = Delivered with errors]
    C -->|unreachable| F[Fall back to local state, flag for review]
    D & E & F --> G[Outcome recorded in the audit trail]
```

> **Depends on #3** — Core "batch file status / finalise" contract.

---

## 10. Monitoring and storage visibility

**As** an operations lead
**I want** a live view of in-flight and recent batches, the state of the cloud
storage targets, and how much storage each contributor is consuming
**So that** problems are visible as they happen and capacity can be managed ahead
of need.

```mermaid
flowchart LR
    A[Batch monitor] --> D[Ops dashboard]
    B[Cloud storage monitor] --> D
    C[Per-contributor storage usage] --> D
```

---

## Dependency summary

| Sub-issue | Needs from platform | Tracked on |
|---|---|---|
| 4 · Batch registration & trust tiering | Register-batch / manifest contract; contributor standing | #3, #2 |
| 5 · Just-in-time authorisation | Per-file short-lived authorisation contract | #3 |
| 6 · Direct device-to-cloud transfer | Cloud storage target; authorisation function; metadata contract | #4 |
| 8 · Trust-tiered routing & moderation | Quarantine store; automated moderation function; library layout | #4 |
| 9 · Completion reconciliation | Batch file-status / finalise contract | #3 |
| all | Capability code + access guard for the pipeline | #2 |
