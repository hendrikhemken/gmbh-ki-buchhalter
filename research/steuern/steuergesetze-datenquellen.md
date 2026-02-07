# Deutsche Steuergesetze - Datenquellen & Online-Verfügbarkeit

> **Version:** 1.0
> **Datum:** 07.02.2026
> **Zweck:** Übersicht aller verfügbaren Quellen für deutsche Steuergesetze in maschinenlesbaren Formaten als Grundlage für die KI-Buchhaltungssoftware

---

## 1. Übersicht

Alle relevanten deutschen Steuergesetze für die Unternehmensbesteuerung sind in maschinenlesbaren Formaten frei verfügbar. Es existieren sowohl offizielle Regierungsquellen als auch Community-gepflegte GitHub-Repositories in Markdown, JSON und XML. Die Daten sind kostenlos nutzbar und werden teils tagesaktuell aktualisiert.

---

## 2. Relevante Steuergesetze für die Unternehmensbesteuerung

### 2.1 Kerngesetze

| Gesetz | Kürzel | Slug (gesetze-im-internet.de) | Relevanz |
|--------|--------|-------------------------------|----------|
| Abgabenordnung | AO | `ao_1977` | Verfahrensrecht, Pflichten, Fristen |
| Einkommensteuergesetz | EStG | `estg` | Personengesellschaften, Einzelunternehmer |
| Körperschaftsteuergesetz | KStG | `kstg_1977` | GmbH-Besteuerung (Kerngesetz) |
| Gewerbesteuergesetz | GewStG | `gewstg` | Gewerbesteuer für alle Gewerbebetriebe |
| Umsatzsteuergesetz | UStG | `ustg_1980` | Mehrwertsteuer / Vorsteuer |
| Handelsgesetzbuch | HGB | `hgb` | Rechnungslegung, Buchführungspflichten |
| Bewertungsgesetz | BewG | `bewg` | Bewertung von Vermögensgegenständen |

### 2.2 Ergänzende Steuergesetze

| Gesetz | Kürzel | Slug | Relevanz |
|--------|--------|------|----------|
| Umwandlungssteuergesetz | UmwStG | `umwstg` | Umstrukturierungen, Verschmelzungen |
| Außensteuergesetz | AStG | `astg` | Auslandsbeziehungen, Verrechnungspreise |
| Erbschaftsteuergesetz | ErbStG | `erbstg_1974` | Schenkungen, Unternehmensübertragung |
| Grunderwerbsteuergesetz | GrEStG | `grestg_1983` | Immobilientransaktionen |
| Investmentsteuergesetz | InvStG | `invstg_2018` | Investmentfonds (vvGmbH) |
| Solidaritätszuschlaggesetz | SolZG | `solzg_1995` | Zuschlag auf KSt/ESt |

### 2.3 Wichtige Durchführungsverordnungen

| Verordnung | Kürzel | Relevanz |
|------------|--------|----------|
| Einkommensteuer-Durchführungsverordnung | EStDV | Details zu EStG |
| Körperschaftsteuer-Durchführungsverordnung | KStDV | Details zu KStG |
| Umsatzsteuer-Durchführungsverordnung | UStDV | Details zu UStG |
| Gewerbesteuer-Durchführungsverordnung | GewStDV | Details zu GewStG |
| Buchführungs- und Aufzeichnungspflichten (GoBD) | GoBD | BMF-Schreiben, kein Gesetz |

---

## 3. GitHub-Repositories

### 3.1 bundestag/gesetze (Markdown) — Empfohlen als Referenz

