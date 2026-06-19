# openDesk Community of Practice — 19. Juni 2026

- **Datum**: Freitag, 19. Juni 2026
- **Uhrzeit**: 14:00 – 15:30 Uhr CET
- **Ort**: BigBlueButton — https://webconf.hrz.uni-marburg.de/n/rooms/7gq-zdy-zje-roq/join
- **Dauer**: ca. 1–1,5 Stunden

---

## 1. Begrüßung & Einführung — 5 Min (14:00–14:05)

- Vorstellungsrunde (wer ist neu?)
- Hinweis: BBB-Raum ist jetzt ein eigener, dauerhafter Raum für dieses Format
- Kurz-Recap letzte CoP
- Agenda für heute

---

## 2. Erfahrungsberichte & Lessons Learned — 30 Min (14:05–14:35)

### a) ILIAS-Stabilisierung

**Status**: ILIAS ist im edu-Stack integriert, aber es gab Stabilitätsprobleme

**Adressierte Probleme**:
- **MariaDB transient "Connection refused"**: Neu erstellte Pods (v.a. ILIAS-Cronjobs) bekamen sporadisch `SQLSTATE[HY000] [2002] Connection refused`.
  - **Fix**: 5-facher Retry-Loop mit 10s Pause im Cronjob implementiert
- **ILIAS-Cron läuft stabil** seit dem Retry-Mechanismus

**Offene Punkte**:
- Gibt es weitere Fehlerbilder im ILIAS-Betrieb?
- Wurde eine neuere ILIAS-Version getestet?
- ILIAS-Upgrade-Pfad dokumentiert?

### b) OIDC / SSO

**Aktueller Stand**:
- **Keycloak Realm `opendesk`** als zentraler Identity Provider
- **SOGo OIDC-Client** registriert: Client-ID `sogo`, Secret in `sogo-sogo` K8s-Secret
- **Planka OIDC-Client** registriert: Client-ID `planka`, Secret in `planka-planka-secrets`
- **Mapper**: `email` und `preferred_username` Claims eingerichtet

**Verwaltung**:
- Admin-Zugriff via `kcadm.sh` auf `ums-keycloak-0`
- User: `kcadmin`

**Diskussion**:
- Welche Dienste sollen als nächstes per OIDC angebunden werden?
- OpenProject? Nextcloud? Etherpad?
- Erfahrungen mit Keycloak-Konfiguration (SAML vs OIDC)?

### c) Backup-Infrastruktur (k8up)

**Aktueller Status**:
- **k8up-Operator** (v2.13.0) läuft im Cluster
- **Backup-Ziel**: `s3:https://s3.hrz.uni-marburg.de/backups`
- **Sprint-6-Fix**: 29 RWO-PVCs mit `k8up.io/exclude: "true"` annotiert — nur RWX-PVCs werden gesichert

**Gesicherte RWX-PVCs**:
| PVC | Service |
|-----|---------|
| `clamav-db` | ClamAV |
| `clamav-tmp` | ClamAV |
| `dovecot` | Dovecot |
| `opendesk-opencloud-data` | openCloud |
| `seaweedfs-all-in-one-data` | SeaweedFS |
| `slidev-slides` | Slidev |

**Bekanntes Problem**: RWO-PVCs brauchen separate Backup-Strategie:
- CSI-Snapshots (Ceph CSI unterstützt das)
- Per-Node-Schedules
- **Noch nicht adressiert** — Diskussionsbedarf

**Produktion-Hardening (Juni 2026)**:
- **Tier-Modell** vorgeschlagen:
  - **Tier A** (Kritisch): Keycloak, PostgreSQL, Redis, MariaDB, MinIO → RPO 1h, RTO 2h, 30d Retention
  - **Tier B** (Wichtig): Nextcloud, OX, OpenProject, ILIAS, Moodle → RPO 1h, RTO 4h, 14d Retention
  - **Tier C** (Experimentell): JupyterHub, Ollama, Dask → RPO 24h, RTO 1d, 7d Retention
