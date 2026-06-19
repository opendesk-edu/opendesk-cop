# openDesk Community of Practice — 19. Juni 2026

- **Datum**: Freitag, 19. Juni 2026 | **Zeit**: 14:00–15:30 CET
- **Ort**: [BigBlueButton](https://webconf.hrz.uni-marburg.de/n/rooms/7gq-zdy-zje-roq/join)
- **Repo**: [github.com/opendesk-edu/opendesk-cop](https://github.com/opendesk-edu/opendesk-cop) — [codeberg.org/opendesk-edu/opendesk-cop](https://codeberg.org/opendesk-edu/opendesk-cop)

---

## Agenda

| Zeit | Dauer | Punkt |
|------|-------|-------|
| 14:00 | 5′ | **1. Begrüßung & Einführung** |
| 14:05 | 25′ | **2. Betrieb & Lessons Learned** — ILIAS, OIDC, Backup, Monitoring, HRZ-Issues |
| 14:30 | 15′ | **3. Upstream & Cluster-Status** — openDesk 1.16.0, Komponenten, Cluster |
| 14:45 | 15′ | **4. Bildungssektor** — edu-Dienste, Roadmap, neue Anforderungen |
| 15:00 | 15′ | **5. Offener Austausch** — Freie Diskussion, Fragen |
| 15:15 | 5′ | **6. Wrap-Up & Action Items** — Nächstes Datum, To-Dos |
| 15:20 | ~10′ | *Puffer / Überlauf* |

---

## 1. Begrüßung & Einführung — 5′ (14:00–14:05)

- Vorstellungsrunde (wer ist neu?)
- Hinweis: BBB-Raum dauerhaft für dieses Format
- Kurz-Recap letzte CoP
- Agenda für heute

---

## 2. Betrieb & Lessons Learned — 25′ (14:05–14:30)

### a) ILIAS-Stabilisierung

**Problem**: MariaDB `Connection refused` bei neu erstellten Pods (ILIAS-Cronjobs).  
**Fix**: 5-facher Retry-Loop mit 10s Pause im Cronjob — läuft seitdem stabil.

**Offen**:
- Weitere Fehlerbilder im ILIAS-Betrieb?
- Neuere ILIAS-Version getestet?
- Upgrade-Pfad dokumentiert?

### b) OIDC / SSO

**Bisher eingerichtet**:
- Keycloak Realm `opendesk` als zentraler IdP
- `email` + `preferred_username` Mapper aktiv
- Verwaltung via `kcadm.sh` auf `ums-keycloak-0` (User: `kcadmin`)

**Fragen**:
- Nächste Dienste für OIDC? OpenProject? Nextcloud? Etherpad?
- Erfahrungen mit SAML vs OIDC?

### c) Backup-Infrastruktur (k8up)

| Aspekt | Status |
|--------|--------|
| Operator | k8up v2.13.0 |
| Ziel | s3://s3.hrz.uni-marburg.de/backups |
| Aktiv gesichert | 6 RWX-PVCs (clamav, dovecot, opencloud, seaweedfs, slidev) |
| Exkludiert | 29 RWO-PVCs (`k8up.io/exclude: true`) |

**Offen**: RWO-PVCs brauchen eigene Strategie (CSI-Snapshots? Per-Node-Schedules?).

**Production-Hardening (Vorschlag)**:
- **Tier A** (kritisch): Keycloak, PostgreSQL, Redis, MariaDB, MinIO → RPO 1h, RTO 2h, 30d Retention
- **Tier B** (wichtig): Nextcloud, OX, OpenProject, ILIAS, Moodle → RPO 1h, RTO 4h, 14d
- **Tier C** (experimentell): JupyterHub, Ollama, Dask → RPO 24h, RTO 1d, 7d

**Historisch**: Feb 2026 — 6 Backup-Pods stuck (10h), Fix: PodConfigRef entfernt.

### d) Monitoring

| Komponente | Metriken | Alerts | Dashboard |
|-----------|----------|--------|-----------|
| Collabora | ✅ | ✅ | ✅ |
| Nextcloud | ✅ | ❌ | ❌ |

**Lücken**: Backup-Health-Dashboards, Ressourcen-Alerts (CPU >80%, Memory >85%, Disk >80%), PrometheusRules teilweise deployed.

**Frage**: Welche Dashboards werden im Alltag vermisst?

### e) Known HRZ Issues

| Problem | Status | Workaround |
|---------|--------|------------|
| DNS CNAME-Ketten | CoreDNS → SERVFAIL | `hostAliases` in Deployments |
| Nextcloud AIO Probe-Bug | initialDelaySeconds falsch | in Chart patchen |
| Planka Ingress | hartcodiert `nginx` | Annotation entfernen |
| Grafana Ingress-Class | Default nginx | auf haproxy umstellen |
| ClamAV ICAP-Restartloop | 137 Rests, stale Socket | Container cleanup |
| k8up RWO-PVCs | Backup-Pod kann nicht mounten | Exkludiert |

### f) Diskussion — Was lief gut? Was nicht?

- Letzter Sprint — Highlights, Pain Points
- Prioritäten für den nächsten Sprint

---

## 3. Upstream & Cluster-Status — 15′ (14:30–14:45)

### openDesk 1.16.0 (nächste Version)

> **Hinweis**: v1.16.0 ist noch nicht released — letzter Tag ist v1.15.1 (08.06.2026). Die folgenden Punkte basieren auf aktuellen Entwicklungstrends und können sich bis zum Release ändern.

- **Nextcloud Worker-Tuning**: `nginx.workers` und `php.memoryLimit` konfigurierbar
  - ⚠️ `workers: "auto"` ist nicht cgroup-bewusst → spawnt pro Host-Core
- **Dovecot/Postfix LoadBalancerIP**: Fixe IP für externe Dienste
- **OX App Suite SSL/TLS**: DB-Verbindung konfigurierbar

### openDesk 1.15.0 (aktuelles Release, 28.05.2026)

- **SeaweedFS** als S3 Object Storage eingeführt
- **OX App Suite 8.48** — Proxy Protocol, Postfix-Verbesserungen, Quotas
- **Nextcloud 32.0.9** (PHP 8.4)
- **OpenProject 17.4.1** (CVE-Fix)
- **HAProxy Ingress Support** (relevant für HRZ — wir nutzen haproxy)

### Komponenten-Upstream

| Komponente | Stand (v1.15.1) | Hinweis |
|-----------|-----------------|---------|
| Element ft. Nordeck Widgets | 1.12.6 | |
| Nextcloud | 32.0.9 | Aktuell im Cluster |
| OX App Suite | 8.48 | |
| OpenProject | 17.4.1 | Aktuell im Cluster |
| Nubus | 1.19.1 | |
| XWiki | 17.10.7 | |
| Collabora | 25.04.9 | |
| Jitsi | 2.0.10590 | |
| PostgreSQL | 17 | Läuft stabil (Etherpad) |
| K3s | 1.32.3 | Im HRZ-Cluster |
| Containerd | 2.0.4-k3s2 | |

### Cluster-Plattform

- **HRZ-Cluster**: 9 Nodes (3 CP, 6 Worker), Debian 12, K3s v1.32.3
- **Storage**: Ceph CSI (RBD SSD + CephFS HDD EC)
- **Monitoring**: kube-prometheus-stack
- **GitOps**: ArgoCD

### Offene Runde

- Was beobachtet ihr in euren Projekten/Organisationen?
- Breaking Changes, auf die wir uns vorbereiten müssen?

---

## 4. Bildungssektor — 15′ (14:45–15:00)

### Aktuelle edu-Dienste im HRZ

| Dienst | Status | Anmerkung |
|--------|--------|-----------|
| **ILIAS** | Läuft | Stabilisierung läuft (s.o.) |
| **Moodle** | Helmchart bereit | Deployment ausstehend |
| **JupyterHub** | Im edu-Stack | |
| **SAML** | Generatoren vorhanden | in `opendesk-edu/scripts/` |

### Fragen an die Runde

- **Welche Dienste fehlen?** ILIAS, Moodle, JupyterHub — reicht das?
  - Stud.IP? HIS? Andere Hochschulsysteme?
- **Neue Anforderungen von Fachbereichen?**
  - Datenschutz & DSGVO — Cloud an Hochschulen
  - On-Premise vs. Cloud-Hosting
- **openDesk-Edu-Roadmap**: Wohin entwickelt sich das Projekt?
  - Nächste Releases und Meilensteine?
  - [opendesk-edu.org](https://opendesk-edu.org/) — Feedback?

### Repositories

- Codeberg: [codeberg.org/opendesk-edu/opendesk-edu](https://codeberg.org/opendesk-edu/opendesk-edu/)
- GitHub (Mirror): [github.com/opendesk-edu/opendesk-edu](https://github.com/opendesk-edu/opendesk-edu/)
- **CoP-Material**: [github.com/opendesk-edu/opendesk-cop](https://github.com/opendesk-edu/opendesk-cop)

---

## 5. Offener Austausch — 15′ (15:00–15:15)

- Freie Diskussion — eigene Themen/Probleme einbringen
- Hilfe bei spezifischen Problemen anbieten/nachfragen
- Wer möchte beim nächsten Mal etwas vorstellen?

**Mögliche Themen**:
- Erfahrungen mit dem Deployment (helmfile)
- Tipps & Tricks für den Betrieb
- Migration bestehender Systeme zu openDesk
- Integration eigener Dienste

---

## 6. Wrap-Up & Action Items — 5′ (15:15–15:20)

- **Wichtige Erkenntnisse** (kurz zusammenfassen)
- **Action Items**:
  - [ ] …
  - [ ] …
- **Nächstes CoP-Datum**: Bitte eintragen im DFN Terminplaner: <https://terminplaner6.dfn.de/de/p/1509a06af2198fc680b2cac353ecca55-1808219>
- **Feedback zur heutigen Session**

---

> *Puffer / Überlauf: bis 15:30*

---

## Anhang

### Abhängigkeitsgraph (openDesk Runtime Architektur)

> **Referenz**: Analyse des upstream [openDesk Deployment Repository](https://gitlab.opencode.de/bmi/opendesk/deployment/opendesk) (v1.15.1, 08.06.2026)
>
> Der Runtime-Graph zeigt IAM, Applications, Integrationen und Mail Flow (Service-to-Service).
> **Storage-Abhängigkeiten** wurden in eine separate Matrix ausgelagert → [`openDesk-storage-topology.md`](openDesk-storage-topology.md)
> **Zoomable**: SVG-Datei (`openDesk-dependency-graph.svg`) liegt im Repo und kann separat geöffnet werden.
>
> ▶️ **Graph-Datei**: [`openDesk-dependency-graph.mmd`](openDesk-dependency-graph.mmd) (Mermaid, editierbar)

### Storage Topology

> Ausgelagert in [`openDesk-storage-topology.md`](openDesk-storage-topology.md) — enthält:
> - Component × Storage Matrix (welche Komponente nutzt welchen Backend)
> - Backend-Detail (Technologie, Protokoll, Persistenz, Backup-Status)
> - Backup-Tier-Modell (Critical / Important / Experimental)
> - Cache & Ephemeral Data Übersicht

Kurzreferenz (Auszug aus der Storage-Topology):

| Storage | Genutzt von | Backup | Typ |
|---------|------------|--------|-----|
| **PostgreSQL** | Nextcloud, Keycloak, OpenProject, XWiki, Element | ✅ Tier A | Stateful |
| **MariaDB** | OX AppSuite | ✅ Tier A | Stateful |
| **S3/MinIO** | Nextcloud, OX, OpenProject, Nubus, Dovecot | ✅ Tier A | Stateful |
| **Redis** | Nextcloud, OX, Intercom | ❌ Cache | Stateless |
| **Memcached** | Nubus, OpenProject | ❌ Cache | Stateless |
| **PVC (RWX)** | Dovecot, ClamAV, CryptPad, NATS | ✅ k8up (6 PVCs) | Stateful |
| **PVC (RWO)** | OpenLDAP, Postfix, XWiki, OX Connector | ❌ 29 ausgeschlossen | Stateful |

### Fragen zur Vorbereitung

1. **ILIAS**: Konkrete Fehlerbilder? Neue Version getestet?
2. **OIDC**: Nächste Dienste zur Anbindung?
3. **Backup**: RWO-Strategie — CSI-Snapshots oder per-Node?
4. **Upstream**: Welche Entwicklungen sind relevant?
5. **Neue Gesichter**: Wer ist zum ersten Mal dabei?
6. **Hardening**: Wie weit ist Production-Hardening gediehen?

### Links & Ressourcen

- BBB-Raum: <https://webconf.hrz.uni-marburg.de/n/rooms/7gq-zdy-zje-roq/join>
- openDesk Edu: <https://opendesk-edu.org/>
- openDesk Edu Codeberg: <https://codeberg.org/opendesk-edu/opendesk-edu/>
- CoP-Repo GitHub: <https://github.com/opendesk-edu/opendesk-cop>
- CoP-Repo Codeberg: <https://codeberg.org/opendesk-edu/opendesk-cop>
- Upstream Deployment: <https://gitlab.opencode.de/bmi/opendesk/deployment/opendesk>
- Runtime Dependency Graph (Mermaid): [`openDesk-dependency-graph.mmd`](openDesk-dependency-graph.mmd)
- Runtime Dependency Graph (SVG): [`openDesk-dependency-graph.svg`](openDesk-dependency-graph.svg)
- Storage Topology & Backup Matrix: [`openDesk-storage-topology.md`](openDesk-storage-topology.md)
