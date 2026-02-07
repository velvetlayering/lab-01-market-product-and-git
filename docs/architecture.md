## Product Choice

Telegram
https://telegram.org/
Telegram is a cloud-based mobile and desktop messaging app with a focus on security and speed.

## Main components

![Telegram Component Diagram](/../../../docs/diagrams/out/telegram/component-diagram/Component%20Diagram.svg)
[Telegram Component Diagram Code](../../../docs/diagrams/src/telegram/component-diagram.puml)

### Desktop App

Desktop App is the application used on personal computers and laptops.

### Web Client

Web Client is a great portable solution for users which have no access to installing applications on their computers and laptops.

### Bot API Server

Bot API Server guarantees synergy with Telegram's features and capabilties.

### Notification/Updates Service

Notification/Updates Service makes sure users do not miss a single notification from their important contacts.

### Media & File Service

Media & File Service provides quick file transmission between users.

![Telegram Sequence Diagram](/../../../docs/diagrams/out/telegram/sequence-diagram/Sequence%20Diagram.svg)
[Sequence Diagram](../../appendix/architectural-views.md#sequence-diagram)

### Group: Media Upload & Initial Processing (top box)

#### What happens in this group

This group covers the moment Alice sends a photo and the system securely uploads and registers the media before it becomes a message.
The goal here is:

- authenticate the client,
- upload the raw media bytes,
- store the media,
- return a reference that can later be used in a message.

---

Components involved

- User Alice
- Mobile App
- MTProto Gateway
- Auth Service
- Media Service
- Distributed File Storage (DFS)

---

#### Step-by-step interaction and data exchange

1. Alice → Mobile App

- Action: Alice selects and sends a photo.
- Data: Local photo file (raw bytes + metadata like size, type).

2. Mobile App → MTProto Gateway

- Action: App initiates a secure MTProto request.
- Data:
  - Encrypted media chunks
  - Session/auth token
  - Upload metadata (file size, MIME type)

3. MTProto Gateway → Auth Service

- Action: Validate Alice’s session.
- Data:
  - Auth token / session ID

4. Auth Service → MTProto Gateway

- Action: Authentication result.
- Data:
  - “OK” + user ID (or failure, if invalid)

5. MTProto Gateway → Media Service

- Action: Forward authenticated media upload.
- Data:
  - User ID
  - Media chunks
  - Upload parameters

6. Media Service → Distributed File Storage

- Action: Persist the media.
- Data:
  - Raw photo bytes (often chunked)
  - Storage key / shard info

7. Distributed File Storage → Media Service

- Action: Confirm storage.
- Data:
  - File ID / media hash / storage location

8. Media Service → MTProto Gateway → Mobile App

- Action: Upload completes.
- Data:
  - Media reference (media_id, access_hash, thumbnail info)

---

#### Result of this group

- The photo is securely stored in distributed storage.
- The client receives a media reference, not the raw file.
- This reference will be used in the next group to create and propagate the actual message.

### Client Side

- Mobile App
  - Deployed on end-user devices (smartphones, tablets).
  - Runs outside the provider’s infrastructure.
  - Communicates over the public Internet using encrypted connections.

---

### Edge / Access Layer

- MTProto Gateway
  - Deployed in globally distributed edge data centers.
  - Located close to users geographically to reduce latency.
  - Terminates client connections, handles encryption, and routes requests inward.

---

### Core Backend (Application Layer)

- Auth Service
  - Deployed in central backend clusters (multiple regions for redundancy).
  - Stateless or lightly stateful; often horizontally scalable.
  - Handles authentication, sessions, and identity validation.
- Media Service
  - Deployed in backend application clusters.
  - Often co-located with storage access layers for throughput.
  - Manages uploads, downloads, and media metadata.

---

### Storage Layer

- Distributed File Storage (DFS)
  - Deployed across multiple data centers and availability zones.
  - Stores replicated media data for fault tolerance.
  - Optimized for large binary objects and high throughput.

---

### Networking Overview

- Client ↔ Gateway: Public Internet (encrypted)
- Gateway ↔ Backend Services: Private internal network
- Backend ↔ Storage: High-bandwidth internal data center network

---

### Key Deployment Properties

- Geographic distribution for low latency
- Horizontal scalability at gateways and services
- Replication and redundancy at the storage layer

## Assumptions:

- I assumed the MTProto Gateway is deployed in multiple edge locations close to users, even though the diagram does not explicitly show regions or latency optimization.
- I assumed services like Auth Service and Media Service are designed to scale horizontally and do not keep critical state in memory.

## Questions:

- How does the system behave if a gateway or media node fails mid-upload?
- How are encryption keys managed internally, and where exactly does encryption/decryption occur beyond the gateway?