- **URL:** https://github.com/bundestag/gesetze
- **Format:** Markdown
- **Inhalt:** Alle Bundesgesetze und -verordnungen
- **Struktur:** Alphabetisch sortierte Verzeichnisse (a-z), 531+ Commits
- **Stars:** 1.900+ | **Forks:** 162+
- **Updates:** Manuell nach Bundesgesetzblatt-Veröffentlichungen
- **Erstellt:** 2012 (langfristig gepflegt)
- **Konvertierung:** XML → Markdown via [bundestag/gesetze-tools](https://github.com/bundestag/gesetze-tools)

**Stärken:**
- Git-versioniert → Änderungen per `git diff` nachvollziehbar
- Markdown → human-readable, einfach in UI darstellbar
- Große Community, etabliertes Projekt

**Schwächen:**
- Updates nicht tagesaktuell
- Keine strukturierten Metadaten (nur Fließtext)

### 3.2 kmein/gesetze (Markdown, tagesaktuell) — Empfohlen für Aktualität

- **URL:** https://github.com/kmein/gesetze
- **Format:** Markdown / Plain Text
- **Inhalt:** Alle Bundesgesetze
- **Updates:** **Tagesaktuell** (automatisiert)
- **Git-Historie:** Seit April 2021
- **Unique Feature:** Jeder Commit = eine Gesetzesänderung → perfekte Audit-Trail

**Stärken:**
- Aktuellste Quelle unter allen Repositories
- Automatisierte tägliche Updates
- Git-History zeigt exakt, wann sich welches Gesetz geändert hat

**Schwächen:**
- Weniger Community als bundestag/gesetze
- Flaches Text-Format ohne strukturierte Metadaten

### 3.3 QuantLaw/gesetze-im-internet (XML-Archiv) — Empfohlen für Compliance

- **URL:** https://github.com/QuantLaw/gesetze-im-internet
- **Format:** XML (offizielles Format von gesetze-im-internet.de)
- **Updates:** Tägliches Archiv seit Mai 2020
- **Historische Daten:** Wöchentliche Snapshots bis zurück Juni 2019
- **Zugang:** Separater `data`-Branch + datierte Releases

**Stärken:**
- Offizielles XML-Format → rechtlich am nächsten an der Quelle
- Timestamped Releases für Reproduzierbarkeit
- Ideal für Compliance-Dokumentation

**Schwächen:**
- Nur XML (erfordert Parsing)
- Großes Repository (viele historische Snapshots)

### 3.4 maxsagt/de_laws_to_json (JSON) — Empfohlen für LLM-Integration

- **URL:** https://github.com/maxsagt/de_laws_to_json
- **Format:** JSON (strukturiert, tokenisiert)
- **Inhalt:** 6.000+ Bundesgesetze
- **Features:**
  - Download aller Gesetze vom XML-Index
  - Transformation zu JSON mit Metadaten
  - Tokenisierung via tiktoken (Token-Counts pro Paragraph)
  - Konsolidierung in eine JSON-Datei

**JSON-Struktur pro Gesetz:**
```json
{
  "abbreviation": "KStG",
  "title": "Körperschaftsteuergesetz",
  "norms": [
    {
      "paragraph": "§ 1",
      "title": "Unbeschränkte Steuerpflicht",
      "text": "...",
      "token_count": 245
    }
  ]
}
```

**Stärken:**
- Perfekt für RAG / Retrieval Augmented Generation
- Token-Counts bereits berechnet → Context-Window-Planung
- Paragraph-Level-Granularität für präzise Zitationen
- Direkt in Vector-DBs (Pinecone, Weaviate, pgvector) ladbar

**Schwächen:**
- Nicht tagesaktuell (Point-in-Time Snapshot)
- Muss manuell neu generiert werden bei Gesetzesänderungen

### 3.5 nfelger/gesetze-aus-dem-internet (JSON + PostgreSQL)

- **URL:** https://github.com/nfelger/gesetze-aus-dem-internet
- **Format:** JSON + PostgreSQL-Backend
- **Features:** Automatisierte Download-Pipeline, inkrementelle Updates
- **Web-Interface:** https://gadi.netlify.app
- **Tech:** Python-basiert mit invoke Tasks
- **Status:** 179 Commits, letzte Updates Dezember 2023

### 3.6 ayunis-core/ayunis-legal-mcp (MCP Server)

- **URL:** https://github.com/ayunis-core/ayunis-legal-mcp
- **Format:** MCP Server für deutsche Gesetze
- **Relevanz:** Direkt nutzbar in unserer MCP-Architektur
- **Zu prüfen:** Umfang, Aktualität, Kompatibilität mit MCP SDK v2

---

## 4. Offizielle Quellen

### 4.1 gesetze-im-internet.de (Primärquelle)

- **URL:** https://www.gesetze-im-internet.de
- **Betreiber:** BMJ (Bundesministerium der Justiz) via juris GmbH
- **Lizenz:** Kostenlos, frei nutzbar und weiterverwendbar

**Verfügbare Formate:**
| Format | Zugang |
|--------|--------|
| HTML | Direkt im Browser |
| PDF | `https://www.gesetze-im-internet.de/[slug]/[Kürzel].pdf` |
| XML | `https://www.gesetze-im-internet.de/[slug]/xml.zip` |
| EPUB | `https://www.gesetze-im-internet.de/[slug]/[Kürzel].epub` |

**XML-Zugang:**

Index aller Gesetze (täglich aktualisiert):
```
https://www.gesetze-im-internet.de/gii-toc.xml
```

Einzelne Gesetze als XML-ZIP:
```
https://www.gesetze-im-internet.de/estg/xml.zip        # EStG
https://www.gesetze-im-internet.de/kstg_1977/xml.zip   # KStG
https://www.gesetze-im-internet.de/ustg_1980/xml.zip   # UStG
https://www.gesetze-im-internet.de/gewstg/xml.zip      # GewStG
https://www.gesetze-im-internet.de/ao_1977/xml.zip     # AO
https://www.gesetze-im-internet.de/hgb/xml.zip         # HGB
https://www.gesetze-im-internet.de/bewg/xml.zip        # BewG
https://www.gesetze-im-internet.de/umwstg/xml.zip      # UmwStG
https://www.gesetze-im-internet.de/astg/xml.zip        # AStG
```

**XML-Schema (DTD):**
```
https://www.gesetze-im-internet.de/dtd/1.01/gii-norm.dtd
```

**XML-Struktur:**
```xml
<dokumente>
  <norm>
    <metadaten>
      <jurabk>KStG</jurabk>
      <ausfertigung-datum>1976-08-31</ausfertigung-datum>
      <fundstelle>...</fundstelle>
    </metadaten>
    <textdaten>
      <Content>...</Content>
    </textdaten>
  </norm>
</dokumente>
```

### 4.2 NeuRIS — Neues Rechtsinformationssystem (Zukunft)

- **URL:** https://testphase.rechtsinformationen.bund.de/
- **API-Docs:** https://docs.rechtsinformationen.bund.de/
- **Status:** Testphase (Stand April 2025)
- **Betreiber:** DigitalService (Bund)
- **Formate:** REST API mit XML, HTML und JSON Output
- **Abdeckung:** Bundesgesetze, Verordnungen, Gerichtsentscheidungen

**Einschätzung:** Vielversprechend als offizielle API-Lösung, aber noch nicht produktionsreif. Datenbestand noch unvollständig, Bulk-Downloads noch nicht verfügbar. Sollte beobachtet werden — sobald stabil, wäre dies die bevorzugte offizielle Datenquelle.

### 4.3 Weitere Quellen

| Quelle | URL | Status | Inhalt |
|--------|-----|--------|--------|
| Open Legal Data | https://de.openlegaldata.io/ | Aktiv | 57.000+ Texte, REST API, SDKs |
| OffeneGesetze.de | https://offenegesetze.de | **Eingestellt** (→ recht.bund.de seit 01/2023) | Bundesgesetzblatt-Archive |
| recht.bund.de | https://www.recht.bund.de | Aktiv | Offizieller Nachfolger von OffeneGesetze |
| GovData | https://www.govdata.de | Aktiv | Open-Data-Portal mit Gesetzesverweisen |

---

## 5. Empfehlung für die KI-Buchhaltungssoftware

### 5.1 Daten-Pipeline (empfohlener Ansatz)

```
┌─────────────────────────────────┐
│   gesetze-im-internet.de       │
│   (XML via gii-toc.xml Index)  │
└──────────────┬──────────────────┘
               │ Täglicher Download (Cron)
               ▼
┌─────────────────────────────────┐
│   XML → JSON Transformation    │
│   (Vorlage: maxsagt/de_laws)   │
└──────────────┬──────────────────┘
               │
       ┌───────┴───────┐
       ▼               ▼
┌──────────────┐ ┌──────────────────┐
│  PostgreSQL  │ │  Vector-DB       │
│  (Volltexte, │ │  (Embeddings     │
│   Metadaten, │ │   pro Paragraph, │
│   Versionen) │ │   pgvector)      │
└──────┬───────┘ └────────┬─────────┘
       │                  │
       └────────┬─────────┘
                ▼
┌─────────────────────────────────┐
│   MCP Server / API Endpoints   │
│   → Claude als AI-Buchhalter   │
│   → Paragraph-genaue Zitation  │
└─────────────────────────────────┘
```

### 5.2 Priorisierung der Quellen

| Priorität | Quelle | Verwendung |
|-----------|--------|------------|
| **P0** | gesetze-im-internet.de (XML) | Primäre Datenquelle, offiziell |
| **P1** | kmein/gesetze (Git) | Change-Detection: täglicher Diff zeigt Gesetzesänderungen |
| **P2** | maxsagt/de_laws_to_json | Vorlage für JSON-Transformation + Tokenisierung |
| **P3** | ayunis-legal-mcp | Referenz-Implementierung für MCP-Integration |
| **Watch** | NeuRIS API | Migration sobald produktionsreif |

### 5.3 Implementierungsschritte

1. **Phase 1 — Prototyp:** Statischer Download der relevanten Steuergesetze (XML → JSON), Laden in PostgreSQL + pgvector
2. **Phase 2 — Automatisierung:** Täglicher Cron-Job: `gii-toc.xml` parsen, geänderte Gesetze erkennen, XML herunterladen, JSON transformieren, DB aktualisieren
3. **Phase 3 — MCP-Integration:** MCP Tool `lookup_tax_law(gesetz, paragraph)` → gibt Gesetzestext + Metadaten zurück
4. **Phase 4 — Versionierung:** Gesetzesversionen in DB speichern, damit Buchungen historisch korrekt dem damals geltenden Recht zugeordnet werden können

### 5.4 Wichtige Hinweise

- **Keine Rechtsverbindlichkeit:** Alle GitHub-Repos und auch gesetze-im-internet.de weisen darauf hin, dass nur der Bundesgesetzblatt-Text rechtsverbindlich ist
- **GoBD-Relevanz:** Für die Verfahrensdokumentation muss dokumentiert werden, welche Gesetzesversion für welche Buchung herangezogen wurde
- **Update-Frequenz:** Steuergesetze ändern sich häufig (z.B. Jahressteuergesetz 2024 vom 05.12.2024, EStG-Änderung vom 16.01.2026) — tägliche Prüfung empfohlen
- **Datenresidenz:** Die Gesetze selbst sind öffentlich und unterliegen keiner Datenresidenz-Pflicht — können auch auf Vercel/Cloud gehostet werden

---

## 6. Technische Referenz

### 6.1 XML-DTD Struktur (gii-norm.dtd v1.01)

Wichtigste Elemente:
- `<dokumente>` — Root-Element, enthält alle `<norm>`-Elemente
- `<norm>` — Einzelner Paragraph/Abschnitt
  - `<metadaten>` — Metadaten (Kürzel, Datum, Fundstelle)
    - `<jurabk>` — Gesetzeskürzel (z.B. "KStG")
    - `<ausfertigung-datum>` — Ausfertigungsdatum
    - `<fundstelle>` — Veröffentlichungsnachweis
    - `<enbez>` — Einzelnorm-Bezeichnung (z.B. "§ 1")
    - `<titel>` — Titel der Norm
  - `<textdaten>` — Gesetzestext
    - `<Content>` — HTML-ähnlicher Inhalt

### 6.2 Direkte Download-URLs (Kopiervorlage)

```bash
# Alle relevanten Steuergesetze herunterladen
BASE="https://www.gesetze-im-internet.de"
LAWS="estg kstg_1977 ustg_1980 gewstg ao_1977 hgb bewg umwstg astg erbstg_1974 grestg_1983 invstg_2018 solzg_1995"

for law in $LAWS; do
  curl -o "${law}.zip" "${BASE}/${law}/xml.zip"
done
```

---

*Dieser Report dient als Grundlage für die Implementierung der Gesetzesdaten-Pipeline in der KI-Buchhaltungssoftware.*
