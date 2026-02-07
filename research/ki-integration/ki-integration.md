# KI-Integration in GmbH-Buchhaltungssoftware

## Umfassender Report: Claude als intelligenter Buchhalter

**Erstellt:** 2026-02-07
**Autor:** KI-Experte (Claude Opus 4.6)
**Version:** 1.3 (aktualisiert mit aktuellen Technologie-Versionen und MCP SDK v2)

---

## Inhaltsverzeichnis

1. [Claude Code als Buchhaltungs-Interface](#1-claude-code-als-buchhaltungs-interface)
2. [MCP (Model Context Protocol) fuer Buchhaltung](#2-mcp-model-context-protocol-fuer-buchhaltung)
3. [Claude Agent SDK / Anthropic API](#3-claude-agent-sdk--anthropic-api)
4. [KI-gestuetzte Belegverarbeitung](#4-ki-gestuetzte-belegverarbeitung)
5. [Automatische Kontierung und Buchung](#5-automatische-kontierung-und-buchung)
6. [Validierung und Qualitaetssicherung](#6-validierung-und-qualitaetssicherung)
7. [Natuerlichsprachliche Interaktion](#7-natuerlichsprachliche-interaktion)
8. [Risiken und Herausforderungen](#8-risiken-und-herausforderungen)
9. [Architektur-Pattern fuer KI-Buchfuehrung](#9-architektur-pattern-fuer-ki-buchfuehrung)
10. [Prompt Engineering fuer Buchhaltung](#10-prompt-engineering-fuer-buchhaltung)
11. [Vergleich und Bewertung](#11-vergleich-und-bewertung)

---

## 1. Claude Code als Buchhaltungs-Interface

### 1.1 Claude Code Projekt-Struktur fuer Buchhaltung

Claude Code ist ein agentenbasiertes CLI-Tool, das direkt im Terminal arbeitet und ueber CLAUDE.md-Dateien, Skills und MCP-Server erweitert werden kann. Fuer eine Buchhaltungssoftware eignet sich folgende Projektstruktur:

```
gmbh-buchhaltung/
├── CLAUDE.md                          # Projekt-Kontext und Konventionen
├── .claude/
│   ├── settings.json                  # Claude Code Einstellungen
│   ├── skills/
│   │   ├── buchung-erfassen/
│   │   │   ├── SKILL.md               # Buchungserfassung
│   │   │   └── templates/
│   │   │       ├── eingangsrechnung.md
│   │   │       └── ausgangsrechnung.md
│   │   ├── ustva-erstellen/
│   │   │   ├── SKILL.md               # USt-Voranmeldung
│   │   │   └── formular-mapping.json
│   │   ├── jahresabschluss/
│   │   │   ├── SKILL.md               # Jahresabschluss-Workflow
│   │   │   └── checkliste.md
│   │   ├── beleg-verarbeiten/
│   │   │   ├── SKILL.md               # Belegverarbeitung mit Vision
│   │   │   └── extract_invoice.py
│   │   └── kontenplan-verwalten/
│   │       ├── SKILL.md               # Kontenplan-Verwaltung
│   │       └── skr03-mapping.json
│   └── mcp-servers/
│       └── buchhaltung-server/        # Lokaler MCP-Server
│           ├── package.json
│           ├── tsconfig.json
│           └── src/
│               ├── index.ts
│               ├── tools/
│               ├── resources/
│               └── prompts/
├── backend/                           # REST-API Backend
│   ├── src/
│   │   ├── api/
│   │   ├── models/
│   │   └── services/
│   └── tests/
├── data/
│   ├── kontenplan/
│   │   ├── skr03.json
│   │   └── skr04.json
│   └── steuersaetze.json
└── docs/
    └── buchungsregeln.md
```

### 1.2 CLAUDE.md Konfiguration fuer Buchhaltungskontext

Die CLAUDE.md-Datei ist das Herzstck der Projektkonfiguration. Sie wird automatisch geladen und gibt Claude den gesamten Kontext:

```markdown
# GmbH-Buchhaltungssoftware

## Projekt-Uebersicht
Du bist ein KI-gestuetzter Buchhalter fuer deutsche GmbHs.
Du arbeitest mit dem SKR03-Kontenrahmen und befolgst die
Grundsaetze ordnungsmaessiger Buchfuehrung (GoB/GoBD).

## Technischer Stack
- Backend: Python 3.13 + FastAPI 0.128+
- Datenbank: PostgreSQL 17
- API-Dokumentation: OpenAPI 3.1
- MCP-Server: TypeScript + @modelcontextprotocol/server (v2, Zod v4)

## Buchhalterische Grundregeln
- IMMER doppelte Buchfuehrung (Soll an Haben)
- Jede Buchung braucht: Datum, Belegnummer, Buchungstext,
  Soll-Konto, Haben-Konto, Betrag
- USt-Saetze: 19% (Regelsteuersatz), 7% (ermaessigt), 0% (steuerfrei)
- Geschaeftsjahr: Kalenderjahr (01.01. - 31.12.)
- Waehrung: EUR

## Kontenrahmen (SKR03 Auszug)
- 0200-0499: Sachanlagen
- 0500-0699: Finanzanlagen
- 1000-1099: Kasse, Bank (z.B. 1200 = Bank)
- 1400-1499: Forderungen aus LuL
- 1600-1699: Verbindlichkeiten aus LuL
- 1770/1771: USt (Zahllast/Vorsteuer)
- 3300-3399: Wareneingang
- 4400-4499: Erloese
- 6000-6999: Betriebliche Aufwendungen

## API-Endpunkte
- POST /api/v1/bookings - Buchung erstellen
- GET /api/v1/bookings - Buchungen abfragen
- GET /api/v1/accounts/{id}/balance - Kontostand abfragen
- POST /api/v1/reports/ustva - USt-Voranmeldung erstellen
- POST /api/v1/documents/process - Beleg verarbeiten

## Wichtige Regeln
- NIEMALS Buchungen ohne Beleg erstellen
- IMMER Plausibilitaetspruefung vor dem Buchen
- Bei Unsicherheit: Vorschlag machen, NICHT automatisch buchen
- Betraege IMMER in Cent speichern (Integer, kein Float)
- Alle Zeitstempel in UTC, Anzeige in Europe/Berlin
```

### 1.3 Skills-System: Buchungsvorgaenge als Skills

Skills sind Verzeichnisse mit einer SKILL.md-Datei, die Claude Code beibringen, wie bestimmte Aufgaben zu erledigen sind. Sie werden automatisch geladen, wenn Claudes Aufgabe zur Skill-Beschreibung passt.

**Beispiel: Skill fuer Buchungserfassung** (`.claude/skills/buchung-erfassen/SKILL.md`):

```markdown
---
name: buchung-erfassen
description: Erfasst Buchungssaetze in der doppelten Buchfuehrung.
  Wird aktiviert bei Buchungsanweisungen wie "Buche...",
  "Erfasse die Rechnung...", "Verbuche den Bankumsatz...".
allowed-tools:
  - mcp__buchhaltung__create_booking
  - mcp__buchhaltung__get_account_balance
  - mcp__buchhaltung__validate_booking
---

# Buchung erfassen

## Workflow

1. **Informationen sammeln**: Extrahiere aus der Anweisung:
   - Datum (oder heutiges Datum)
   - Belegnummer (oder generiere naechste)
   - Buchungstext (kurze Beschreibung)
   - Betrag (brutto/netto beachten)
   - USt-Satz (19%, 7%, 0%)

2. **Konten bestimmen**:
   - Bestimme Soll- und Haben-Konto nach SKR03
   - Bei Unsicherheit: Nutze get_account_balance zur Pruefung
   - Beachte: Aufwand = Soll, Ertrag = Haben

3. **USt berechnen**:
   - Netto-Betrag ermitteln
   - USt separat buchen (1770 Vorsteuer / 1771 Umsatzsteuer)

4. **Validierung**:
   - Nutze validate_booking vor dem Erstellen
   - Pruefe: Soll = Haben (Summe muss 0 ergeben)
   - Pruefe: Konten existieren und sind bebuchbar

5. **Buchung erstellen**:
   - Nutze create_booking mit allen Feldern
   - Zeige dem Benutzer die erstellte Buchung

## Beispiel: Eingangsrechnung

Anweisung: "Buche Rechnung von Muster GmbH, 1.190 EUR brutto,
Bueromoebel"

Ergebnis:
| Soll  | Haben | Betrag    | Text                    |
|-------|-------|-----------|-------------------------|
| 0420  | 1600  | 1.000,00  | Bueromoebel Muster GmbH |
| 1576  | 1600  |   190,00  | VSt 19% Muster GmbH     |

Buchungstext: "ER Muster GmbH - Bueromoebel"
```

**Beispiel: Skill fuer USt-Voranmeldung** (`.claude/skills/ustva-erstellen/SKILL.md`):

```markdown
---
name: ustva-erstellen
description: Erstellt die Umsatzsteuer-Voranmeldung fuer einen
  gegebenen Zeitraum. Aktiviert bei "USt-Voranmeldung",
  "Umsatzsteuer melden", "ELSTER Voranmeldung".
allowed-tools:
  - mcp__buchhaltung__generate_ustva
  - mcp__buchhaltung__get_account_balance
  - mcp__buchhaltung__get_bookings_by_period
---

# USt-Voranmeldung erstellen

## Workflow

1. **Zeitraum bestimmen**: Monat oder Quartal
2. **Daten aggregieren**:
   - Konto 1771 (Umsatzsteuer) Summe ermitteln
   - Konto 1576 (Vorsteuer) Summe ermitteln
   - Innergemeinschaftliche Erwerbe pruefen
   - Steuerfreie Umsaetze (Par. 4 UStG) erfassen
3. **Kennzahlen berechnen**:
   - KZ 81: Steuerpflichtige Umsaetze 19%
   - KZ 86: Steuerpflichtige Umsaetze 7%
   - KZ 66: Vorsteuer
   - KZ 83: Zahllast / Erstattung
4. **Report generieren**: Formatiert als ELSTER-kompatible Ausgabe
5. **Validierung**: Gegenpruefung mit Kontensalden
```

### 1.4 Interaktion mit Backend-APIs

Claude Code interagiert ueber MCP-Tools mit dem Backend. Der MCP-Server fungiert als Bruecke zwischen Claude und der REST-API:

```
Claude Code Terminal
    ↓ (natuerliche Sprache)
Claude AI (Reasoning)
    ↓ (Tool-Aufruf)
MCP Server (buchhaltung-server)
    ↓ (HTTP-Request)
FastAPI Backend
    ↓ (SQL)
PostgreSQL Datenbank
```

### 1.5 Beispiel-Workflows im Claude Code Terminal

```bash
# Benutzer startet Claude Code im Projekt
$ claude

# Workflow 1: Rechnung verbuchen
> Buche die Eingangsrechnung Nr. 2024-0142 von der
> Muster IT GmbH ueber 2.380 EUR brutto fuer IT-Beratung.

Claude: Ich verbuche die Eingangsrechnung:
  - Lieferant: Muster IT GmbH
  - Rechnungsnr.: 2024-0142
  - Bruttobetrag: 2.380,00 EUR
  - Nettobetrag: 2.000,00 EUR
  - USt 19%: 380,00 EUR
  - Kategorie: IT-Beratung → Konto 6020 (EDV-Kosten)

  Buchungssatz:
  | Soll | Haben | Betrag   |
  |------|-------|----------|
  | 6020 | 1600  | 2.000,00 |
  | 1576 | 1600  |   380,00 |

  Soll ich diese Buchung durchfuehren? [J/N]

> J

Claude: Buchung erfolgreich erstellt (ID: BU-2024-00847).
  Verbindlichkeit gegenueber Muster IT GmbH: 2.380,00 EUR
  auf Konto 1600.

# Workflow 2: Kontostand abfragen
> Wie hoch ist der aktuelle Bankbestand?

Claude: Kontostand Bank (1200) zum 07.02.2026:
  Saldo: 45.230,17 EUR (Soll)
  Letzte Bewegung: 05.02.2026 - Miete Buero (-1.500,00 EUR)

# Workflow 3: Beleg verarbeiten
> /beleg-verarbeiten [Foto der Rechnung wird angehaengt]

Claude: Ich habe den Beleg analysiert:
  - Aussteller: Office Supplies AG
  - Datum: 03.02.2026
  - Betrag: 178,50 EUR brutto (150,00 + 28,50 USt)
  - Beschreibung: Druckerpapier, Toner
  - Vorgeschlagene Kontierung: 6815 (Bueromate.)
  Confidence: 94%

  Soll ich mit dieser Kontierung buchen?
```

---

## 2. MCP (Model Context Protocol) fuer Buchhaltung

### 2.1 MCP Server-Architektur

Das Model Context Protocol (MCP) ist ein offener Standard von Anthropic (November 2024), der mittlerweile von der Agentic AI Foundation (AAIF) unter der Linux Foundation verwaltet wird. MCP definiert drei Kernprimitive:

- **Tools**: Funktionen, die Claude aufrufen kann (z.B. Buchung erstellen)
- **Resources**: Datenquellen, die Claude lesen kann (z.B. Kontenplan)
- **Prompts**: Vordefinierte Prompt-Templates (z.B. Buchungsvorlage)

**Architektur-Diagramm:**

```
┌─────────────────┐     ┌──────────────────────┐     ┌──────────────┐
│   Claude Code   │────▶│  MCP Buchhaltung-     │────▶│  PostgreSQL  │
│   (MCP Client)  │◀────│  Server (TypeScript)  │◀────│  Datenbank   │
└─────────────────┘     └──────────────────────┘     └──────────────┘
     STDIO/HTTP              │          │
                             ▼          ▼
                     ┌───────────┐ ┌──────────┐
                     │  FastAPI  │ │  ELSTER   │
                     │  Backend  │ │  API      │
                     └───────────┘ └──────────┘
```

### 2.2 MCP Server Implementation (TypeScript)

```typescript
// src/index.ts - Buchhaltungs-MCP-Server
// MCP TypeScript SDK v2 - neues Package-Layout
import { McpServer } from "@modelcontextprotocol/server";
import { StdioServerTransport } from
  "@modelcontextprotocol/server/transports/stdio";
import * as z from "zod/v4";

const server = new McpServer(
  {
    name: "buchhaltung",
    version: "1.0.0",
  },
  {
    capabilities: {
      tools: {},
      resources: {},
      prompts: {},
      logging: {},
    },
  }
);

// ============================================================
// TOOLS
// ============================================================

// Tool: Buchung erstellen
server.tool(
  "create_booking",
  "Erstellt einen Buchungssatz in der doppelten Buchfuehrung",
  {
    date: z.string().describe("Buchungsdatum im Format YYYY-MM-DD"),
    reference: z.string().describe("Belegnummer"),
    description: z.string().describe("Buchungstext"),
    line_items: z.array(z.object({
      debit_account: z.string()
        .describe("Soll-Konto (SKR03 Kontonummer)"),
      credit_account: z.string()
        .describe("Haben-Konto (SKR03 Kontonummer)"),
      amount_cents: z.number().int()
        .describe("Betrag in Cent (Integer)"),
      tax_rate: z.enum(["0", "7", "19"])
        .describe("USt-Satz in Prozent"),
      tax_amount_cents: z.number().int().optional()
        .describe("USt-Betrag in Cent"),
    })).describe("Buchungszeilen (Soll an Haben)"),
  },
  async ({ date, reference, description, line_items }) => {
    // Validierung: Soll = Haben
    const totalDebit = line_items.reduce(
      (sum, item) => sum + item.amount_cents, 0
    );

    // API-Aufruf an Backend
    const response = await fetch(
      "http://localhost:8000/api/v1/bookings",
      {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          date, reference, description, line_items
        }),
      }
    );

    if (!response.ok) {
      const error = await response.json();
      return {
        content: [{
          type: "text",
          text: `Fehler bei Buchung: ${error.detail}`,
        }],
        isError: true,
      };
    }

    const booking = await response.json();
    return {
      content: [{
        type: "text",
        text: `Buchung erstellt: ID ${booking.id}\n` +
          `Datum: ${booking.date}\n` +
          `Beleg: ${booking.reference}\n` +
          `Text: ${booking.description}\n` +
          `Betrag: ${(totalDebit / 100).toFixed(2)} EUR`,
      }],
    };
  }
);

// Tool: Kontosaldo abfragen
server.tool(
  "get_account_balance",
  "Gibt den aktuellen Saldo eines Kontos zurueck",
  {
    account_number: z.string()
      .describe("SKR03 Kontonummer (z.B. '1200' fuer Bank)"),
    as_of_date: z.string().optional()
      .describe("Stichtag (YYYY-MM-DD), Standard: heute"),
  },
  async ({ account_number, as_of_date }) => {
    const dateParam = as_of_date
      ? `?as_of_date=${as_of_date}` : "";
    const response = await fetch(
      `http://localhost:8000/api/v1/accounts/` +
      `${account_number}/balance${dateParam}`
    );
    const data = await response.json();
    return {
      content: [{
        type: "text",
        text: `Konto ${account_number} ` +
          `(${data.account_name}):\n` +
          `Saldo: ${(data.balance_cents / 100).toFixed(2)} EUR` +
          ` (${data.side})\n` +
          `Stand: ${data.as_of_date}`,
      }],
    };
  }
);

// Tool: Buchung validieren (ohne zu speichern)
server.tool(
  "validate_booking",
  "Prueft einen Buchungssatz auf Korrektheit ohne ihn zu speichern",
  {
    line_items: z.array(z.object({
      debit_account: z.string(),
      credit_account: z.string(),
      amount_cents: z.number().int(),
      tax_rate: z.enum(["0", "7", "19"]),
    })),
  },
  async ({ line_items }) => {
    const response = await fetch(
      "http://localhost:8000/api/v1/bookings/validate",
      {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ line_items }),
      }
    );
    const result = await response.json();
    return {
      content: [{
        type: "text",
        text: result.valid
          ? "Buchung ist valide."
          : `Validierungsfehler:\n${result.errors.join("\n")}`,
      }],
    };
  }
);

// Tool: USt-Voranmeldung generieren
server.tool(
  "generate_ustva",
  "Erstellt die USt-Voranmeldung fuer einen Zeitraum",
  {
    year: z.number().int().describe("Jahr"),
    month: z.number().int().min(1).max(12)
      .describe("Monat (1-12)"),
  },
  async ({ year, month }) => {
    const response = await fetch(
      "http://localhost:8000/api/v1/reports/ustva",
      {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ year, month }),
      }
    );
    const ustva = await response.json();
    return {
      content: [{
        type: "text",
        text:
          `USt-Voranmeldung ${month}/${year}\n` +
          `================================\n` +
          `KZ 81 - Umsaetze 19%: ` +
          `${(ustva.kz81 / 100).toFixed(2)} EUR\n` +
          `KZ 86 - Umsaetze 7%: ` +
          `${(ustva.kz86 / 100).toFixed(2)} EUR\n` +
          `KZ 66 - Vorsteuer: ` +
          `${(ustva.kz66 / 100).toFixed(2)} EUR\n` +
          `KZ 83 - Zahllast: ` +
          `${(ustva.kz83 / 100).toFixed(2)} EUR\n`,
      }],
    };
  }
);

// Tool: Buchungen nach Zeitraum abfragen
server.tool(
  "get_bookings_by_period",
  "Ruft alle Buchungen eines Zeitraums ab",
  {
    from_date: z.string().describe("Von-Datum (YYYY-MM-DD)"),
    to_date: z.string().describe("Bis-Datum (YYYY-MM-DD)"),
    account_number: z.string().optional()
      .describe("Optional: Filter nach Kontonummer"),
  },
  async ({ from_date, to_date, account_number }) => {
    let url = `http://localhost:8000/api/v1/bookings` +
      `?from=${from_date}&to=${to_date}`;
    if (account_number) url += `&account=${account_number}`;

    const response = await fetch(url);
    const bookings = await response.json();
    return {
      content: [{
        type: "text",
        text: JSON.stringify(bookings, null, 2),
      }],
    };
  }
);

// Tool: Beleg mit Vision verarbeiten
server.tool(
  "process_document",
  "Verarbeitet einen Beleg (Rechnung, Quittung) und " +
  "extrahiert Buchungsdaten",
  {
    file_path: z.string()
      .describe("Pfad zur Belegdatei (PDF, JPG, PNG)"),
  },
  async ({ file_path }) => {
    const formData = new FormData();
    const fs = await import("fs");
    const fileBuffer = fs.readFileSync(file_path);
    formData.append("file", new Blob([fileBuffer]), file_path);

    const response = await fetch(
      "http://localhost:8000/api/v1/documents/process",
      { method: "POST", body: formData }
    );
    const result = await response.json();
    return {
      content: [{
        type: "text",
        text:
          `Beleg erkannt:\n` +
          `Aussteller: ${result.vendor}\n` +
          `Datum: ${result.date}\n` +
          `Rechnungsnr.: ${result.invoice_number}\n` +
          `Brutto: ${(result.gross_cents / 100).toFixed(2)} EUR\n` +
          `Netto: ${(result.net_cents / 100).toFixed(2)} EUR\n` +
          `USt: ${(result.tax_cents / 100).toFixed(2)} EUR ` +
          `(${result.tax_rate}%)\n` +
          `Vorgeschlagenes Konto: ${result.suggested_account} ` +
          `(${result.suggested_account_name})\n` +
          `Confidence: ${(result.confidence * 100).toFixed(0)}%`,
      }],
    };
  }
);

// ============================================================
// RESOURCES (MCP SDK v2 - server.resource() Convenience-Methode)
// Alternativ: server.registerResource() fuer erweiterte Optionen
// ============================================================

// Resource: Kontenplan SKR03
server.resource(
  "kontenplan-skr03",
  "buchhaltung://kontenplan/skr03",
  {
    description: "Vollstaendiger SKR03-Kontenrahmen mit " +
      "Kontonummern, Namen und Kategorien",
    mimeType: "application/json",
  },
  async () => {
    const response = await fetch(
      "http://localhost:8000/api/v1/chart-of-accounts/skr03"
    );
    const data = await response.json();
    return {
      contents: [{
        uri: "buchhaltung://kontenplan/skr03",
        mimeType: "application/json",
        text: JSON.stringify(data, null, 2),
      }],
    };
  }
);

// Resource: Aktuelle Salden aller Konten
server.resource(
  "aktuelle-salden",
  "buchhaltung://salden/aktuell",
  {
    description: "Aktuelle Salden aller bebuchten Konten",
    mimeType: "application/json",
  },
  async () => {
    const response = await fetch(
      "http://localhost:8000/api/v1/accounts/balances"
    );
    const data = await response.json();
    return {
      contents: [{
        uri: "buchhaltung://salden/aktuell",
        mimeType: "application/json",
        text: JSON.stringify(data, null, 2),
      }],
    };
  }
);

// Resource: Steuersaetze und steuerliche Parameter
server.resource(
  "steuersaetze",
  "buchhaltung://config/steuersaetze",
  {
    description: "Aktuelle Steuersaetze und steuerliche Parameter",
    mimeType: "application/json",
  },
  async () => {
    return {
      contents: [{
        uri: "buchhaltung://config/steuersaetze",
        mimeType: "application/json",
        text: JSON.stringify({
          ust_regelsatz: 19,
          ust_ermaessigt: 7,
          koerperschaftsteuer: 15,
          solidaritaetszuschlag: 5.5,
          gewerbesteuer_messzahl: 3.5,
          gewerbesteuer_hebesatz_beispiel: 400,
          freibetrag_gewerbesteuer: 24500,
        }, null, 2),
      }],
    };
  }
);

// ============================================================
// PROMPTS
// ============================================================

// Prompt: Eingangsrechnung buchen
server.prompt(
  "eingangsrechnung-buchen",
  "Hilft beim Verbuchen einer Eingangsrechnung",
  {
    lieferant: z.string().describe("Name des Lieferanten"),
    betrag: z.string().describe("Bruttobetrag in EUR"),
    beschreibung: z.string().describe("Beschreibung der Leistung"),
    rechnungsnummer: z.string().optional()
      .describe("Rechnungsnummer"),
  },
  ({ lieferant, betrag, beschreibung, rechnungsnummer }) => ({
    messages: [{
      role: "user",
      content: {
        type: "text",
        text:
          `Bitte verbuche folgende Eingangsrechnung:\n` +
          `- Lieferant: ${lieferant}\n` +
          `- Bruttobetrag: ${betrag} EUR\n` +
          `- Beschreibung: ${beschreibung}\n` +
          (rechnungsnummer
            ? `- Rechnungsnr.: ${rechnungsnummer}\n` : "") +
          `\nBitte:\n` +
          `1. Bestimme das passende Aufwandskonto (SKR03)\n` +
          `2. Berechne Netto und USt (19% Regelsteuersatz,\n` +
          `   sofern nicht anders angegeben)\n` +
          `3. Erstelle den Buchungssatz\n` +
          `4. Validiere und buche`,
      },
    }],
  })
);

// Prompt: Bankumsaetze kontieren
server.prompt(
  "bankumsaetze-kontieren",
  "Kontiert und verbucht importierte Bankumsaetze",
  {
    zeitraum: z.string()
      .describe("Zeitraum der Bankumsaetze (z.B. 'Januar 2026')"),
  },
  ({ zeitraum }) => ({
    messages: [{
      role: "user",
      content: {
        type: "text",
        text:
          `Bitte kontiere die Bankumsaetze fuer ${zeitraum}.\n\n` +
          `Workflow:\n` +
          `1. Rufe die unkonteierten Bankumsaetze ab\n` +
          `2. Fuer jeden Umsatz:\n` +
          `   a) Analysiere Verwendungszweck und Empfaenger\n` +
          `   b) Schlage Gegenkonto vor (mit Confidence)\n` +
          `   c) Bei Confidence > 90%: automatisch buchen\n` +
          `   d) Bei Confidence <= 90%: zur Freigabe vorlegen\n` +
          `3. Zeige Zusammenfassung am Ende`,
      },
    }],
  })
);

// MCP SDK v2: Transport-Anbindung
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("Buchhaltungs-MCP-Server gestartet (SDK v2)");
}

main().catch(console.error);
```

### 2.3 Claude Code MCP-Server Konfiguration

In der Claude Code Settings-Datei (`.claude/settings.json`) wird der MCP-Server registriert:

```json
{
  "mcpServers": {
    "buchhaltung": {
      "command": "npx",
      "args": ["tsx", ".claude/mcp-servers/buchhaltung-server/src/index.ts"],
      "env": {
        "API_BASE_URL": "http://localhost:8000",
        "API_KEY": "${BUCHHALTUNG_API_KEY}"
      }
    }
  }
}
```

**Hinweis**: Ab MCP SDK v2 heisst das Package `@modelcontextprotocol/server` (nicht mehr `@modelcontextprotocol/sdk`). Die `package.json` des MCP-Servers muss entsprechend aktualisiert werden:

```json
{
  "dependencies": {
    "@modelcontextprotocol/server": "^2.0.0",
    "zod": "^4.0.0"
  }
}
```

---

## 3. Claude Agent SDK / Anthropic API

### 3.1 Agent SDK fuer autonome Buchfuehrungs-Workflows

Das Claude Agent SDK (fruher Claude Code SDK) gibt Entwicklern die gleichen Tools, den Agent Loop und das Context Management, die auch Claude Code antreiben - programmierbar in Python und TypeScript.

**Installation:**

```bash
pip install claude-agent-sdk
```

**Zwei Ansaetze fuer Agent-Integration:**

| Ansatz | Vorteile | Nachteile |
|---|---|---|
| **Claude Agent SDK (In-Process)** | Kein IPC-Overhead, einfaches Debugging, ein Python-Prozess | Nur Python-Tools |
| **Externer MCP Server (TypeScript)** | Sprachunabhaengig, wiederverwendbar fuer Claude Code CLI | Separater Prozess, IPC-Latenz |

**Empfehlung**: Claude Agent SDK fuer Backend-Workflows (Batch-Verarbeitung, autonome Buchung), externer MCP Server fuer Claude Code CLI-Nutzung.

**Beispiel: Autonomer Buchfuehrungs-Agent:**

```python
# agent_buchhalter.py
from claude_agent_sdk import (
    ClaudeSDKClient, ClaudeAgentOptions,
    create_sdk_mcp_server, tool,
)
from typing import Any

# Custom Tools als In-Process MCP Server (kein separater Prozess!)
@tool("buchung_erstellen", "Erstellt einen Buchungssatz", {
    "datum": str, "beschreibung": str, "zeilen": list
})
async def buchung_erstellen(args: dict[str, Any]) -> dict:
    """Erstellt Buchung ueber Backend-API."""
    import httpx
    async with httpx.AsyncClient() as client:
        response = await client.post(
            "http://localhost:8000/api/v1/journal-entries",
            json={
                "entry_date": args["datum"],
                "description": args["beschreibung"],
                "source": "AI_GENERATED",
                "lines": args["zeilen"],
            },
        )
        result = response.json()
    return {"content": [{"type": "text", "text": f"Buchung erstellt: {result['id']}"}]}

# In-Process MCP Server (kein IPC-Overhead!)
buchhaltung_server = create_sdk_mcp_server(
    name="buchhaltung",
    version="1.0.0",
    tools=[buchung_erstellen],
)

# Agent konfigurieren
options = ClaudeAgentOptions(
    model="claude-sonnet-4-5-20250929",
    mcp_servers={
        "buchhaltung": buchhaltung_server,  # In-Process
        "extern": {                          # Externer MCP Server (optional)
            "type": "stdio",
            "command": "npx",
            "args": ["tsx", "mcp-server/src/index.ts"],
        },
    },
    allowed_tools=["mcp__buchhaltung__buchung_erstellen"],
    max_turns=20,
)

# Agent ausfuehren
async def main():
    async with ClaudeSDKClient(options=options) as client:
        await client.query(
            "Verarbeite alle Eingangsrechnungen aus dem Ordner "
            "/belege/2026/02/ und erstelle die Buchungssaetze."
        )
        async for msg in client.receive_response():
            print(msg)
```

### 3.2 Tool Use mit der Anthropic API (direkt)

Fuer individuelle Integration ohne Agent SDK kann die Anthropic API direkt mit Tool-Definitionen genutzt werden:

```python
# api_tool_use.py
import anthropic

client = anthropic.Anthropic()

# Tool-Definitionen
tools = [
    {
        "name": "create_booking",
        "description": "Erstellt einen Buchungssatz",
        "input_schema": {
            "type": "object",
            "properties": {
                "date": {
                    "type": "string",
                    "description": "Buchungsdatum (YYYY-MM-DD)"
                },
                "reference": {
                    "type": "string",
                    "description": "Belegnummer"
                },
                "description": {
                    "type": "string",
                    "description": "Buchungstext"
                },
                "debit_account": {
                    "type": "string",
                    "description": "Soll-Konto (SKR03)"
                },
                "credit_account": {
                    "type": "string",
                    "description": "Haben-Konto (SKR03)"
                },
                "amount_cents": {
                    "type": "integer",
                    "description": "Betrag in Cent"
                },
                "tax_rate": {
                    "type": "integer",
                    "enum": [0, 7, 19],
                    "description": "USt-Satz"
                },
            },
            "required": [
                "date", "reference", "description",
                "debit_account", "credit_account",
                "amount_cents", "tax_rate"
            ],
        },
    },
    {
        "name": "get_account_balance",
        "description": "Gibt den Saldo eines Kontos zurueck",
        "input_schema": {
            "type": "object",
            "properties": {
                "account_number": {
                    "type": "string",
                    "description": "SKR03 Kontonummer"
                },
            },
            "required": ["account_number"],
        },
    },
]

# Buchungs-Anfrage mit Tool Use
response = client.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=4096,
    system="""Du bist ein Buchhalter fuer eine deutsche GmbH.
    Nutze die verfuegbaren Tools fuer alle Buchungsoperationen.
    Arbeite mit SKR03. Buche immer mit doppelter Buchfuehrung.
    Betraege immer in Cent (Integer).""",
    tools=tools,
    messages=[
        {
            "role": "user",
            "content": "Buche die Telefonrechnung von Telekom "
                       "ueber 59,50 EUR brutto fuer Januar 2026.",
        }
    ],
)

# Tool-Aufrufe verarbeiten
for block in response.content:
    if block.type == "tool_use":
        print(f"Tool: {block.name}")
        print(f"Input: {block.input}")
        # Hier: Tool ausfuehren und Ergebnis zuruecksenden
```

### 3.3 Structured Output fuer Buchungssaetze

Structured Outputs garantieren, dass Claude valides JSON zurueckgibt, das exakt einem Schema entspricht:

```python
# structured_booking.py
import anthropic
from pydantic import BaseModel
from typing import Optional

class BookingLineItem(BaseModel):
    debit_account: str
    debit_account_name: str
    credit_account: str
    credit_account_name: str
    amount_cents: int
    tax_rate: int
    tax_amount_cents: int
    description: str

class BookingProposal(BaseModel):
    date: str
    reference: str
    booking_text: str
    line_items: list[BookingLineItem]
    total_gross_cents: int
    total_net_cents: int
    total_tax_cents: int
    confidence: float
    reasoning: str

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-sonnet-4-5-20250929",
    max_tokens=4096,
    system="Du bist ein Buchhalter. Erstelle Buchungsvorschlaege "
           "als strukturiertes JSON.",
    messages=[{
        "role": "user",
        "content": "Eingangsrechnung: Weber Consulting GmbH, "
                   "3.570 EUR brutto, Unternehmensberatung, "
                   "Rechnungsnr. WC-2026-0088, Datum 01.02.2026"
    }],
    output_config={
        "format": {
            "type": "json_schema",
            "schema": BookingProposal.model_json_schema(),
        }
    },
)

# Ergebnis ist garantiert valides JSON
import json
proposal = BookingProposal.model_validate_json(
    response.content[0].text
)
print(f"Buchungsvorschlag (Confidence: {proposal.confidence}):")
for item in proposal.line_items:
    print(f"  {item.debit_account} an {item.credit_account}: "
          f"{item.amount_cents / 100:.2f} EUR - {item.description}")

# Alternativ (einfacher): client.messages.parse() mit Pydantic
# Verfuegbar ab Anthropic SDK 1.x
parsed_result = client.messages.parse(
    model="claude-sonnet-4-5-20250929",
    max_tokens=4096,
    messages=[{
        "role": "user",
        "content": "Eingangsrechnung: Weber Consulting GmbH, "
                   "3.570 EUR brutto, Unternehmensberatung"
    }],
    output_format=BookingProposal,  # Pydantic-Modell direkt!
)

# Ergebnis ist bereits ein BookingProposal-Objekt
proposal = parsed_result.parsed_output
print(f"Buchung: {proposal.booking_text}")

# Hinweis: messages.parse() ist die modernere Alternative zu
# output_config + model_validate_json(). Beide Ansaetze sind valide.
```

### 3.4 Streaming fuer Echtzeit-Feedback

```python
# streaming_booking.py
import anthropic

client = anthropic.Anthropic()

# Streaming fuer lange Auswertungen
with client.messages.stream(
    model="claude-sonnet-4-5-20250929",
    max_tokens=8192,
    system="Du bist ein Buchhalter. Erstelle detaillierte "
           "Finanzanalysen.",
    messages=[{
        "role": "user",
        "content": "Erstelle eine GuV-Analyse fuer Q4 2025 "
                   "basierend auf den folgenden Kontensalden: ..."
    }],
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

### 3.5 Batch-Verarbeitung von Belegen

```python
# batch_processing.py
import anthropic
import base64
from pathlib import Path

client = anthropic.Anthropic()

def process_invoice_batch(invoice_dir: str):
    """Verarbeitet einen Ordner voller Rechnungen als Batch."""
    invoices = list(Path(invoice_dir).glob("*.pdf")) + \
               list(Path(invoice_dir).glob("*.jpg")) + \
               list(Path(invoice_dir).glob("*.png"))

    # Batch-Requests erstellen (50% Rabatt auf Token-Kosten)
    requests = []
    for i, invoice_path in enumerate(invoices):
        with open(invoice_path, "rb") as f:
            image_data = base64.standard_b64encode(
                f.read()
            ).decode("utf-8")

        media_type = (
            "image/jpeg" if invoice_path.suffix in [".jpg", ".jpeg"]
            else "image/png" if invoice_path.suffix == ".png"
            else "application/pdf"
        )

        requests.append({
            "custom_id": f"invoice-{i}-{invoice_path.name}",
            "params": {
                "model": "claude-sonnet-4-5-20250929",
                "max_tokens": 2048,
                "system": "Extrahiere alle Rechnungsdaten. "
                          "Antworte als JSON.",
                "messages": [{
                    "role": "user",
                    "content": [
                        {
                            "type": "image",
                            "source": {
                                "type": "base64",
                                "media_type": media_type,
                                "data": image_data,
                            },
                        },
                        {
                            "type": "text",
                            "text": "Extrahiere: Aussteller, "
                                "Datum, Rechnungsnummer, "
                                "Positionen, Netto, USt, Brutto, "
                                "IBAN, USt-IdNr.",
                        },
                    ],
                }],
            },
        })

    # Batch absenden
    batch = client.batches.create(requests=requests)
    print(f"Batch erstellt: {batch.id}")
    print(f"Anzahl Belege: {len(requests)}")
    return batch.id
```

---

## 4. KI-gestuetzte Belegverarbeitung

### 4.1 OCR + Claude Vision fuer Belegerkennung

Claude's multimodale Faehigkeiten ermoeglichen die direkte Verarbeitung von Rechnungsbildern und PDFs ohne separates OCR-Tool:

```python
# beleg_verarbeitung.py
import anthropic
import base64
from pydantic import BaseModel
from typing import Optional

class InvoiceData(BaseModel):
    vendor_name: str
    vendor_address: Optional[str]
    vendor_tax_id: Optional[str]      # USt-IdNr
    invoice_number: str
    invoice_date: str                  # YYYY-MM-DD
    due_date: Optional[str]
    line_items: list[dict]
    net_amount_cents: int
    tax_rate: int
    tax_amount_cents: int
    gross_amount_cents: int
    currency: str
    iban: Optional[str]
    payment_reference: Optional[str]
    confidence: float

client = anthropic.Anthropic()

def extract_invoice_data(image_path: str) -> InvoiceData:
    """Extrahiert Rechnungsdaten aus einem Belegbild."""
    with open(image_path, "rb") as f:
        image_data = base64.standard_b64encode(
            f.read()
        ).decode("utf-8")

    response = client.messages.create(
        model="claude-sonnet-4-5-20250929",
        max_tokens=2048,
        system="""Du bist ein Experte fuer deutsche
        Rechnungsverarbeitung. Extrahiere alle relevanten
        Daten praezise. Betraege immer in Cent als Integer.
        Datumsformat: YYYY-MM-DD. Wenn ein Feld nicht
        erkennbar ist, setze null.""",
        messages=[{
            "role": "user",
            "content": [
                {
                    "type": "image",
                    "source": {
                        "type": "base64",
                        "media_type": "image/jpeg",
                        "data": image_data,
                    },
                },
                {
                    "type": "text",
                    "text": "Extrahiere alle Rechnungsdaten "
                            "aus diesem Beleg.",
                },
            ],
        }],
        output_config={
            "format": {
                "type": "json_schema",
                "schema": InvoiceData.model_json_schema(),
            }
        },
    )

    return InvoiceData.model_validate_json(
        response.content[0].text
    )
```

### 4.2 Automatische Kategorisierung und Kontierung

```python
# kontierung.py
class AccountingSuggestion(BaseModel):
    debit_account: str
    debit_account_name: str
    credit_account: str
    credit_account_name: str
    tax_account: str
    tax_account_name: str
    booking_text: str
    confidence: float
    reasoning: str
    alternative_accounts: list[dict]

def suggest_account_assignment(
    invoice: InvoiceData,
    booking_history: list[dict],
) -> AccountingSuggestion:
    """Schlaegt Kontierung basierend auf Rechnungsdaten
    und Buchungshistorie vor."""

    # Historische Buchungen des Lieferanten als Kontext
    vendor_history = [
        b for b in booking_history
        if b["vendor"] == invoice.vendor_name
    ]

    history_context = ""
    if vendor_history:
        history_context = (
            "\n\nBisherige Buchungen dieses Lieferanten:\n"
        )
        for b in vendor_history[-5:]:
            history_context += (
                f"- {b['date']}: Konto {b['debit_account']} "
                f"({b['description']})\n"
            )

    response = client.messages.create(
        model="claude-sonnet-4-5-20250929",
        max_tokens=1024,
        system="""Du bist ein Buchhalter fuer eine deutsche GmbH
        mit SKR03. Bestimme die korrekte Kontierung fuer den
        gegebenen Beleg. Beruecksichtige die Buchungshistorie
        fuer konsistente Kontierung.""",
        messages=[{
            "role": "user",
            "content":
                f"Kontiere folgenden Beleg:\n"
                f"Lieferant: {invoice.vendor_name}\n"
                f"Beschreibung: {invoice.line_items}\n"
                f"Nettobetrag: "
                f"{invoice.net_amount_cents / 100:.2f} EUR\n"
                f"USt-Satz: {invoice.tax_rate}%\n"
                f"{history_context}",
        }],
        output_config={
            "format": {
                "type": "json_schema",
                "schema":
                    AccountingSuggestion.model_json_schema(),
            }
        },
    )

    return AccountingSuggestion.model_validate_json(
        response.content[0].text
    )
```

### 4.3 Lernende Kontierungsvorschlaege

Das System verbessert sich ueber die Zeit durch:

1. **Feedback-Loop**: Bestaetigt oder korrigiert der Nutzer einen Vorschlag, wird dies gespeichert
2. **Few-Shot Learning**: Historische Kontierungen werden als Beispiele in den Prompt eingefuegt
3. **Vendor-Profile**: Pro Lieferant werden typische Kontierungen gespeichert
4. **Regelbasierter Fallback**: Bei niedrigem Confidence-Score greifen deterministische Regeln

```python
# lernende_kontierung.py
def get_enhanced_prompt_with_history(
    vendor_name: str,
    description: str,
    db_session,
) -> str:
    """Erstellt einen angereicherten Prompt mit
    historischen Kontierungen als Few-Shot-Examples."""

    # Letzte 10 Kontierungen dieses Lieferanten
    history = db_session.query(Booking).filter(
        Booking.vendor == vendor_name
    ).order_by(Booking.date.desc()).limit(10).all()

    # Letzte 5 aehnliche Beschreibungen (anderer Lieferant)
    similar = db_session.query(Booking).filter(
        Booking.description.ilike(f"%{description}%")
    ).limit(5).all()

    examples = "## Bisherige Kontierungen\n\n"
    for b in history:
        examples += (
            f"- {b.vendor}: '{b.description}' → "
            f"Konto {b.debit_account} "
            f"(Korrektur: {'Nein' if not b.was_corrected else 'Ja, zu ' + b.corrected_account})\n"
        )

    if similar:
        examples += "\n## Aehnliche Buchungen (andere Lieferanten)\n\n"
        for b in similar:
            examples += (
                f"- {b.vendor}: '{b.description}' → "
                f"Konto {b.debit_account}\n"
            )

    return examples
```

### 4.4 Umgang mit mehrdeutigen Belegen

```python
# mehrdeutige_belege.py
class AmbiguousDocumentResult(BaseModel):
    is_ambiguous: bool
    primary_interpretation: dict
    alternative_interpretations: list[dict]
    questions_for_user: list[str]
    recommended_action: str  # "auto_book", "ask_user", "reject"

def handle_ambiguous_document(
    invoice: InvoiceData,
) -> AmbiguousDocumentResult:
    """Erkennt und behandelt mehrdeutige Belege."""

    response = client.messages.create(
        model="claude-sonnet-4-5-20250929",
        max_tokens=1024,
        system="""Analysiere den Beleg auf Mehrdeutigkeiten.
        Moegliche Probleme:
        - Unklar ob Eingangs- oder Ausgangsrechnung
        - Mehrere moegliche Aufwandskonten
        - USt-Satz unklar (19% vs 7% vs steuerfrei)
        - Gemischte Steuersaetze auf einer Rechnung
        - Innergemeinschaftliche Lieferung unklar
        - Abgrenzung Anlageverm. vs Aufwand unklar
          (GWG-Grenze 800 EUR netto beachten)""",
        messages=[{
            "role": "user",
            "content": f"Pruefe diesen Beleg: {invoice.model_dump_json()}",
        }],
        output_config={
            "format": {
                "type": "json_schema",
                "schema":
                    AmbiguousDocumentResult.model_json_schema(),
            }
        },
    )

    return AmbiguousDocumentResult.model_validate_json(
        response.content[0].text
    )
```

---

## 5. Automatische Kontierung und Buchung

### 5.1 Regelbasierte Kontierung + KI-Verbesserung

Ein hybrides System kombiniert deterministische Regeln mit KI-Verbesserung:

```python
# hybrid_kontierung.py
from dataclasses import dataclass

@dataclass
class KontierungsRegel:
    """Deterministische Kontierungsregel."""
    name: str
    vendor_pattern: str       # Regex fuer Lieferantennamen
    description_pattern: str  # Regex fuer Beschreibung
    debit_account: str
    priority: int             # Hoehere Prio = wird zuerst geprueft

# Regelwerk (deterministisch, immer gleich)
KONTIERUNGSREGELN = [
    KontierungsRegel(
        name="Miete",
        vendor_pattern=r"(?i)(immobilien|hausverwaltung|miete)",
        description_pattern=r"(?i)(miete|pacht|bueromiete)",
        debit_account="4210",  # Miete
        priority=100,
    ),
    KontierungsRegel(
        name="Telefon/Internet",
        vendor_pattern=r"(?i)(telekom|vodafone|o2|1und1)",
        description_pattern=r"(?i)(telefon|internet|mobilfunk)",
        debit_account="6805",  # Telefon
        priority=90,
    ),
    KontierungsRegel(
        name="Versicherung",
        vendor_pattern=r"(?i)(allianz|axa|hdi|zurich)",
        description_pattern=r"(?i)(versicherung|police|praemie)",
        debit_account="6430",  # Versicherungen
        priority=90,
    ),
    KontierungsRegel(
        name="Kfz-Kosten",
        vendor_pattern=r"(?i)(aral|shell|total|adac)",
        description_pattern=r"(?i)(tanken|benzin|diesel|kfz)",
        debit_account="6530",  # Kfz-Kosten
        priority=80,
    ),
    KontierungsRegel(
        name="Porto",
        vendor_pattern=r"(?i)(deutsche post|dhl|ups|hermes)",
        description_pattern=r"(?i)(porto|versand|paket)",
        debit_account="6810",  # Porto
        priority=80,
    ),
]

import re

def regelbasierte_kontierung(
    vendor: str,
    description: str,
) -> tuple[str, float] | None:
    """Versucht regelbasierte Kontierung.
    Gibt (Konto, Confidence) oder None zurueck."""

    for regel in sorted(
        KONTIERUNGSREGELN, key=lambda r: r.priority, reverse=True
    ):
        vendor_match = re.search(regel.vendor_pattern, vendor)
        desc_match = re.search(
            regel.description_pattern, description
        )

        if vendor_match and desc_match:
            return (regel.debit_account, 0.99)
        elif vendor_match or desc_match:
            return (regel.debit_account, 0.75)

    return None  # Keine Regel greift → KI uebernimmt


def hybrid_kontierung(
    invoice: InvoiceData,
    booking_history: list[dict],
) -> AccountingSuggestion:
    """Kombiniert Regeln und KI fuer optimale Kontierung."""

    # Schritt 1: Regelbasierte Kontierung versuchen
    regel_result = regelbasierte_kontierung(
        invoice.vendor_name,
        str(invoice.line_items),
    )

    if regel_result and regel_result[1] >= 0.95:
        # Hohe Confidence → Regel verwenden
        return AccountingSuggestion(
            debit_account=regel_result[0],
            confidence=regel_result[1],
            reasoning="Regelbasierte Kontierung",
            # ... weitere Felder
        )

    # Schritt 2: KI-Kontierung mit Regelhinweis
    return suggest_account_assignment(
        invoice=invoice,
        booking_history=booking_history,
    )
```

### 5.2 Confidence Scores und Eskalation

```python
# confidence_workflow.py
from enum import Enum

class BookingAction(Enum):
    AUTO_BOOK = "auto_book"          # Confidence >= 95%
    SUGGEST_AND_CONFIRM = "suggest"  # 70% <= Confidence < 95%
    MANUAL_REVIEW = "manual"         # Confidence < 70%
    REJECT = "reject"                # Confidence < 30%

def determine_action(
    confidence: float,
    amount_cents: int,
    is_new_vendor: bool,
) -> BookingAction:
    """Bestimmt Aktion basierend auf Confidence und Risiko."""

    # Grosse Betraege erfordern hoehere Confidence
    high_value_threshold = 500000  # 5.000 EUR
    if amount_cents > high_value_threshold:
        confidence *= 0.9  # Effektiv hoehere Schwelle

    # Neue Lieferanten erfordern hoehere Confidence
    if is_new_vendor:
        confidence *= 0.85

    if confidence >= 0.95:
        return BookingAction.AUTO_BOOK
    elif confidence >= 0.70:
        return BookingAction.SUGGEST_AND_CONFIRM
    elif confidence >= 0.30:
        return BookingAction.MANUAL_REVIEW
    else:
        return BookingAction.REJECT
```

### 5.3 Batch-Kontierung von Bankumsaetzen

```python
# bank_batch_kontierung.py
from dataclasses import dataclass

@dataclass
class BankTransaction:
    date: str
    amount_cents: int
    counterparty: str
    reference: str
    iban: str

@dataclass
class BatchResult:
    auto_booked: list[dict]
    needs_review: list[dict]
    rejected: list[dict]
    total: int

async def batch_kontiere_bankumsaetze(
    transactions: list[BankTransaction],
    booking_history: list[dict],
) -> BatchResult:
    """Kontiert eine Liste von Bankumsaetzen im Batch."""

    result = BatchResult(
        auto_booked=[], needs_review=[], rejected=[], total=0
    )

    # Alle Transaktionen als Batch an Claude senden
    # (effizienter als einzeln)
    tx_text = "\n".join([
        f"{i+1}. {t.date} | {t.amount_cents/100:.2f} EUR | "
        f"{t.counterparty} | {t.reference}"
        for i, t in enumerate(transactions)
    ])

    response = client.messages.create(
        model="claude-sonnet-4-5-20250929",
        max_tokens=8192,
        system="""Du bist Buchhalter. Kontiere die folgenden
        Bankumsaetze. Fuer jeden Umsatz:
        - Bestimme ob Einnahme oder Ausgabe
        - Bestimme das Gegenkonto (SKR03)
        - Schaetze den USt-Satz
        - Gib einen Confidence-Score (0-1)
        Antworte als JSON-Array.""",
        messages=[{
            "role": "user",
            "content": f"Bankumsaetze:\n{tx_text}",
        }],
    )

    # Ergebnisse verarbeiten und nach Confidence sortieren
    # ... (Verarbeitung wie oben beschrieben)

    return result
```

---

## 6. Validierung und Qualitaetssicherung

### 6.1 KI-basierte Plausibilitaetspruefung

```python
# plausibilitaet.py
class PlausibilityCheck(BaseModel):
    is_plausible: bool
    issues: list[dict]       # Gefundene Probleme
    warnings: list[dict]     # Warnungen (nicht kritisch)
    suggestions: list[str]   # Verbesserungsvorschlaege

def check_booking_plausibility(
    booking: dict,
    account_balances: dict,
    recent_bookings: list[dict],
) -> PlausibilityCheck:
    """Prueft eine Buchung auf Plausibilitaet."""

    response = client.messages.create(
        model="claude-sonnet-4-5-20250929",
        max_tokens=2048,
        system="""Du bist ein Wirtschaftspruefer. Pruefe die
        Buchung auf Plausibilitaet. Achte auf:

        1. KONTIERUNG:
           - Passt das Soll-Konto zum Geschaeftsvorfall?
           - Passt das Haben-Konto?
           - Ist die Kontenklasse korrekt?

        2. BETRAG:
           - Ist der Betrag ungewoehnlich hoch/niedrig?
           - Vergleiche mit historischen Buchungen
           - Stimmt die USt-Berechnung?

        3. ZEITLICHE ABGRENZUNG:
           - Gehoert die Buchung in diesen Zeitraum?
           - Gibt es Abgrenzungsbedarf?

        4. KONSISTENZ:
           - Widerspricht die Buchung anderen Buchungen?
           - Ist die Belegnummerierung konsistent?

        5. STEUERLICHE KORREKTHEIT:
           - Richtiger USt-Satz?
           - Vorsteuerabzug berechtigt?
           - Innergemeinschaftlich korrekt?""",
        messages=[{
            "role": "user",
            "content":
                f"Pruefe diese Buchung:\n"
                f"{json.dumps(booking, indent=2)}\n\n"
                f"Aktuelle Kontensalden:\n"
                f"{json.dumps(account_balances, indent=2)}\n\n"
                f"Letzte 10 Buchungen:\n"
                f"{json.dumps(recent_bookings, indent=2)}",
        }],
        output_config={
            "format": {
                "type": "json_schema",
                "schema": PlausibilityCheck.model_json_schema(),
            }
        },
    )

    return PlausibilityCheck.model_validate_json(
        response.content[0].text
    )
```

### 6.2 Anomalie-Erkennung

```python
# anomalie_erkennung.py
class AnomalyReport(BaseModel):
    anomalies_found: bool
    anomalies: list[dict]
    risk_score: float        # 0 (kein Risiko) bis 1 (kritisch)
    recommended_actions: list[str]

def detect_anomalies(
    bookings: list[dict],
    period: str,
) -> AnomalyReport:
    """Erkennt Anomalien in Buchungen eines Zeitraums."""

    response = client.messages.create(
        model="claude-sonnet-4-5-20250929",
        max_tokens=4096,
        system="""Analysiere die Buchungen auf Anomalien:

        1. BETRAGSANOMALIEN:
           - Ungewoehnlich hohe/niedrige Betraege
           - Runde Betraege bei normalerweise krummen
           - Betraege knapp unter Schwellenwerten (z.B. 799 EUR
             bei GWG-Grenze 800 EUR)

        2. KONTENANOMALIEN:
           - Buchungen auf untypischen Konten
           - Konten mit ungewoehnlichem Saldo
           - Buchungen auf gesperrte/inaktive Konten

        3. MUSTERANOMALIEN:
           - Haeufung von Buchungen am Monatsende
           - Gegenbuchungen (Buchung + sofortige Stornierung)
           - Splittbuchungen knapp unter Freigabegrenzen

        4. ZEITANOMALIEN:
           - Buchungen an Wochenenden/Feiertagen
           - Ungewoehnliche Buchungszeiten
           - Luecken in Belegnummern""",
        messages=[{
            "role": "user",
            "content":
                f"Analysiere Buchungen {period}:\n"
                f"{json.dumps(bookings, indent=2)}",
        }],
        output_config={
            "format": {
                "type": "json_schema",
                "schema": AnomalyReport.model_json_schema(),
            }
        },
    )

    return AnomalyReport.model_validate_json(
        response.content[0].text
    )
```

### 6.3 Human-in-the-Loop Workflow

```python
# human_in_the_loop.py
from enum import Enum
from dataclasses import dataclass

class ReviewStatus(Enum):
    PENDING = "pending"
    APPROVED = "approved"
    REJECTED = "rejected"
    MODIFIED = "modified"

@dataclass
class ReviewItem:
    booking_id: str
    booking: dict
    ai_confidence: float
    ai_reasoning: str
    plausibility_check: PlausibilityCheck
    status: ReviewStatus
    reviewer: str | None = None
    reviewer_comment: str | None = None

def create_review_queue(
    bookings: list[dict],
    threshold: float = 0.95,
) -> list[ReviewItem]:
    """Erstellt eine Review-Queue fuer Buchungen
    unter dem Confidence-Schwellenwert."""

    queue = []
    for booking in bookings:
        if booking["confidence"] < threshold:
            check = check_booking_plausibility(
                booking,
                get_current_balances(),
                get_recent_bookings(),
            )
            queue.append(ReviewItem(
                booking_id=booking["id"],
                booking=booking,
                ai_confidence=booking["confidence"],
                ai_reasoning=booking["reasoning"],
                plausibility_check=check,
                status=ReviewStatus.PENDING,
            ))

    # Sortierung: Niedrigste Confidence zuerst
    queue.sort(key=lambda r: r.ai_confidence)
    return queue
```

---

## 7. Natuerlichsprachliche Interaktion

### 7.1 Abfragen (Read Operations)

Die natuerlichsprachliche Abfrage ist eine Kernstaerke des Claude-basierten Systems. Benutzer koennen Finanzdaten in natuerlicher Sprache abfragen:

```
Benutzer: "Wie hoch war der Umsatz im Q3?"

Claude: [Ruft intern get_bookings_by_period und
         get_account_balance auf]

Antwort: Der Umsatz im Q3 2025 (Juli - September):

  Konto 4400 (Erloese 19% USt):    125.340,00 EUR
  Konto 4300 (Erloese 7% USt):       8.200,00 EUR
  Konto 4400 (Erloese steuerfrei):    3.100,00 EUR
  ─────────────────────────────────────────────
  Gesamtumsatz Q3 2025:             136.640,00 EUR

  Vergleich Q2 2025: 128.900,00 EUR (+6,0%)
  Vergleich Q3 2024: 119.500,00 EUR (+14,3%)
```

### 7.2 Buchungsanweisungen (Write Operations)

```
Benutzer: "Buche die Rechnung von Lieferant X ueber 1.190 EUR
           brutto"

Claude: Ich verbuche die Eingangsrechnung:

  Lieferant: Lieferant X
  Brutto: 1.190,00 EUR
  Netto:  1.000,00 EUR (bei 19% USt)
  USt:      190,00 EUR

  Frage: Wofuer war die Rechnung? Ich schlage dann
  das passende Aufwandskonto vor.

Benutzer: "IT-Dienstleistung"

Claude: Kontierungsvorschlag:
  | Soll | Haben | Betrag   | Text                      |
  |------|-------|----------|---------------------------|
  | 6020 | 1600  | 1.000,00 | IT-DL Lieferant X         |
  | 1576 | 1600  |   190,00 | VSt 19% Lieferant X       |

  Aufwandskonto: 6020 (Fremdleistungen EDV)
  Confidence: 92%

  Soll ich buchen? [J/N]
```

### 7.3 Berichte generieren

```
Benutzer: "Erstelle die USt-Voranmeldung fuer Oktober"

Claude: [Ruft generate_ustva auf]

USt-Voranmeldung Oktober 2025
================================
Steuernummer: 123/456/78901
Zeitraum: 01.10.2025 - 31.10.2025

Steuerpflichtige Umsaetze:
  KZ 81 - Umsaetze 19%:       82.500,00 EUR
          USt darauf:          15.675,00 EUR
  KZ 86 - Umsaetze 7%:         4.200,00 EUR
          USt darauf:             294,00 EUR

Vorsteuer:
  KZ 66 - Abziehbare VSt:      9.230,00 EUR

Zahllast:
  KZ 83 - Verbleibend:         6.739,00 EUR

Faelligkeit: 10.11.2025
IBAN fuer Zahlung: DE89 3704 0044 0532 0130 00

Soll ich die Voranmeldung zur ELSTER-Uebermittlung
vorbereiten?
```

### 7.4 Erklaerungen und Analysen

```
Benutzer: "Warum ist das Konto 1200 im Minus?"

Claude: Konto 1200 (Bank) hat aktuell einen negativen
Saldo von -2.340,17 EUR. Analyse:

Die letzten signifikanten Bewegungen:
  03.02. Miete Buero:           -1.500,00 EUR
  03.02. Versicherung:            -450,00 EUR
  02.02. Gehaelter:             -8.200,00 EUR
  01.02. Eingang Kunde A:       +5.600,00 EUR
  31.01. Saldo Vortrag:         +2.209,83 EUR

Ursache: Die Gehaltsauszahlungen am 02.02. in Kombination
mit Miete und Versicherung am 03.02. haben den Saldo ins
Minus gedrueckt. Der letzte groessere Zahlungseingang
(Kunde A, 5.600 EUR) reichte nicht aus.

Erwartete Eingaenge:
  - Rechnung RE-2026-0047 (Kunde B): 3.570,00 EUR
    (faellig 10.02.)
  - Rechnung RE-2026-0051 (Kunde C): 7.140,00 EUR
    (faellig 15.02.)

Prognose: Der Saldo sollte nach Eingang von Kunde B
am ~10.02. wieder positiv sein.
```

---

## 8. Risiken und Herausforderungen

### 8.1 Halluzinationen bei Finanzdaten

**Problem**: LLMs koennen plausibel klingende aber falsche Zahlen generieren. Bei Finanzdaten ist dies besonders kritisch.

**Mitigation-Strategien**:

1. **Keine freie Zahlengenerierung**: Claude darf Zahlen NUR aus der Datenbank/Tools lesen, NIEMALS selbst "erinnern"
2. **Deterministische Berechnungen auslagern**: USt-Berechnung, Salden-Ermittlung etc. im Backend, NICHT durch Claude
3. **Gegenpruefung**: Jede Claude-Ausgabe mit Zahlen wird programmatisch gegen die Datenbank validiert
4. **Structured Output**: JSON-Schema erzwingt korrekte Datentypen

```python
# Beispiel: Sichere Zahlenausgabe
# FALSCH: Claude soll Zahlen frei formulieren
# RICHTIG: Claude bekommt Zahlen aus Tools und formatiert sie

# Im System Prompt:
ANTI_HALLUCINATION_PROMPT = """
WICHTIG: Du darfst NIEMALS Zahlen, Betraege oder Salden
aus dem Gedaechtnis nennen. IMMER zuerst das entsprechende
Tool aufrufen:
- Fuer Kontensalden: get_account_balance
- Fuer Umsaetze: get_bookings_by_period
- Fuer USt-Berechnung: calculate_tax (Backend)

Wenn ein Tool nicht verfuegbar ist, antworte:
"Ich kann diese Zahl gerade nicht zuverlaessig ermitteln."
"""
```

### 8.2 Determinismus

**Problem**: Gleiche Eingabe sollte gleiche Buchung ergeben. LLMs sind aber nicht deterministisch.

**Loesung**:
- `temperature=0` fuer alle Buchungsentscheidungen
- Regelbasierte Logik fuer eindeutige Faelle (kein LLM noetig)
- LLM nur fuer mehrdeutige Faelle mit Human-in-the-Loop
- Alle Buchungsentscheidungen werden geloggt (Audit Trail)

### 8.3 GoBD-Compliance

Die GoBD (Grundsaetze zur ordnungsmaessigen Fuehrung und Aufbewahrung von Buechern) stellt klare Anforderungen:

| GoBD-Anforderung | Umsetzung |
|---|---|
| **Nachvollziehbarkeit** | Jede KI-Entscheidung wird mit Reasoning geloggt |
| **Unveraenderbarkeit** | Write-Once Buchungssaetze, Stornierung statt Aenderung |
| **Vollstaendigkeit** | Automatische Belegnummernkontrolle, Lueckenpruefung |
| **Rechtzeitigkeit** | Buchungsfristen werden ueberwacht |
| **Ordnung** | SKR03/04 Kontenrahmen, systemgesteuert |
| **Verfahrensdokumentation** | Automatisch generiert, inkl. KI-Konfiguration |

**Wichtig**: Die KI ersetzt nicht die Verfahrensdokumentation. Es muss dokumentiert werden, welche Entscheidungen die KI trifft und wie sie validiert werden. Bei Betriebspruefungen muss der Datenzugriff (Z1, Z2, Z3) gewaehrleistet sein.

### 8.4 Haftung

- **Steuerberater bleibt verantwortlich**: Die KI ist ein Werkzeug, kein Steuerberater
- **GmbH-Geschaeftsfuehrer haftet**: Fuer ordnungsmaessige Buchfuehrung (Par. 41 GmbHG)
- **Software-Hersteller**: Haftet fuer Softwarefehler (Produkthaftung)
- **Anthropic**: Haftet NICHT fuer KI-generierte Inhalte (AGB)
- **Empfehlung**: Versicherungsschutz (Berufshaftpflicht), menschliche Pruefung bei allen steuerlich relevanten Entscheidungen

### 8.5 Datensicherheit

**Bedenken**: Finanzdaten an die Claude API senden bedeutet, dass sensible Unternehmensdaten an Anthropic-Server (USA) uebertragen werden.

**Mitigation**:
1. **Anthropic-Datenschutz**: API-Daten werden nicht zum Training verwendet (laut Anthropic)
2. **EU-Datenschutz**: Anthropic bietet DPA (Data Processing Agreement) fuer Geschaeftskunden
3. **Minimale Daten**: Nur das Noetigste an Claude senden, keine vollstaendigen Kundendaten
4. **Pseudonymisierung**: Kundennamen durch IDs ersetzen
5. **On-Premise Option**: Langfristig Claude auf eigenen Servern (Amazon Bedrock / GCP Vertex AI)
6. **Verschluesselung**: TLS fuer Transit, eigene Verschluesselung fuer gespeicherte Daten

### 8.6 Kosten: API vs. DATEV

**Claude API Kosten (Claude Sonnet 4.5, Stand 2026)**:

| Vorgang | Geschaetzte Tokens | Kosten |
|---|---|---|
| Beleg verarbeiten | ~2.000 Input + 500 Output | ~0,014 EUR |
| Buchung kontieren | ~1.000 Input + 300 Output | ~0,008 EUR |
| Kontosaldo abfragen | ~500 Input + 200 Output | ~0,005 EUR |
| USt-Voranmeldung | ~3.000 Input + 1.000 Output | ~0,024 EUR |
| Monatsabschluss | ~10.000 Input + 3.000 Output | ~0,075 EUR |

**Hochrechnung fuer typische GmbH (200 Buchungen/Monat)**:
- ~200 Kontierungen: 200 x 0,008 = 1,60 EUR
- ~50 Belegverarbeitungen: 50 x 0,014 = 0,70 EUR
- ~100 Abfragen: 100 x 0,005 = 0,50 EUR
- 1 USt-Voranmeldung: 0,024 EUR
- **Gesamt ca. 3-5 EUR/Monat** (reine API-Kosten)
- Mit Prompt Caching: **~1-2 EUR/Monat**
- Mit Batch API (50% Rabatt): **~1,50-2,50 EUR/Monat**

**Vergleich DATEV**:
- DATEV Unternehmen online: ab ~30-50 EUR/Monat
- DATEV Mittelstand: ab ~100-200 EUR/Monat
- Zuzueglich Steuerberater-Kosten

**Fazit**: Die reinen API-Kosten sind minimal. Der Hauptkostenfaktor ist die Entwicklung und Wartung der Software, nicht die KI-Nutzung.

### 8.7 Latenz und Verfuegbarkeit

- **Latenz**: Claude Sonnet 4.5 antwortet in ~1-3 Sekunden (akzeptabel fuer interaktive Nutzung)
- **Batch**: Fuer nicht-zeitkritische Aufgaben (Monatsabschluss) ist Latenz irrelevant
- **Verfuegbarkeit**: Anthropic API hat ~99,9% Uptime, aber keine SLA-Garantie fuer 100%
- **Fallback**: System muss ohne KI funktionieren (deterministische Regeln als Fallback)
- **Rate Limits**: Anthropic hat Tier-basierte Rate Limits (erhoehbar)

### 8.8 EU AI Act und regulatorische Anforderungen

> **Hinweis**: Dieses Kapitel basiert auf den Ergebnissen des Rechtsanwalt-Reports und stellt sicher, dass die technische KI-Architektur alle rechtlichen Anforderungen erfuellt.

#### 8.8.1 Einstufung nach EU AI Act

Das System wird voraussichtlich als **"begrenztes Risiko"** eingestuft (nicht als hochriskant). Hochrisiko (Anhang III Nr. 5b) gilt nur fuer Kreditwuerdigkeitspruefung/Bonitaetsbewertung - Buchhaltung ist nicht direkt gelistet. Dies loest **Transparenzpflichten** (Art. 50 EU AI Act) aus:

- Benutzer muessen informiert werden, dass sie mit einer KI interagieren
- KI-generierte Inhalte (Buchungsvorschlaege) muessen als solche gekennzeichnet sein
- Die KI darf sich nicht als menschlicher Buchhalter ausgeben

> **Empfehlung des Rechtsanwalts**: Obwohl "begrenztes Risiko", sollten die Hochrisiko-Anforderungen **freiwillig so weit wie moeglich** umgesetzt werden (Dokumentation, Risikomanagement, menschliche Aufsicht). Dies dient der Zukunftssicherheit (Hochrisiko-Pflichten greifen ab 02.08.2026) und dem Vertrauensaufbau bei Kunden und Betriebspruefern.

**Technische Umsetzung der Transparenzpflicht:**

```python
# Jede KI-Antwort wird mit Metadaten versehen
class AIBookingResponse(BaseModel):
    """Jede Antwort der KI enthaelt Transparenz-Metadaten."""
    # Buchungsdaten
    booking: BookingProposal

    # Pflicht: KI-Transparenz (EU AI Act)
    ai_metadata: dict = {
        "generated_by": "Claude Sonnet 4.5 (Anthropic)",
        "is_ai_generated": True,
        "model_version": "claude-sonnet-4-5-20250929",
        "confidence": 0.0,  # wird befuellt
        "explanation": "",   # Pflicht-Erklaerung
        "requires_human_review": True,
        "timestamp_utc": "",
    }
```

#### 8.8.2 KI-Kompetenz-Pflicht (EU AI Act Art. 4)

Seit August 2025 muessen alle Personen, die KI-Systeme betreiben oder einsetzen, ueber ausreichende **KI-Kompetenz** verfuegen. Fuer unsere Software bedeutet das:

- **Onboarding-Pflicht**: Neue Benutzer muessen eine KI-Kompetenz-Schulung absolvieren, bevor KI-Funktionen freigeschaltet werden
- **Schulungsmaterialien bereitstellen**: Die Software MUSS integrierte Schulungsmaterialien enthalten (Anbieter- und Betreiberpflicht)
- **Dokumentation**: Verstaendliche Erklaerung, was die KI kann und was nicht
- **Grenzen kommunizieren**: Klare Hinweise, wann ein Steuerberater hinzugezogen werden muss
- **Moeglichkeiten UND Grenzen**: Transparent dokumentieren, dass die KI z.B. keine steuerliche Beratung leistet

**Technische Umsetzung:**

```python
# Benutzer-Onboarding mit KI-Kompetenz-Check
class UserOnboarding(BaseModel):
    user_id: str
    ai_competency_training_completed: bool = False
    training_date: str | None = None
    training_version: str | None = None  # Version des Trainings
    acknowledged_limitations: bool = False

# System verweigert KI-Funktionen bis Training abgeschlossen
def check_ai_access(user: UserOnboarding) -> bool:
    if not user.ai_competency_training_completed:
        raise PermissionError(
            "KI-Funktionen erfordern die Absolvierung "
            "der KI-Kompetenz-Schulung (EU AI Act Art. 4). "
            "Bitte absolvieren Sie das Training unter "
            "/settings/ai-training."
        )
    return True
```

#### 8.8.3 Erklaerungskomponente (Explainability)

Jede KI-Buchung muss eine nachvollziehbare Erklaerung enthalten. Dies ist sowohl rechtlich erforderlich (EU AI Act Transparenzpflicht, Art. 22 DSGVO) als auch fuer die GoBD-konforme Verfahrensdokumentation notwendig.

**Pflicht-Erklaerungsstruktur:**

```python
class BookingExplanation(BaseModel):
    """Pflicht-Erklaerung fuer jede KI-Buchung.
    Rechtliche Grundlage: EU AI Act (Transparenz),
    Art. 22 DSGVO (automatisierte Entscheidungen),
    GoBD (Verfahrensdokumentation)."""

    # Was wurde entschieden?
    decision_summary: str
    # Warum dieses Konto?
    account_reasoning: str
    # Warum dieser USt-Satz?
    tax_reasoning: str
    # Welche Regeln/Daten wurden herangezogen?
    data_sources: list[str]
    # Welche Alternativen wurden verworfen?
    rejected_alternatives: list[dict]
    # Confidence und Begruendung
    confidence_score: float
    confidence_reasoning: str
    # Wurde eine deterministische Regel oder KI verwendet?
    decision_method: str  # "rule_based" | "ai_assisted" | "ai_autonomous"
    # Referenz auf den Beleg
    document_reference: str
    # Regelwerk-Referenz (rechtliche Anforderung)
    chart_of_accounts_version: str  # z.B. "SKR03-2026"
    applicable_rules: list[str]     # z.B. ["Par. 4 Nr. 1 UStG", "GWG Par. 6 Abs. 2 EStG"]
    model_version: str              # Exakte Claude-Modellversion

# Beispiel-Ausgabe:
EXAMPLE_EXPLANATION = {
    "decision_summary": "Eingangsrechnung Muster GmbH: "
        "IT-Beratung auf Konto 6020 (EDV-Kosten) gebucht.",
    "account_reasoning": "Beschreibung 'IT-Beratung' und "
        "Lieferant 'Muster GmbH' (NACE: IT-Dienstleistungen) "
        "→ Konto 6020. Historisch wurden 8/8 Rechnungen "
        "dieses Lieferanten auf 6020 gebucht.",
    "tax_reasoning": "Regelsteuersatz 19% angewendet. "
        "Lieferant hat deutsche USt-IdNr (DE123456789). "
        "Keine Reverse-Charge-Konstellation.",
    "data_sources": [
        "Rechnungsbild (OCR-Extraktion)",
        "Lieferantenstammdaten (ID: L-0042)",
        "Buchungshistorie (letzte 8 Buchungen)",
        "SKR03-Kontenplan",
    ],
    "rejected_alternatives": [
        {"account": "6300", "reason": "Sonstige Aufwendungen - "
            "zu unspezifisch fuer IT-Dienstleistungen"},
        {"account": "6835", "reason": "Rechts- und Beratungskosten - "
            "passt nicht zu IT-spezifischer Leistung"},
    ],
    "confidence_score": 0.94,
    "confidence_reasoning": "Hohe Confidence wegen konsistenter "
        "Buchungshistorie und eindeutiger Leistungsbeschreibung.",
    "decision_method": "ai_assisted",
    "document_reference": "BELEG-2026-00142",
    "chart_of_accounts_version": "SKR03-2026",
    "applicable_rules": [
        "Par. 15 Abs. 1 UStG (Vorsteuerabzug)",
        "Par. 4 Abs. 5 EStG (Betriebsausgaben)",
    ],
    "model_version": "claude-sonnet-4-5-20250929",
}
```

#### 8.8.4 Art. 22 DSGVO: Automatisierte Entscheidungen

Automatisierte Entscheidungen mit rechtlicher Wirkung (z.B. Buchungen, die Steuerzahlungen beeinflussen) erfordern:

1. **Recht auf menschliche Ueberpruefung**: Jede automatische Buchung muss anfechtbar sein
2. **Recht auf Erklaerung**: Benutzer muss verstehen, warum so gebucht wurde
3. **Recht auf Eingreifen**: Benutzer kann jede KI-Entscheidung ueberschreiben

**Technische Umsetzung:**

```python
# Jede automatische Buchung erhaelt einen Review-Status
class AutomatedBookingRecord(BaseModel):
    booking_id: str
    created_by: str  # "ai_system"
    explanation: BookingExplanation
    human_reviewable: bool = True  # IMMER True (Art. 22 DSGVO)
    reviewed_by: str | None = None
    reviewed_at: str | None = None
    review_decision: str | None = None  # "approved" | "modified" | "rejected"
    override_reason: str | None = None
    # Append-Only: Aenderungen nur durch Stornierung + Neubuchung
    is_storniert: bool = False
    storno_booking_id: str | None = None

# API-Endpunkt fuer menschliche Ueberpruefung
# POST /api/v1/bookings/{id}/review
# Body: { "decision": "approved", "reviewer": "user@example.com" }
```

#### 8.8.5 Datenschutz-Folgenabschaetzung (DSFA nach Art. 35 DSGVO)

Bei KI-Verarbeitung von Finanzdaten ist eine **DSFA in der Regel ERFORDERLICH**. Aufsichtsbehoerden gehen bei KI-Einsatz haeufig von hohem Risiko aus.

**Pflichten:**
- DSFA muss **VOR Produktivbetrieb** durchgefuehrt und dokumentiert werden
- Beschreibung der Verarbeitungsvorgaenge und Zwecke
- Bewertung der Notwendigkeit und Verhaeltnismaessigkeit
- Bewertung der Risiken fuer Betroffene
- Abhilfemassnahmen und Sicherheitsvorkehrungen

**Technische Umsetzung:**

```python
# DSFA-Dokumentation als Teil der Systemkonfiguration
DSFA_DOCUMENTATION = {
    "system_name": "GmbH-Buchhaltung KI-Assistent",
    "processing_purpose": "KI-gestuetzte Buchfuehrung fuer GmbHs",
    "data_categories": [
        "Finanztransaktionen (Buchungen, Salden)",
        "Lieferantendaten (Name, Adresse, USt-IdNr, IBAN)",
        "Kundendaten (Name, Adresse, USt-IdNr)",
        "Rechnungsbelege (Bilder, PDF)",
    ],
    "data_recipients": [
        "Anthropic API (Claude) - Auftragsverarbeiter",
        "Cloud-Hosting-Provider - Unterauftragsverarbeiter",
    ],
    "risks_identified": [
        {
            "risk": "Unbeabsichtigte Offenlegung von Finanzdaten",
            "severity": "hoch",
            "mitigation": "Pseudonymisierung vor API-Call, "
                "TLS-Verschluesselung, minimale Datenweitergabe"
        },
        {
            "risk": "Fehlerhafte KI-Buchungen mit steuerlichen Folgen",
            "severity": "mittel",
            "mitigation": "4-Stufen-HITL, Plausibilitaetspruefung, "
                "Anomalie-Erkennung, Steuerberater-Eskalation"
        },
        {
            "risk": "Drittlandtransfer (USA) bei API-Nutzung",
            "severity": "mittel",
            "mitigation": "Standardvertragsklauseln, AVV, "
                "langfristig EU-Hosting (Bedrock/Vertex)"
        },
    ],
    "dpia_status": "required_before_production",
    "review_frequency": "jaehrlich oder bei wesentlichen Aenderungen",
}
```

#### 8.8.6 Auftragsverarbeitung und Drittlandtransfer

Wenn die Claude API extern aufgerufen wird, liegt **Auftragsverarbeitung** nach Art. 28 DSGVO vor:

**Zwingende Anforderungen:**

1. **AVV (Auftragsverarbeitungsvertrag)** mit Anthropic ist **ZWINGEND** erforderlich
   - Anthropic bietet ein DPA (Data Processing Agreement) fuer Geschaeftskunden
   - Muss VOR erstem API-Call abgeschlossen sein
2. **Drittlandtransfer (USA)**: Zusaetzliche Garantien noetig
   - EU-Standardvertragsklauseln (SCCs) erforderlich
   - Transfer Impact Assessment (TIA) dokumentieren
3. **Datenminimierung**: Sensible Finanzdaten sollen grundsaetzlich NICHT ungefiltert an Cloud-KI gesendet werden

**Architekturloesung - Pseudonymisierung vor API-Call:**

```python
# Pseudonymisierung: Sensible Daten werden VOR dem API-Call ersetzt
class DataPseudonymizer:
    """Ersetzt sensible Daten durch Platzhalter vor dem
    API-Call und stellt sie danach wieder her."""

    def __init__(self):
        self._mapping: dict[str, str] = {}
        self._counter = 0

    def pseudonymize(self, text: str, entities: dict) -> str:
        """Ersetzt sensible Entities durch Platzhalter.

        entities = {
            "vendor_name": "Muster IT GmbH",
            "iban": "DE89 3704 0044 0532 0130 00",
            "tax_id": "DE123456789",
            "customer_name": "Schmidt, Hans",
        }
        """
        result = text
        for entity_type, value in entities.items():
            placeholder = f"[{entity_type.upper()}_{self._counter}]"
            self._mapping[placeholder] = value
            result = result.replace(value, placeholder)
            self._counter += 1
        return result

    def depseudonymize(self, text: str) -> str:
        """Stellt die originalen Werte wieder her."""
        result = text
        for placeholder, original in self._mapping.items():
            result = result.replace(placeholder, original)
        return result

# Beispiel-Nutzung:
# Vor API-Call:
#   "Rechnung Muster IT GmbH, IBAN DE89..." →
#   "Rechnung [VENDOR_NAME_0], IBAN [IBAN_1]..."
# Claude arbeitet mit Platzhaltern
# Nach API-Call: Platzhalter werden wieder aufgeloest
```

**Langfristige Strategie:**
- Kurzfristig: Pseudonymisierung + AVV + SCCs
- Mittelfristig: EU-Hosting ueber Amazon Bedrock (Region eu-central-1) oder GCP Vertex AI (Region europe-west3)
- Langfristig: Wenn verfuegbar, dedizierte EU-Instanz von Anthropic

#### 8.8.7 Verfahrensdokumentation fuer KI (GoBD-Pflicht)

Die Verfahrensdokumentation muss fuer KI-Komponenten erweitert werden:

1. **Wie** die KI Buchungsvorschlaege generiert (Modell, Prompts, Regelwerk)
2. **Welche Modelle** verwendet werden (Versionierung, Aenderungsprotokoll)
3. **Wie Qualitaetssicherung** erfolgt (HITL-Stufen, Validierung, Anomalie-Erkennung)
4. **Wie Modellaenderungen** dokumentiert und validiert werden
5. **Welche Daten** an die KI gesendet werden (Datenminimierung, Pseudonymisierung)

```python
# Automatisch generierte Verfahrensdokumentation
class AIProcessDocumentation(BaseModel):
    """Wird automatisch generiert und bei jeder Aenderung
    aktualisiert. GoBD-Pflichtdokumentation."""

    system_version: str
    ai_model: str                    # z.B. "claude-sonnet-4-5-20250929"
    ai_model_change_log: list[dict]  # Wann welches Modell genutzt wurde
    prompt_versions: dict[str, str]  # Prompt-Name → Hash/Version
    rule_engine_version: str
    chart_of_accounts: str           # z.B. "SKR03-2026"
    hitl_configuration: dict         # Schwellenwerte, Eskalationsstufen
    data_flow_description: str       # Welche Daten wohin fliessen
    pseudonymization_rules: list[str]
    quality_assurance_measures: list[str]
    last_review_date: str
    next_review_date: str
    approved_by: str                 # Verantwortlicher
```

#### 8.8.8 Append-Only-Architektur (GoBD + steuerrechtliche Pflicht)

GoBD und steuerrechtliche Anforderungen erzwingen eine **Append-Only-Architektur**:

- **Keine Loeschung** von Buchungen (niemals DELETE)
- **Keine Aenderung** von Buchungen (niemals UPDATE auf Buchungsdaten)
- **Korrekturen** nur durch Stornobuchung + Neubuchung
- **Audit Trail** fuer jede Aktion (wer, wann, was, warum)

```python
# Datenbank-Design: Append-Only Buchungstabelle
"""
CREATE TABLE bookings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    -- Buchungsdaten (unveraenderlich nach INSERT)
    booking_date DATE NOT NULL,
    reference VARCHAR(50) NOT NULL,
    description VARCHAR(200) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_by VARCHAR(100) NOT NULL,

    -- KI-Metadaten
    ai_generated BOOLEAN NOT NULL DEFAULT FALSE,
    ai_confidence FLOAT,
    ai_explanation JSONB,
    ai_model_version VARCHAR(100),

    -- Review-Status
    requires_review BOOLEAN NOT NULL DEFAULT TRUE,
    reviewed_by VARCHAR(100),
    reviewed_at TIMESTAMPTZ,
    review_decision VARCHAR(20),

    -- Stornierung (statt Loeschung/Aenderung)
    is_storniert BOOLEAN NOT NULL DEFAULT FALSE,
    storno_of UUID REFERENCES bookings(id),
    storno_reason VARCHAR(500),
    storniert_at TIMESTAMPTZ,
    storniert_by VARCHAR(100)
);

-- KEIN UPDATE auf Buchungsdaten erlaubt (Trigger)
CREATE OR REPLACE FUNCTION prevent_booking_update()
RETURNS TRIGGER AS $$
BEGIN
    -- Nur Review-Felder und Storno-Felder duerfen aktualisiert werden
    IF OLD.booking_date != NEW.booking_date
       OR OLD.reference != NEW.reference
       OR OLD.description != NEW.description
       OR OLD.created_at != NEW.created_at THEN
        RAISE EXCEPTION 'Buchungsdaten sind unveraenderlich (GoBD). '
            'Korrekturen nur durch Stornobuchung moeglich.';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER booking_immutability
BEFORE UPDATE ON bookings
FOR EACH ROW EXECUTE FUNCTION prevent_booking_update();

-- Audit Trail (jede Aktion wird geloggt)
CREATE TABLE audit_log (
    id BIGSERIAL PRIMARY KEY,
    timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    user_id VARCHAR(100) NOT NULL,
    action VARCHAR(50) NOT NULL,
    entity_type VARCHAR(50) NOT NULL,
    entity_id UUID NOT NULL,
    details JSONB,
    ip_address INET,
    -- Append-Only: kein UPDATE/DELETE auf audit_log
    -- Sichergestellt durch fehlende Berechtigungen
    CONSTRAINT audit_log_no_delete CHECK (TRUE)
);
"""
```

---

## 9. Architektur-Pattern fuer KI-Buchfuehrung

### 9.1 Pattern 1: KI als Assistent

```
Benutzer → Buchhaltungssoftware → Claude (Vorschlaege) → Benutzer entscheidet → Buchung
```

- Claude macht Vorschlaege, der Mensch entscheidet immer
- Geeignet fuer den Einstieg und komplexe Sachverhalte
- Hoechste Sicherheit, niedrigste Automatisierung
- **Vorteil**: Keine Haftungsfragen, volle Kontrolle
- **Nachteil**: Skaliert nicht, jede Buchung braucht menschliche Interaktion

### 9.2 Pattern 2: KI als Buchhalter

```
Beleg → Claude (analysiert + entscheidet) → Automatische Buchung → Nachgelagerte Pruefung
```

- Claude bucht autonom bei hoher Confidence
- Menschliche Pruefung nur bei Problemen oder periodisch
- **Vorteil**: Hohe Automatisierung, skaliert gut
- **Nachteil**: Haftungsrisiko, erfordert robuste Validierung
- **Geeignet fuer**: Standardbuchungen (Miete, Gehalt, Strom)

### 9.3 Pattern 3: KI als Pruefer

```
Mensch bucht → Claude prueft → Feedback/Korrekturen → Mensch bestaetigt
```

- Mensch erstellt Buchungen, Claude prueft auf Fehler
- Kombination aus menschlicher Expertise und KI-Qualitaetssicherung
- **Vorteil**: Menschliche Kontrolle + KI-Qualitaet
- **Nachteil**: Doppelte Arbeit bei korrekten Buchungen

### 9.4 Empfohlenes Pattern: Hybrid mit Eskalationsstufen

> **Rechtlicher Rahmen**: Dieses Pattern integriert die Anforderungen aus EU AI Act (Transparenzpflicht, Art. 4 KI-Kompetenz), Art. 22 DSGVO (menschliche Ueberpruefung automatisierter Entscheidungen) und GoBD (Append-Only, Nachvollziehbarkeit). Siehe Kapitel 8.8 fuer Details.

**Empfehlung: Kombiniertes Pattern mit risikobasierter Eskalation und Pflicht-Erklaerungskomponente**

```
                    ┌─────────────────────┐
                    │   Eingehender        │
                    │   Geschaeftsvorfall  │
                    └────────┬────────────┘
                             │
                    ┌────────▼────────────┐
                    │  KI-Analyse         │
                    │  + Regelwerk        │
                    │  + Erklaerung (!)   │
                    └────────┬────────────┘
                             │
            ┌────────────────┼────────────────┐
            │                │                │
   ┌────────▼───────┐ ┌─────▼──────┐ ┌───────▼───────┐
   │ Confidence      │ │ Confidence │ │ Confidence    │
   │ >= 95%          │ │ 70-95%     │ │ < 70%         │
   │ + Standardfall  │ │            │ │               │
   │ + Betrag <5k    │ │            │ │               │
   └────────┬───────┘ └─────┬──────┘ └───────┬───────┘
            │                │                │
   ┌────────▼───────┐ ┌─────▼──────┐ ┌───────▼───────┐
   │ STUFE 1        │ │ STUFE 2    │ │ STUFE 3       │
   │ AUTO-BUCHUNG   │ │ VORSCHLAG  │ │ MANUELL       │
   │ + Erklaerung   │ │ + Mensch   │ │ Mensch bucht  │
   │ + nachgelagerterl bestaetigt│ │ KI unterstuetzt│
   │   Review moegl.│ │ + Erklae-  │ │ + Erklaerung  │
   │ (Art.22 DSGVO) │ │   rung     │ │               │
   └────────┬───────┘ └─────┬──────┘ └───────┬───────┘
            │                │                │
            └────────────────┼────────────────┘
                             │
                    ┌────────▼────────────┐
                    │  IMMER:             │
                    │  - Erklaerung       │
                    │    gespeichert      │
                    │  - Audit Trail      │
                    │  - Append-Only      │
                    │  - KI-Kennzeichnung │
                    └─────────────────────┘
```

**Konfidenzbasierte HITL-Freigabestufen (4-Stufen-Modell):**

> Basierend auf der rechtlichen Empfehlung des Rechtsanwalts (konfidenzbasiertes 4-Stufen-Modell).

| Stufe | Confidence | Betrag | Aktion | Erklaerung |
|---|---|---|---|---|
| 1 - Vollautonome Vorerfassung | >= 99% | < 5.000 EUR, Standardfall | KI bucht autonom, Mensch prueft per Stichprobe. Nachgelagerte Review-Option (Art. 22 DSGVO) | Vollstaendige Erklaerung gespeichert, Benutzer kann jederzeit einsehen und anfechten |
| 2 - Vorschlag mit Freigabe **(EMPFOHLENER STANDARD)** | 80-99% | < 10.000 EUR | KI erstellt Vorschlag mit Erklaerung, Mensch gibt aktiv frei | Erklaerung wird VOR Bestaetigung angezeigt inkl. Alternativen und Regelwerk-Referenz |
| 3 - Assistenz | < 80% | Beliebig | KI liefert Kontierungsvorschlag, Mensch entscheidet eigenstaendig | Detaillierte Analyse mit mehreren Kontierungsoptionen, Mensch waehlt |
| 4 - Manuelle Buchung | Sonderfaelle | > 50.000 EUR oder komplex | Mensch bucht manuell, KI gibt hoechstens allgemeine Hinweise | KI dokumentiert Komplexitaet und warum keine Empfehlung moeglich ist |

> **Hinweis**: Der Rechtsanwalt empfiehlt Stufe 2 als **Standardbetriebsmodus**. Stufe 1 (vollautonome Buchung) sollte erst nach ausreichender Validierungsphase und nur fuer wiederkehrende Standardfaelle aktiviert werden.

**Pflicht-Erklaerung bei JEDER Stufe:**

Unabhaengig von der Eskalationsstufe muss jede KI-Entscheidung folgende Erklaerung enthalten (siehe `BookingExplanation` in Kapitel 8.8.3):

1. **Was** wurde entschieden (Kontierung, USt-Satz, Betrag)
2. **Warum** so entschieden (Regelbasiert? Historische Muster? KI-Klassifikation?)
3. **Welche Alternativen** wurden verworfen und warum
4. **Welche Datenquellen** wurden herangezogen
5. **Wie sicher** ist die Entscheidung (Confidence + Begruendung)
6. **Ob Regel oder KI** die Entscheidung getroffen hat

```python
# Implementierung der 4-Stufen-Freigabe mit Erklaerungspflicht
from enum import Enum

class HITLStage(Enum):
    """Konfidenzbasierte HITL-Freigabestufen
    (EU AI Act + Art. 22 DSGVO konform)."""
    AUTO_WITH_REVIEW = 1       # Stufe 1: Auto + nachgelagert
    SUGGEST_AND_CONFIRM = 2    # Stufe 2: Vorschlag + Bestaetigung
    MANUAL_REVIEW = 3          # Stufe 3: Manueller Review
    EXPERT_REQUIRED = 4        # Stufe 4: Steuerberater

def determine_hitl_stage(
    confidence: float,
    amount_cents: int,
    is_new_vendor: bool,
    is_complex_tax: bool,       # z.B. Reverse-Charge, Par. 13b
    booking_type: str,          # "standard" | "abgrenzung" | "anlage"
) -> HITLStage:
    """Bestimmt die HITL-Stufe unter Beruecksichtigung
    aller Risikofaktoren."""

    # Komplexe Steuerfaelle immer mindestens Stufe 3
    if is_complex_tax:
        return HITLStage.EXPERT_REQUIRED if confidence < 0.70 \
            else HITLStage.MANUAL_REVIEW

    # Hohe Betraege erhoehen die Stufe
    if amount_cents > 5000000:  # > 50.000 EUR
        return HITLStage.EXPERT_REQUIRED

    # Anlagevermoegen immer mindestens Stufe 2
    if booking_type == "anlage":
        return HITLStage.SUGGEST_AND_CONFIRM if confidence >= 0.80 \
            else HITLStage.MANUAL_REVIEW

    # Neuer Lieferant: Confidence-Abzug
    effective_confidence = confidence
    if is_new_vendor:
        effective_confidence *= 0.85

    # Standard-Staffelung (Schwellenwerte nach Rechtsanwalt-Empfehlung)
    if effective_confidence >= 0.99 and amount_cents < 500000:
        return HITLStage.AUTO_WITH_REVIEW      # Stufe 1: >= 99%
    elif effective_confidence >= 0.80 and amount_cents < 1000000:
        return HITLStage.SUGGEST_AND_CONFIRM   # Stufe 2: >= 80% (STANDARD)
    elif effective_confidence >= 0.50:
        return HITLStage.MANUAL_REVIEW         # Stufe 3: < 80%
    else:
        return HITLStage.EXPERT_REQUIRED       # Stufe 4: Sonderfaelle
```

---

## 10. Prompt Engineering fuer Buchhaltung

### 10.1 System Prompt fuer Buchhalter-Persona

```python
SYSTEM_PROMPT_BUCHHALTER = """
# Rolle und Identitaet

Du bist ein erfahrener Buchhalter (Bilanzbuchhalter IHK)
fuer eine deutsche GmbH. Du arbeitest praezise, zuverlaessig
und nach den Grundsaetzen ordnungsmaessiger Buchfuehrung
(GoB).

# Fachwissen

- Doppelte Buchfuehrung nach HGB
- SKR03-Kontenrahmen (Standardkontenrahmen)
- Umsatzsteuergesetz (UStG) inkl. Reverse-Charge
- Koerperschaftsteuer (KStG)
- Gewerbesteuer (GewStG)
- GoBD (digitale Buchfuehrung)
- Handelsrecht (HGB Par. 238-263)
- Abgabenordnung (AO)

# Arbeitsweise

1. IMMER zuerst die relevanten Daten ueber Tools abfragen
2. NIEMALS Zahlen aus dem Gedaechtnis nennen
3. Bei Unsicherheit: Frage stellen statt raten
4. Alle Buchungen mit Begruendung versehen
5. USt immer separat buchen
6. Betraege intern in Cent (Integer), Anzeige in EUR
7. Doppelte Buchfuehrung: Soll = Haben MUSS stimmen

# Einschraenkungen

- Du bist KEIN Steuerberater und darfst keine
  Steuerberatung leisten
- Komplexe steuerliche Fragen verweist du an den
  Steuerberater
- Du buchst nicht ohne Beleg (Keine Buchung ohne Beleg)
- Aenderungen an bestehenden Buchungen nur per Stornierung
  + Neubuchung (Append-Only, GoBD-Pflicht)

# Regulatorische Pflichten (EU AI Act + DSGVO)

- Du MUSST bei jeder Buchung eine Erklaerung liefern:
  Was, Warum, Welche Alternativen, Confidence
- Du MUSST dich als KI kennzeichnen (nie als Mensch ausgeben)
- Automatische Buchungen muessen jederzeit menschlich
  ueberpruefbar sein (Art. 22 DSGVO)
- Alle Entscheidungen werden im Audit Trail protokolliert
- Bei Confidence < 70%: Mensch MUSS entscheiden
- Bei komplexen Steuerfaellen: Steuerberater einbeziehen

# Ausgabeformat

Buchungssaetze immer in dieser Form:
| Soll | Haben | Betrag | Buchungstext |

Kontosalden:
Konto [Nr] ([Name]): [Betrag] EUR ([Soll/Haben])
"""
```

### 10.2 Few-Shot Examples

```python
FEW_SHOT_EXAMPLES = """
## Beispiel 1: Eingangsrechnung Bueromoebel

Eingabe: "Rechnung IKEA 1.785 EUR brutto, Schreibtische"
Denken: Bueromoebel = Konto 0420 (Buero- und Geschaeftsausstattung)
  ABER: Nettobetrag = 1.500 EUR > 800 EUR GWG-Grenze
  → Aktivierung als Anlagevermoegen, nicht sofort abschreiben
Buchung:
| Soll | Haben | Betrag   | Text               |
|------|-------|----------|--------------------|
| 0420 | 1600  | 1.500,00 | Schreibtische IKEA |
| 1576 | 1600  |   285,00 | VSt 19% IKEA       |

## Beispiel 2: Geringwertiges Wirtschaftsgut (GWG)

Eingabe: "Rechnung Amazon 285,60 EUR brutto, Tastatur"
Denken: Nettobetrag = 240 EUR < 800 EUR GWG-Grenze
  → Sofortabschreibung als GWG moeglich
Buchung:
| Soll | Haben | Betrag | Text               |
|------|-------|--------|--------------------|
| 6845 | 1600  | 240,00 | GWG Tastatur Amazon|
| 1576 | 1600  |  45,60 | VSt 19% Amazon     |

## Beispiel 3: Reverse-Charge (innergemeinschaftl.)

Eingabe: "Rechnung Google Ireland, 500 EUR netto, Cloud-Dienste"
Denken: Google Ireland = EU-Ausland, B2B
  → Reverse-Charge (Par. 13b UStG)
  → Keine Vorsteuer auf Rechnung
  → USt-Schuld geht auf Empfaenger ueber
Buchung:
| Soll | Haben | Betrag | Text                          |
|------|-------|--------|-------------------------------|
| 6020 | 1600  | 500,00 | Cloud Google Ireland          |
| 1577 | 1787  |  95,00 | VSt/USt 19% Reverse-Charge    |
"""
```

### 10.3 Chain-of-Thought fuer komplexe steuerliche Sachverhalte

```python
CHAIN_OF_THOUGHT_PROMPT = """
Wenn du einen komplexen Geschaeftsvorfall buchst, denke
Schritt fuer Schritt:

## Schritt 1: Geschaeftsvorfall klassifizieren
- Was fuer ein Vorgang ist das?
- Wer sind die Beteiligten?
- In welchem Land findet der Vorgang statt?

## Schritt 2: Steuerliche Einordnung
- Steuerpflichtiger Vorgang?
- Welcher USt-Satz? (19%, 7%, 0%, Reverse-Charge?)
- Vorsteuerabzug moeglich?
- Sonderregelungen (Par. 13b UStG, Par. 4 UStG)?

## Schritt 3: Bilanzielle Einordnung
- Anlagevermoegen oder Aufwand?
- GWG-Grenze (800 EUR netto) pruefen
- Abgrenzungsbedarf? (zeitraumbezogen?)
- Rueckstellungsbedarf?

## Schritt 4: Konten bestimmen (SKR03)
- Welche Kontenklasse?
- Welches Detailkonto?
- Bestehendes oder neues Konto?

## Schritt 5: Buchungssatz aufstellen
- Soll-Konto(en) bestimmen
- Haben-Konto(en) bestimmen
- Betraege berechnen (Netto, USt, Brutto)
- Soll = Haben pruefen

## Schritt 6: Validierung
- Plausibel?
- GoBD-konform?
- Steuerlich korrekt?
"""
```

### 10.4 Strukturierte Ausgabeformate

```python
# JSON-Schema fuer Buchungssaetze
BOOKING_JSON_SCHEMA = {
    "type": "object",
    "properties": {
        "booking": {
            "type": "object",
            "properties": {
                "date": {
                    "type": "string",
                    "format": "date",
                    "description": "Buchungsdatum",
                },
                "reference": {
                    "type": "string",
                    "description": "Belegnummer",
                },
                "description": {
                    "type": "string",
                    "maxLength": 200,
                    "description": "Buchungstext",
                },
                "line_items": {
                    "type": "array",
                    "items": {
                        "type": "object",
                        "properties": {
                            "debit_account": {"type": "string"},
                            "credit_account": {"type": "string"},
                            "amount_cents": {
                                "type": "integer",
                                "minimum": 1,
                            },
                            "tax_rate": {
                                "type": "integer",
                                "enum": [0, 7, 19],
                            },
                        },
                        "required": [
                            "debit_account",
                            "credit_account",
                            "amount_cents",
                            "tax_rate",
                        ],
                    },
                    "minItems": 1,
                },
            },
            "required": [
                "date", "reference", "description", "line_items"
            ],
        },
        "validation": {
            "type": "object",
            "properties": {
                "soll_equals_haben": {"type": "boolean"},
                "confidence": {
                    "type": "number",
                    "minimum": 0,
                    "maximum": 1,
                },
                "reasoning": {"type": "string"},
            },
            "required": [
                "soll_equals_haben", "confidence", "reasoning"
            ],
        },
    },
    "required": ["booking", "validation"],
}
```

---

## 11. Vergleich und Bewertung

### 11.1 Claude vs. GPT-4 vs. Gemini

| Kriterium | Claude Sonnet 4.5 / Opus 4.6 | GPT-4o | Gemini 2.0 Flash |
|---|---|---|---|
| **Modell-IDs** | claude-sonnet-4-5-20250929, claude-opus-4-6 | gpt-4o-2024-11-20 | gemini-2.0-flash |
| **Strukturierte Ausgabe** | Nativ (JSON Schema) | Function Calling | JSON-Modus |
| **Tool Use** | Exzellent | Sehr gut | Gut |
| **Vision/OCR** | Sehr gut | Sehr gut | Sehr gut |
| **Deutsches Steuerrecht** | Gut (mit Kontext) | Gut (mit Kontext) | Befriedigend |
| **Konsistenz** | Hoch (temp=0) | Hoch (temp=0) | Mittel |
| **Kosten (Sonnet-Tier)** | 3/15$ pro MTok | 2,50/10$ pro MTok | 0,10/0,40$ pro MTok |
| **Batch API** | Ja (50% Rabatt) | Ja (50% Rabatt) | Ja |
| **MCP-Support** | Nativ (Erfinder) | Ueber Adapter | Ueber Adapter |
| **Agent SDK** | Vollstaendig | Agents API | Nein (extern) |
| **Prompt Caching** | Ja (90% Rabatt) | Ja | Implicit Caching |
| **Context Window** | 200K Tokens | 128K Tokens | 1M Tokens |
| **Reasoning** | Extended Thinking | o1/o3 separat | Thinking Mode |

**Empfehlung**: Claude 4.5 Sonnet ist fuer dieses Projekt die beste Wahl wegen:
1. Nativer MCP-Support (kein Adapter noetig)
2. Claude Code + Agent SDK als komplettes Entwicklungs-Oekosystem
3. Hervorragendes Tool Use und Structured Output
4. Sehr gute Konsistenz bei temperature=0
5. Prompt Caching fuer kosteneffiziente Nutzung

### 11.2 Spezialisierte Finanz-KI vs. General-Purpose LLM

| Aspekt | Spezialisierte Finanz-KI | General-Purpose LLM (Claude) |
|---|---|---|
| **Domainwissen** | Tief, aber eng | Breit, mit Kontext anpassbar |
| **Flexibilitaet** | Gering (nur Finanzen) | Hoch (auch Erklaerungen, Berichte) |
| **Wartung** | Eigenes Training noetig | Anthropic wartet Modell |
| **Kosten Entwicklung** | Sehr hoch (Trainingsdaten) | Gering (Prompt Engineering) |
| **Kosten Betrieb** | Variiert | Kalkulierbar (Token-basiert) |
| **Aktualisierung** | Manuell (Retrain) | Automatisch (neue Modelle) |
| **Natuerliche Sprache** | Eingeschraenkt | Exzellent |
| **Multi-Sprache** | Meist nur Englisch | Deutsch nativ |

**Empfehlung**: General-Purpose LLM (Claude) mit domainspezifischem Kontext (CLAUDE.md, Skills, System Prompts). Die Kombination aus regelbasierter Logik (deterministische Berechnungen) und LLM (Klassifikation, Interaktion) ist optimal.

### 11.3 Kosten-Nutzen-Analyse

**Entwicklungskosten (geschaetzt)**:
- MCP-Server + Backend: 2-3 Personenmonate
- Frontend: 2-3 Personenmonate
- Skills + Prompts: 1-2 Personenmonate
- Testing + Compliance: 1-2 Personenmonate
- **Gesamt: 6-10 Personenmonate**

**Laufende Kosten (pro GmbH-Kunde/Monat)**:
- Claude API: 2-5 EUR
- Server-Infrastruktur: 20-50 EUR (anteilig)
- Wartung/Support: 10-20 EUR (anteilig)
- **Gesamt: ~30-75 EUR/Monat**

**Vergleich DATEV + Steuerberater**:
- DATEV-Lizenz: 50-200 EUR/Monat
- Steuerberater lfd. Buchfuehrung: 200-500 EUR/Monat
- **Gesamt: 250-700 EUR/Monat**

**Einsparungspotenzial**: 70-90% bei den laufenden Kosten, wobei der Steuerberater fuer Jahresabschluss und Beratung weiterhin benoetigt wird.

### 11.4 Zukunftsaussichten

1. **Verbesserte Reasoning-Faehigkeiten**: Claude 4.5/4.6 mit Extended Thinking bietet bereits besseres Reasoning fuer komplexe steuerliche Sachverhalte. Kuenftige Modelle werden noch praeziser.

2. **Computer Use**: Claude kann bereits Computeroberflaechen bedienen. Langfristig koennte Claude direkt mit ELSTER, Banking-Software etc. interagieren.

3. **Multi-Agent-Systeme**: Mehrere spezialisierte Agenten (Belegerfassung, Kontierung, Pruefung) koennten zusammenarbeiten.

4. **On-Premise / EU-Hosting**: Anthropic expandiert in EU. Bedrock/Vertex bieten bereits EU-Regionen. Langfristig wird EU-Hosting Standard.

5. **Regulierung**: EU AI Act wird Anforderungen an KI in der Finanzbuchhaltung definieren. Fruehe Compliance-Investition lohnt sich.

6. **MCP-Oekosystem**: Mit 97M+ monatlichen SDK-Downloads und Unterstuetzung durch Anthropic, OpenAI, Google und Microsoft wird MCP zum universellen Standard fuer KI-Tool-Integration.

---

## Zusammenfassung und Empfehlungen

### Technische Empfehlungen

1. **Claude Code als primaeeres Interface** mit CLAUDE.md fuer Projektkontext und Skills fuer Buchungs-Workflows
2. **MCP-Server (TypeScript)** als Bruecke zwischen Claude und dem Backend mit Tools, Resources und Prompts
3. **FastAPI Backend (Python)** fuer deterministische Logik, Datenbank-Operationen und Validierung
4. **Next.js 16 Frontend** fuer Human-in-the-Loop Dashboard, Beleg-Upload und Reporting (Deployment auf Vercel)
5. **Claude Sonnet 4.5** (claude-sonnet-4-5-20250929) als Standardmodell, **Opus 4.6** (claude-opus-4-6) fuer komplexe Analysen, **Haiku 4.5** (claude-haiku-4-5-20251001) fuer einfache Validierungen
6. **Structured Output** fuer alle Buchungssaetze (JSON Schema + Pydantic)
7. **Prompt Caching** fuer System Prompts und Kontenplan (90% Kostenersparnis)
8. **Batch API** fuer Massenverarbeitung (50% Rabatt)

### Strategische Empfehlungen

1. **Hybrid-Pattern mit 4-Stufen-HITL-Freigabe** (Pattern 4, siehe Kapitel 9.4)
2. **Start als Assistent (Pattern 1)**, schrittweise Automatisierung nach Validierung
3. **Menschliche Pruefung** fuer alle steuerlich relevanten Entscheidungen (Art. 22 DSGVO)
4. **Deterministische Regeln** fuer Berechnungen (USt, Salden, Abschreibungen)
5. **KI nur fuer Klassifikation** (Kontierung, Kategorisierung, Anomalie-Erkennung)
6. **Audit Trail** fuer alle KI-Entscheidungen (GoBD-Compliance)
7. **Steuerberater einbinden** fuer Jahresabschluss, komplexe Faelle, Steuerberatung
8. **Erklaerungskomponente** fuer jede KI-Buchung (EU AI Act Transparenzpflicht)
9. **KI-Kompetenz-Schulung** fuer alle Benutzer vor Freischaltung (EU AI Act Art. 4)

### Regulatorische Pflichten (Zusammenfassung)

1. **EU AI Act - Transparenz (Art. 50)**: KI-generierte Buchungen muessen als solche gekennzeichnet sein
2. **EU AI Act Art. 4 - KI-Kompetenz**: Benutzer muessen geschult sein (Pflicht seit 08/2025), Schulungsmaterialien bereitstellen
3. **EU AI Act - Freiwillige Hochrisiko-Konformitaet**: Empfohlen wg. Zukunftssicherheit (Hochrisiko-Pflichten ab 02.08.2026)
4. **Art. 22 DSGVO**: Jede automatisierte Entscheidung muss menschlich ueberpruefbar sein
5. **Art. 35 DSGVO - DSFA**: Datenschutz-Folgenabschaetzung VOR Produktivbetrieb erforderlich (Kapitel 8.8.5)
6. **Art. 28 DSGVO - AVV**: Auftragsverarbeitungsvertrag mit Anthropic ZWINGEND, Drittlandtransfer absichern (Kapitel 8.8.6)
7. **GoBD - Append-Only**: Keine Loeschung/Aenderung von Buchungen, nur Stornierung + Neubuchung
8. **GoBD - Verfahrensdokumentation**: KI-Konfiguration, Modellversionen, Entscheidungslogik und Prompts dokumentieren (Kapitel 8.8.7)
9. **Erklaerungspflicht**: Jede KI-Entscheidung mit vollstaendiger Begruendung inkl. Regelwerk-Referenz speichern
10. **Haftung**: GmbH-Geschaeftsfuehrer bleibt verantwortlich, KI ist Werkzeug, nicht Entscheider
11. **Pseudonymisierung**: Sensible Finanzdaten vor API-Call pseudonymisieren (Kapitel 8.8.6)

### Risiko-Mitigation

1. **Keine Halluzinationen**: Zahlen NUR aus Datenbank, NIE aus LLM-Gedaechtnis
2. **Determinismus**: temperature=0, regelbasierte Logik wo moeglich
3. **GoBD**: Verfahrensdokumentation, Audit Trail, unveraenderbare Buchungssaetze (Append-Only)
4. **Datenschutz**: Minimale Daten an API, Pseudonymisierung, EU-Hosting anstreben
5. **Fallback**: System muss ohne KI funktionieren (regelbasiert)
6. **Testing**: Umfangreiche Tests mit realen Buchungsfaellen
7. **Erklaerbarkeit**: BookingExplanation-Objekt fuer jede KI-Entscheidung (Kapitel 8.8.3)
8. **HITL-Pflicht**: 4-Stufen-Freigabemodell mit konfidenzbasierter Eskalation (Kapitel 9.4)