- Backup-Verifikation: Monatlicher Restore-Test, wöchentliche Checksummen

**Historische Incidents**:
- Feb 2026: 6 Backup-Pods in `ContainerCreating` (10h) — unvollständiger PodConfig
- Fix: PodConfigRef entfernt, Stuck-Pods gelöscht
- Seitdem: Läuft, aber RWO-Problem ungelöst

### d) Monitoring

**Aktueller Stand**:
- **Prometheus + Grafana** (kube-prometheus-stack) im Cluster
- **Node-Exporter** auf allen Nodes
- **openDesk-Dashboards** via ConfigMap (wenn aktiviert)

**Verfügbare Dashboards** (laut Chart `opendesk-dashboards`):
- Collabora: ✅ Metriken, ✅ Alerts, ✅ Dashboard
- Nextcloud: ✅ Metriken, ❌ Alerts, ❌ Dashboard

**Bekannte Lücken**:
- Fehlende Alerts für viele Dienste
- Keine Backup-Health-Dashboards (failure, stuck, storage)
- Ressourcen-Alerts fehlen (CPU >80%, Memory >85%, Disk >80%)
- PrometheusRules für openDesk-spezifische Alerts sind teilweise deployed

**Diskussion**:
- Welche Dashboards werden im Alltag vermisst?
- Wer hat Erfahrung mit Prometheus-Alerting?

### e) Known HRZ Issues (laufend)

| Problem | Status | Workaround |
|---------|--------|------------|
| **DNS CNAME-Ketten** schlagen fehl | CoreDNS → SERVFAIL | `hostAliases` in Deployments |
| **Nextcloud AIO Probe-Bug** | Readiness/Startup-Probe falsch konfiguriert | `initialDelaySeconds` patchen |
| **Planka Ingress-Annotation** | `kubernetes.io/ingress.class: nginx` hartcodiert | Annotation entfernen für HAProxy |
| **Grafana Ingress-Class** | Default nginx statt haproxy | Auf haproxy umstellen |
| **ClamAV ICAP-Restartloop** | 137 Restarts, stale Socket/PID | Container cleanup |
| **k8up RWO-PVCs** | Backup-Pod kann nicht alle mounten | RWO-PVCs exkludiert |

### Diskussion — 10 Min

- Was lief gut im letzten Sprint?
- Welche neuen Pain Points sind aufgetreten?
- Was sollte im nächsten Sprint priorisiert werden?

---

## 3. Aktuelle Upstream-Entwicklungen — 15 Min (14:35–14:50)

### Cluster-Plattform
- **K3s v1.32.3** im HRZ-Cluster — Containerd 2.0.4
- **9 Nodes** (3 Control-Plane, 6 Worker) — Debian 12
- **Ceph CSI** (RBD SSD + CephFS HDD EC)

### openDesk 1.16.0 (aktuelle Version)
- **Nextcloud Worker-Tuning**: `nginx.workers` und `php.memoryLimit` jetzt konfigurierbar
  - Wichtig: `workers: "auto"` ist nicht cgroup-bewusst → spawnt pro Host-Core
- **Dovecot/Postfix LoadBalancerIP**: Fixe IP für externe Dienste konfigurierbar
- **OX App Suite SSL/TLS**: DB-Verbindung jetzt mit SSL/TLS konfigurierbar

### openDesk 1.15.0
- **External-Sharing-Quotas**: Per-User-Quota für Share-Links und Gäste
- **Postfix Proxy Protocol**: HAProxy-Unterstützung für echte Client-IPs hinter LB
- **OX Context-Quotas**: Limits für Tasks, Contacts, Attachments

### Komponenten-Upstream
- **openCloud / Infinite Scale**: Status? Läuft im edu-Stack?
- **OX App Suite**: Neue Versionen? Breaking Changes?
- **Nextcloud**: AIO-Integration — läuft stabil?
- **Keycloak**: Neue Versionen? Update-Pfad?
- **PostgreSQL 17**: Im Einsatz für Etherpad — läuft stabil
- **k8up-Upstream**: Wir forken/maintainen selbst — was ist der Stand?

