# GmbH KI-Buchhalter

KI-gestützte Buchhaltungssoftware für deutsche GmbHs als DATEV-Alternative.

## Projektübersicht

Dieses Projekt entwickelt eine moderne, KI-gestützte Buchhaltungslösung speziell für deutsche GmbHs (vermögensverwaltend und operativ). Claude fungiert als AI-Buchhalter über MCP Tools und API Endpoints.

## Research Reports

Die Forschungsphase umfasst vier Fachberichte und eine Gesamtsynthese:

| Report | Beschreibung |
|--------|-------------|
| [Rechtliche Anforderungen](research/recht/rechtliche-anforderungen.md) | GoBD, HGB, DSGVO, EU AI Act, IDW PS 880 |
| [Steuerliche Anforderungen](research/steuern/steuerliche-anforderungen.md) | KStG, GewStG, UStG, EStG, Steuerberater-Schnittstellen |
| [Technische Architektur](research/technik/technische-architektur.md) | Tech Stack, API Design, Datenbank, DevOps, Security |
| [KI-Integration](research/ki-integration/ki-integration.md) | Claude Agent SDK, MCP Server, Prompt Engineering, HITL |
| [Gesamtreport](research/GESAMTREPORT.md) | Synthese aller Reports mit Roadmap und Risikoanalyse |

## Tech Stack

- **Backend:** Python 3.13+ / FastAPI 0.128+ / Pydantic 2.12+
- **Datenbank:** PostgreSQL 17+ / SQLAlchemy 2.1 / Alembic
- **Frontend:** Next.js 16 / React 19 / TypeScript
- **KI:** Claude (Anthropic SDK) / Claude Agent SDK / MCP Server
- **Hosting:** Hetzner (Backend, DE Datenresidenz) + Vercel (Frontend)

## Status

Aktuell in der Research- und Planungsphase. Die Reports dienen als Grundlage für die Prototyp-Entwicklung.

## Lizenz

MIT
