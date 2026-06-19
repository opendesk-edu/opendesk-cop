## Goal
- Vorbereitung der openDesk Community of Practice Session (19. Juni 2026, 14:00 CET) mit detailliertem Session-Markdown
- Veröffentlichung als öffentliches Git-Repo auf GitHub und Codeberg für PR-Workflow
- Analyse des upstream openDesk Deployment Repository (GitLab) als Vorbereitung für die Session

## Constraints & Preferences
- Repo-Name: `opendesk-edu/cop`
- Public Repo, Branch `main`
- Kein lokales Setup — nur openDesk-Status, Architektur, Betriebsthemen
- Session-Fokus: ILIAS, OIDC, Backup, Monitoring, Upstream, Bildung

## Progress
### Done
- **Session-Markdown erstellt und befüllt** (`2025-06-19-community-of-practice-session.md`):
  - Agenda + Inhalte aus Projektdokumenten (ILIAS Retry-Loop, OIDC-Client-Registrierung, k8up Tier-Modell, Monitoring-Lücken, openDesk 1.16.0/1.15.0 Features, edu-Dienste, Known HRZ Issues)
  - Commit c8c9425, gepusht auf beide Remotes
- **GitHub-Repo erstellt**: https://github.com/opendesk-edu/cop
- **Codeberg-Repo erstellt**: https://codeberg.org/opendesk-edu/cop
- **Beide Remotes konfiguriert**, `master` → `main` umbenannt, alter Branch gelöscht
- **AGENTS.md-Cop-Eintrag entfernt** (falscher Ort)

### Latest: Upstream-Analyse
- **openDesk Deployment Repository analysiert** (`https://gitlab.opencode.de/bmi/opendesk/deployment/opendesk`):
  - Aktuellstes Release: **v1.15.1** (2026-06-08) — OpenProject 17.4.1 CVE-Fix
  - Letztes Major-Release: **v1.15.0** (28.05.2026) — SeaweedFS, OX 8.48, Proxy Protocol, Per-User-Quotas
  - Keine Commits seit 08.06.2026 — vermutlich Arbeit an v1.16.0
  - Relevante Trends: HAProxy Ingress Support (v1.13.0), Nextcloud 32, OX 8.46→8.48, Nubus Multi-IdP SSO
  - Komponenten-Stand dokumentiert (Element 1.12.6, Nextcloud 32.0.9, OX 8.48, OpenProject 17.4.1, etc.)

## Key Decisions
- Repo `opendesk-edu/cop` statt persönlichem Namespace
- Codeberg-Login via `graphwiz-ai`, GitHub via `tobias-weiss-ai-xr`
- Default-Branch `main`
- Upstream-Repository als primäre Referenz für Session-Inhalte genutzt

## Next Steps
- **Session durchführen** (19.06.2026, 14:00 CET, BBB)
- Nach Session: Notizen/Ergebnisse committen und pushen
- Upstream-Entwicklung beobachten (v1.16.0?)
- Einladung zur Mitarbeit/PRs an CoP-Repo verbreiten

## Critical Context
- BBB-Raum: https://webconf.hrz.uni-marburg.de/n/rooms/7gq-zdy-zje-roq/join
- openDesk Edu: https://opendesk-edu.org/
- CoP-Repo GitHub: https://github.com/opendesk-edu/cop
- CoP-Repo Codeberg: https://codeberg.org/opendesk-edu/cop
- Upstream openDesk: https://gitlab.opencode.de/bmi/opendesk/deployment/opendesk
- Cluster: 9 Nodes, K3s v1.32.3, Ceph CSI
- Aktuellstes Release upstream: v1.15.1 (08.06.2026)
- Session-Dauer: ca. 1–1,5h

## Relevant Files
- `/home/weissto_local/git/opendesk_git/cop/2025-06-19-community-of-practice-session.md`: Session-Agenda mit ausgefülltem Inhalt
- `/home/weissto_local/git/opendesk_git/AGENTS.md`: Projekt-Wissensbasis