### Offene Runde
- Was beobachtet ihr in euren Projekten/Organisationen?
- Gibt es Breaking Changes, auf die wir uns vorbereiten müssen?

---

## 4. Themen & Schwerpunkte für den Bildungssektor — 15 Min (14:50–15:05)

### Aktuelle edu-Dienste im HRZ
- **ILIAS**: Läuft im edu-Stack — Stabilisierung läuft
- **Moodle**: Helmchart bereit, Deployment noch ausstehend (Task #5)
- **JupyterHub**: Im edu-Stack enthalten
- **SAML-Authentifizierung**: SAML-Generatoren in `opendesk-edu/scripts/`

### Fragen an die Runde
- **Welche Dienste sind im edu-Kontext besonders gefragt?**
  - ILIAS, Moodle, JupyterHub — reicht das?
  - Fehlt was? (z.B. Stud.IP, HIS, etc.)
- **Neue Anforderungen von Fachbereichen?**
  - Datenschutz & DSGVO — Cloud-Dienste an Hochschulen
  - On-Premise vs. Cloud-Hosting
- **openDesk-Edu-Roadmap**:
  - Wohin entwickelt sich das Projekt?
  - Nächste Releases und Meilensteine?
  - openDesk-Edu-Website (https://opendesk-edu.org/) — Feedback?

### Repositories
- Codeberg: https://codeberg.org/opendesk-edu/opendesk-edu/
- GitHub (Mirror): https://github.com/opendesk-edu/opendesk-edu/
- **NEU**: CoP-Session-Material: https://github.com/opendesk-edu/cop

---

## 5. Offener Austausch & Fragen — 15–20 Min (15:05–15:25)

- **Freie Diskussion**
- **Eigene Themen/Probleme einbringen**
- **Hilfe bei spezifischen Problemen anbieten/nachfragen**
- **Wer möchte beim nächsten Mal etwas vorstellen?**

### Mögliche Themen aus der Community:
- Erfahrungen mit dem deployment (helmfile)
- Tipps & Tricks für den Betrieb
- Migration von bestehenden Systemen zu openDesk
- Integration eigener Dienste

---

## 6. Wrap-Up — 5 Min (15:25–15:30)

- **Wichtige Erkenntnisse** (kurz zusammenfassen)
- **Action Items / To-Dos**:
  - [ ] …
  - [ ] …
- **Nächstes CoP-Datum**: Wann? Wer übernimmt Moderation?
- **Feedback zur heutigen Session**

---

## Anhang: Notizen & Vorbereitung

### Fragen zur Vorbereitung
1. **ILIAS**: Gibt es konkrete Fehlerbilder? Neue Version getestet?
2. **OIDC**: Nächste Dienste zur Anbindung?
3. **Backup**: RWO-Strategie — CSI-Snapshots oder per-Node?
4. **Upstream**: Welche konkreten Entwicklungen sind relevant?
5. **Neue Gesichter**: Wer ist zum ersten Mal dabei?
6. **Hardening**: Wie weit ist das Production-Hardening gediehen?

### Links & Ressourcen
- BBB-Raum: https://webconf.hrz.uni-marburg.de/n/rooms/7gq-zdy-zje-roq/join
- openDesk Edu: https://opendesk-edu.org/
- openDesk Edu Codeberg: https://codeberg.org/opendesk-edu/opendesk-edu/
- CoP-Repo GitHub: https://github.com/opendesk-edu/cop
- CoP-Repo Codeberg: https://codeberg.org/opendesk-edu/cop
- Cluster: 9 Nodes, K3s v1.32.3, Ceph CSI (RBD SSD + CephFS HDD EC)

### Timing-Puffer
- Gesamt: 90 Min geblockt
- Kernthemen: ~60 Min
- Puffer: ~30 Min (Diskussion kann fließen)
- Bei weniger Teilnehmern: kompakter, früherer Abschluss möglich
