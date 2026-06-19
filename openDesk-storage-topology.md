# openDesk Storage Topology

> **Purpose**: Documents which components consume which storage backends, protocols used, and backup strategy.
> **Relation**: Companion to [`openDesk-dependency-graph.mmd`](openDesk-dependency-graph.mmd) — the main graph covers runtime architecture; this file covers data persistence.

---

## 1. Component × Storage Matrix

| Component | PostgreSQL | MariaDB | Cassandra | S3/MinIO | Redis | Memcached | PVC |
|-----------|-----------|---------|-----------|----------|-------|-----------|-----|
| **Keycloak / Nubus** | ✅ Config, Sessions | — | — | ✅ Portal Assets | — | ✅ UMC Sessions | — |
| **OpenLDAP** | — | — | — | — | — | — | ✅ LDAP Data |
| **OX AppSuite** | — | ✅ ConfigDB, PRIMARYDB_n | — | ✅ Attachments | ✅ Session Cache | — | — |
| **OX Dovecot** | — | — | ✅ Metadata, ACLs (EE) | ✅ Mail Store (EE) | — | — | ✅ Mail Spool |
| **Postfix-Base** | — | — | — | — | — | — | ✅ Spool, Config |
| **Postfix-OX** | — | — | — | — | — | — | (via Dovecot) |
| **Nextcloud** | ✅ Metadata | — | — | ✅ File Data | ✅ Locking, Cache | — | — |
| **Collabora** | — | — | — | — | — | — | (ephemeral) |
| **CryptPad** | — | — | — | — | — | — | ✅ Pad Data |
| **OpenProject** | ✅ Metadata | — | — | ✅ Attachments | — | ✅ Sessions, Cache | ✅ Repos, Assets |
| **XWiki** | ✅ Content | — | — | — | — | — | ✅ Attachments |
| **Element/Synapse** | ✅ Messages, Rooms | — | — | — | — | — | ✅ Media Store |
| **Jitsi** | — | — | — | — | — | — | (ephemeral) |
| **Notes** | — | — | — | — | — | — | ✅ Content |
| **Intercom** | — | — | — | — | ✅ Session, Queue | — | — |
| **OX Connector** | — | — | — | — | — | — | ✅ Config |
| **ClamAV** | — | — | — | — | — | — | ✅ Virus DB |
| **NATS (AMQ)** | — | — | — | — | — | — | ✅ Queue Store |

---

## 2. Backend Detail

| Backend | Technology | Protocol | Persistence | Backup | State |
|---------|-----------|----------|-------------|--------|-------|
| **PostgreSQL** | CloudNativePG / StatefulSet | TCP 5432 | ✅ Durable | ✅ Tier A (1h RPO) | Stateful |
| **MariaDB** | StatefulSet (Galera) | TCP 3306 | ✅ Durable | ✅ Tier A (1h RPO) | Stateful |
| **Cassandra** | StatefulSet | TCP 9042 | ✅ Durable | ✅ | Stateful (EE only) |
| **S3/MinIO** | MinIO Operator / SeaweedFS | HTTPS 443 | ✅ Durable | ✅ Tier A | Stateful |
| **Redis** | StatefulSet | TCP 6379 | ❌ Cache / AOF | ❌ Rebuildable | Stateless |
| **Memcached** | Deployment | TCP 11211 | ❌ Cache only | ❌ Rebuildable | Stateless |
| **PVC (RWX)** | CephFS (HDD EC) | Filesystem | ✅ Durable | ✅ k8up | Stateful |
| **PVC (RWO)** | Ceph RBD (SSD) | Block | ✅ Durable | ❌ Excluded (k8up) | Stateful |

> **Note**: RWO-PVCs are currently excluded from k8up backups because backup pods cannot mount them while in use. Strategy needed (CSI Snapshots, Per-Node Schedules, or application-level backup).

---

## 3. Data Flow Detail

### PostgreSQL — which component creates which data

| Component | Key Tables / Data | Size Estimate |
|-----------|------------------|---------------|
| Keycloak | `REALM`, `USER_ENTITY`, `FED_USER`, `EVENT_ENTITY` | Small–Medium |
| Nextcloud | `oc_filecache`, `oc_storages`, `oc_share`, `oc_jobs` | Medium–Large |
| OpenProject | `projects`, `work_packages`, `journals`, `attachments` | Medium |
| XWiki | `xwikidoc`, `xwikiattachment`, `xwikiobjects` | Small–Medium |
| Element/Synapse | `events`, `state_groups`, `room_memberships` | Large (chat history) |

### S3 Bucket Layout

| Bucket / Path | Used By | Contents |
|--------------|---------|----------|
| `nubus-portal` | Keycloak / Portal | Static portal assets |
| `nextcloud-files` | Nextcloud | User file data (primary) |
| `openproject-files` | OpenProject | Work package attachments |
| `ox-attachments` | OX AppSuite | Mail/file attachments |
| `dovecot-mail` | Dovecot | Mail store (EE) |

---

## 4. Backup Tier Model

| Tier | Criticality | Components | RPO | RTO | Retention | Strategy |
|------|------------|-----------|-----|-----|-----------|----------|
| **A** | Critical | Keycloak, PostgreSQL, MariaDB, MinIO, Redis | 1h | 2h | 30d | k8up + frequent snapshots |
| **B** | Important | Nextcloud, OX AppSuite, OpenProject, XWiki, Element | 1h | 4h | 14d | k8up daily + incremental |
| **C** | Experimental | JupyterHub, Ollama, Dask (edu stack) | 24h | 1d | 7d | Basic snapshot |

> **Current tooling**: k8up v2.13.0 → S3 (s3://s3.hrz.uni-marburg.de/backups)
> **Active**: 6 RWX PVCs (clamav, dovecot, opencloud, seaweedfs, slidev)
> **Excluded**: 29 RWO PVCs (pending CSI-snapshot strategy)

---

## 5. Cache & Ephemeral Data

These backends hold **no durable state** — they are rebuilt on restart:

| Backend | Components | Eviction Policy | Purpose |
|---------|-----------|----------------|---------|
| **Redis** | Nextcloud, OX AppSuite, Intercom | LRU + TTL | Session store, lock registry, job queue |
| **Memcached** | Keycloak (UMC), OpenProject | LRU | Session cache, page cache |
| **Cluster-internal** | Collabora, Jitsi | Process-bound | Document buffers, media streams |

---

## Reference

- Main runtime graph: [`openDesk-dependency-graph.mmd`](openDesk-dependency-graph.mmd)
- Upstream deployment: <https://gitlab.opencode.de/bmi/opendesk/deployment/opendesk>
- k8up docs: <https://k8up.io/>
- Cluster: 9 nodes, K3s v1.32.3, Ceph CSI (RBD SSD + CephFS HDD EC)
