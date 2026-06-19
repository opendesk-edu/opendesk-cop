# openDesk Community of Practice — 19. Juni 2026

- **Datum**: Freitag, 19. Juni 2026
- **Uhrzeit**: 14:00 – 15:30 Uhr CET
- **Ort**: BigBlueButton — https://webconf.hrz.uni-marburg.de/n/rooms/7gq-zdy-zje-roq/join
- **Dauer**: ca. 1–1,5 Stunden

---

## 1. Begrüßung & Einführung — 5 Min (14:00–14:05)

- Vorstellungsrunde (wer ist neu?)
- Hinweis: BBB-Raum ist jetzt ein eigener, dauerhafter Raum für dieses Format
- Kurz recap: letzte CoP (wann? was waren die Key-Takeaways?)
- Agenda für heute

---

## 2. Erfahrungsberichte & Lessons Learned — 30 Min (14:05–14:35)

### Aktueller Sprint: Stabilisierung

**a) ILIAS-Stabilisierung**
- Status der ILIAS-Integration im edu-Stack
- Welche Stabilitätsprobleme wurden adressiert?
  - MariaDB transient \"Connection refused\" — Retry-Loop in Cronjobs (5 Versuche, 10s Pause)
  - Weitere ILIAS-spezifische Themen aus dem Betrieb
- Offene Punkte / Known Issues

**b) OIDC / SSO**
- OIDC-Client-Registrierung: SOGo und Planka sind registriert (Keycloak Realm `opendesk`)
  - SOGo: Client-ID `sogo`, Secret in `sogo-sogo` Secret
  - Planka: Client-ID `planka`, Secret in `planka-planka-secrets`
- Mapper für `email` und `preferred_username` eingerichtet
- Nächste Dienste für OIDC-Anbindung?
- Erfahrungen mit Keycloak-Admin (`kcadm.sh` auf `ums-keycloak-0`)

**c) Backup-Infrastruktur (k8up)**
- Status: 29 RWO-PVCs mit `k8up.io/exclude: "true"` annotiert (Sprint 6)
- Nur RWX-PVCs werden im Haupt-Schedule gesichert:
  - `clamav-db`, `clamav-tmp`, `dovecot`, `opendesk-opencloud-data`,
    `seaweedfs-all-in-one-data`, `slidev-slides`
- Bekanntes Problem: RWO-PVCs brauchen separate Strategie (CSI-Snapshots oder per-Node-Schedules)
- Backup-Ziel: `s3:https://s3.hrz.uni-marburg.de/backups`
- Diskussion: Reichen die aktuellen Backups? Was fehlt?

**d) Monitoring**
- Prometheus + Grafana im Cluster
- Neue Dashboards? (Verweis auf `monitoring/` im Repo)
- Bekannte Monitoring-Lücken?

### Diskussion — 10 Min

- Was lief gut? Was nicht?
- Welche Probleme sind im laufenden Betrieb aufgetreten?
- Gibt es Pain Points, die im nächsten Sprint adressiert werden sollten?

---

## 3. Aktuelle Upstream-Entwicklungen — 15 Min (14:35–14:50)

- **K3s**: Version 1.32.3 im Cluster — relevante Änderungen?
- **Helm / Helmfile**: Neue Features oder Breaking Changes?
- **openDesk-Upstream**: Was tut sich bei den openDesk-Kernkomponenten?
  - openCloud / Infinite Scale?
  - OX App Suite?
  - Nextcloud?
- **Andere relevante Projekte**:
  - Keycloak (neue Versionen?)
  - PostgreSQL 17 (im Einsatz für Etherpad)
  - k8up-Upstream (wir forken/maintainen selbst)

*Offene Runde: Was beobachtet ihr in euren Projekten?*

---

## 4. Themen & Schwerpunkte für den Bildungssektor — 15 Min (14:50–15:05)

- **Aktuelle Anforderungen aus HRZ-Perspektive**:
  - ILIAS, Moodle, JupyterHub — Betriebserfahrungen
  - Saml-Authentifizierung (SAML-Generatoren in `opendesk-edu/scripts/`)
- **Themen aus der Runde**:
  - Welche Dienste sind im edu-Kontext besonders gefragt?
  - Gibt es neue Anforderungen von Fachbereichen / Einrichtungen?
  - Datenschutz & DSGVO — Handhabung bei Cloud-Diensten?
- **openDesk-Edu-Roadmap**: Wohin entwickelt sich das Projekt?
  - Website: https://opendesk-edu.org/
  - Codeberg: https://codeberg.org/opendesk-edu/opendesk-edu/
  - GitHub (Mirror): https://github.com/opendesk-edu/opendesk-edu/

---

## 5. Offener Austausch & Fragen — 15–20 Min (15:05–15:25)

- Freie Diskussion
- Eigene Themen einbringen
- Hilfe bei spezifischen Problemen anbieten/nachfragen
- Wer möchte beim nächsten Mal etwas vorstellen?

---

## 6. Wrap-Up — 5 Min (15:25–15:30)

- **Wichtige Erkenntnisse** (kurz zusammenfassen)
- **Action Items / To-Dos**:
  - [ ] …
  - [ ] …
- **Nächstes CoP-Date**: Wann? Wer übernimmt Moderation?
- **Feedback zur heutigen Session**

---

## Notizen / Vorbereitung

### Fragen zur Vorbereitung:
1. **ILIAS**: Gibt es konkrete Fehlerbilder, die ich ansprechen soll? Neue Version getestet?
2. **Backup**: RWO-Backup-Strategie — gibt es hier Neuigkeiten?
3. **Upstream**: Welche konkreten Upstream-Entwicklungen sind gerade relevant?
4. **Neue Gesichter**: Wer ist zum ersten Mal dabei? (Vorstellungsrunde anpassen)

### Links & Ressourcen:
- BBB-Raum: https://webconf.hrz.uni-marburg.de/n/rooms/7gq-zdy-zje-roq/join
- openDesk Edu: https://opendesk-edu.org/
- openDesk Edu Codeberg: https://codeberg.org/opendesk-edu/opendesk-edu/
- openDesk Edu GitHub: https://github.com/opendesk-edu/opendesk-edu/
- Cluster-Info: 9 Nodes, K3s v1.32.3, Ceph CSI (RBD SSD + CephFS HDD EC)

### Timing-Puffer:
- Gesamt: 90 Min geblockt
- Kernthemen: ~60 Min
- Puffer: ~30 Min (Diskussion kann fließen)
- Bei weniger Teilnehmern: kompakter, früherer Abschluss möglich
