# Rechtliche Anforderungen an eine KI-gestuetzte Buchhaltungssoftware fuer GmbHs in Deutschland

## Umfassender Rechtsreport

**Stand:** Februar 2026
**Zweck:** Analyse aller rechtlichen Anforderungen an die Entwicklung einer KI-gestuetzten Buchhaltungssoftware als Alternative zu DATEV fuer GmbHs in Deutschland

---

## Inhaltsverzeichnis

1. [GoBD-Anforderungen](#1-gobd-anforderungen)
2. [HGB-Anforderungen](#2-hgb-anforderungen)
3. [Abgabenordnung](#3-abgabenordnung)
4. [Datenschutz (DSGVO)](#4-datenschutz-dsgvo)
5. [Zertifizierungen und Pruefungen](#5-zertifizierungen-und-pruefungen)
6. [Haftungsrisiken](#6-haftungsrisiken)
7. [KI-spezifische rechtliche Fragen](#7-ki-spezifische-rechtliche-fragen)
8. [ELSTER und Finanzverwaltung](#8-elster-und-finanzverwaltung)
9. [Zusammenfassung und Handlungsempfehlungen](#9-zusammenfassung-und-handlungsempfehlungen)

---

## 1. GoBD-Anforderungen

### 1.1 Ueberblick und Rechtsgrundlage

Die **Grundsaetze zur ordnungsgemaessen Fuehrung und Aufbewahrung von Buechern, Aufzeichnungen und Unterlagen in elektronischer Form sowie zum Datenzugriff (GoBD)** bilden das zentrale Regelwerk fuer elektronische Buchfuehrungssysteme in Deutschland. Die aktuelle Fassung wurde am **14. Juli 2025** durch ein BMF-Schreiben aktualisiert und gilt unmittelbar ohne Uebergangsfrist.

Die GoBD konkretisieren die Anforderungen aus:
- **HGB** (insbesondere ss 238, 239, 257 HGB)
- **AO** (insbesondere ss 145-147 AO)
- **UStG** (insbesondere ss 14, 14a UStG)

### 1.2 Grundprinzipien

#### 1.2.1 Nachvollziehbarkeit und Nachpruefbarkeit (Rz. 28-35 GoBD)

Jeder Geschaeftsvorfall muss von der Entstehung ueber die Erfassung bis zur Speicherung und Auswertung **lueckenlos nachvollziehbar** sein. Dies bedeutet fuer die Software:

- **Belegprinzip:** Keine Buchung ohne Beleg. Die Software muss sicherstellen, dass zu jeder Buchung ein digitaler oder digitalisierter Beleg verknuepft ist.
- **Progressiver und retrograder Pruefpfad:** Vom Beleg zur Buchung und zurueck muss jeder Geschaeftsvorfall nachvollziehbar sein.
- **Klare Kontierung:** Jede Buchung muss eine eindeutige Kontierung aufweisen.

**Umsetzungshinweis fuer die Software:**
```
Jede Buchung muss folgende Pflichtfelder enthalten:
- Eindeutige Buchungsnummer (fortlaufend)
- Buchungsdatum und Belegdatum
- Belegverweis/Belegnummer
- Soll-Konto und Haben-Konto
- Betrag und Waehrung
- Buchungstext
- Erfassungszeitpunkt (Timestamp)
- Erfassender Benutzer/System
```

#### 1.2.2 Unveraenderbarkeit (Rz. 58-67 GoBD)

Die Unveraenderbarkeit ist eine der **kritischsten Anforderungen** fuer die Softwareentwicklung:

- **Buchungen duerfen nach Festschreibung nicht mehr geloescht oder ueberschrieben werden.** Korrekturen erfolgen ausschliesslich durch Stornobuchungen oder Aenderungsbuchungen.
- **Technische Sicherstellung:** Die Software muss durch technische Massnahmen (z.B. Hash-Werte, Versionierung, Write-Once-Speicher) gewaehrleisten, dass einmal erfasste Daten nicht nachtraeglich manipuliert werden koennen.
- **Festschreibung:** Buchungen muessen zu definierten Zeitpunkten (spaetestens zum Periodenabschluss) festgeschrieben werden. Nach Festschreibung ist eine Aenderung technisch auszuschliessen.

**Umsetzungshinweis fuer die Software:**
- Implementierung eines **Append-Only-Journals** (kein UPDATE oder DELETE auf Buchungssaetze)
- Kryptografische Verkettung der Buchungssaetze (aehnlich einer Blockchain-Struktur)
- Trennung von "offenen" und "festgeschriebenen" Buchungszeitraeumen
- Hash-Werte ueber Buchungsbloecke zur Integritaetspruefung

#### 1.2.3 Vollstaendigkeit (Rz. 36-43 GoBD)

- **Alle** Geschaeftsvorfaelle muessen erfasst werden (keine Luecken).
- Fortlaufende Nummerierung ohne Luecken.
- Keine nachtraegliche Einfuegung zwischen bestehenden Buchungssaetzen.

#### 1.2.4 Richtigkeit (Rz. 44-47 GoBD)

- Geschaeftsvorfaelle muessen inhaltlich zutreffend abgebildet werden.
- Kontierung und Betraege muessen sachlich richtig sein.
- **Bei KI-gestuetzter Kontierung:** Besondere Sorgfaltspflicht bei der Validierung.

#### 1.2.5 Zeitgerechte Erfassung (Rz. 48-53 GoBD)

- Geschaeftsvorfaelle sind **zeitnah** zu erfassen.
- Bei Bargeschaeften: taeglich (Kassenbestand).
- Bei unbaren Geschaeftsvorfaellen: innerhalb von **zehn Tagen**.
- Bei periodenfremd erfassten Vorgaengen: Nachvollziehbarkeit muss gewaehrleistet sein.

#### 1.2.6 Ordnung (Rz. 54-57 GoBD)

- Geschaeftsvorfaelle muessen sachlich und zeitlich geordnet erfasst werden.
- **Journalfunktion:** Chronologische Aufzeichnung aller Buchungen.
- **Kontenfunktion:** Sachliche Ordnung nach Konten.

### 1.3 Verfahrensdokumentation (Rz. 151-155 GoBD)

Die Verfahrensdokumentation ist **Pflicht** fuer alle Steuerpflichtigen, die ein DV-gestuetztes Buchfuehrungssystem nutzen. Sie ist eines der am haeufigsten beanstandeten Elemente bei Betriebspruefungen.

#### Pflichtinhalte der Verfahrensdokumentation:

**a) Allgemeine Beschreibung:**
- Beschreibung der Geschaeftstaetigkeit
- Steuerliche Pflichten des Unternehmens
- Zustaendigkeiten und Verantwortlichkeiten

**b) Anwenderdokumentation:**
- Beschreibung aller Softwarefunktionen
- Eingabemasken und Schnittstellen
- Berechtigungskonzept
- Bedienungsanleitung

**c) Technische Systemdokumentation:**
- IT-Infrastruktur und Systemarchitektur
- Datenmodell und Datenbankstruktur
- Schnittstellen zu Drittsystemen (z.B. Bankanbindung, ELSTER)
- Sicherheitskonzept (Zugriffsschutz, Verschluesselung, Backup)

**d) Betriebsdokumentation:**
- Beschreibung der Betriebsablaeufe
- Datensicherungskonzept
- Notfallkonzept
- Aktualisierung und Versionierung

**Besonderheit bei KI-Systemen:**
Die Verfahrensdokumentation muss zusaetzlich beschreiben:
- Wie die KI Buchungsvorschlaege generiert
- Welche Modelle und Algorithmen verwendet werden
- Wie die Qualitaetssicherung der KI-Ergebnisse erfolgt
- Welche Kontrollmechanismen (Human-in-the-Loop) implementiert sind
- Wie Modellaenderungen dokumentiert und validiert werden

### 1.4 Protokollierung und Audit Trail (Rz. 58-67 GoBD)

Die Software muss einen **lueckenlosen Audit Trail** implementieren:

- **Jede Aenderung** an Stammdaten, Buchungsdaten und Einstellungen muss protokolliert werden.
- Das Protokoll muss enthalten: Wer, Was, Wann, Alter Wert, Neuer Wert.
- Die **Aenderungshistorie selbst darf nachtraeglich nicht veraenderbar** sein.
- Protokolle muessen waehrend der gesamten Aufbewahrungsfrist maschinell auswertbar bleiben.
- Ueber den gesamten Lebenszyklus eines Belegs muessen alle Bearbeitungsschritte lueckenlos protokolliert werden.
- Das Protokoll muss mit dem jeweiligen Beleg bzw. der Buchung verknuepft sein.

**Umsetzungshinweis:**
```
Audit-Log-Eintrag (Pflichtfelder):
- Timestamp (hochaufloesend, z.B. Mikrosekunden)
- Benutzer/System-ID
- Aktion (CREATE, MODIFY, DELETE_ATTEMPT, VIEW, EXPORT)
- Betroffene Entitaet (Buchung, Stammdatum, Beleg)
- Entitaets-ID
- Vorheriger Zustand (bei Aenderungen)
- Neuer Zustand (bei Aenderungen)
- IP-Adresse / Session-ID
- Hashwert des Eintrags
- Verweis auf vorherigen Log-Eintrag (Verkettung)
```

### 1.5 Datensicherung und Datensicherheit (Rz. 103-107 GoBD)

- Regelmaessige Datensicherungen sind durchzufuehren.
- Backup-Konzept mit definierten Sicherungsintervallen.
- Physische Trennung der Sicherungskopien.
- Regelmaessige Tests der Wiederherstellbarkeit.
- Zugriffsschutz durch Berechtigungskonzept mit Rollen und Rechten.

### 1.6 E-Rechnungspflicht ab 2025

Seit dem **1. Januar 2025** gilt die Pflicht zum Empfang elektronischer Rechnungen im B2B-Bereich:

- Elektronische Rechnungen muessen in einem strukturierten Format gemaess **EN 16931** ausgestellt und empfangen werden (z.B. **XRechnung** oder **ZUGFeRD**).
- Eine einfache PDF-Rechnung gilt **nicht** mehr als elektronische Rechnung.
- Die Software muss E-Rechnungen importieren, validieren und verarbeiten koennen.

**Stufenweiser Zeitplan:**
- Ab 01.01.2025: Empfangspflicht fuer alle Unternehmen
- Ab 01.01.2027: Ausstellungspflicht fuer Unternehmen mit > 800.000 EUR Vorjahresumsatz
- Ab 01.01.2028: Ausstellungspflicht fuer alle Unternehmen

---

## 2. HGB-Anforderungen

### 2.1 Buchfuehrungspflicht (ss 238-241a HGB)

#### s 238 HGB - Buchfuehrungspflicht

Jeder Kaufmann (und damit jede GmbH als Formkaufmann gemaess s 6 Abs. 1 HGB i.V.m. s 13 Abs. 3 GmbHG) ist verpflichtet:

- **Buecher zu fuehren** und in diesen seine Handelsgeschaefte und die Lage seines Vermoegens nach den Grundsaetzen ordnungsgemaesser Buchfuehrung ersichtlich zu machen (s 238 Abs. 1 S. 1 HGB).
- Die Buchfuehrung muss so beschaffen sein, dass sie einem **sachverstaendigen Dritten** innerhalb angemessener Zeit einen Ueberblick ueber die Geschaeftsvorfaelle und ueber die Lage des Unternehmens vermitteln kann (s 238 Abs. 1 S. 2 HGB).
- Geschaeftsvorfaelle muessen sich in ihrer **Entstehung und Abwicklung verfolgen** lassen (s 238 Abs. 1 S. 3 HGB).

#### s 239 HGB - Fuehrung der Handelsbuecher

Relevante Anforderungen fuer die Software:
- **Lebende Sprache** und **anerkannte Abkuerzungen** (s 239 Abs. 1 HGB)
- **Vollstaendige, richtige, zeitgerechte und geordnete** Eintragungen (s 239 Abs. 2 HGB)
- **Unveraenderbarkeit:** Eine Eintragung oder Aufzeichnung darf nicht in einer Weise veraendert werden, dass der urspruengliche Inhalt nicht mehr feststellbar ist (s 239 Abs. 3 HGB)
- **Elektronische Buchfuehrung** ist zulaessig, wenn die Daten waehrend der Aufbewahrungsfrist verfuegbar und lesbar gemacht werden koennen (s 239 Abs. 4 HGB)

### 2.2 Grundsaetze ordnungsgemaesser Buchfuehrung (GoB)

Die GoB sind teils gesetzlich kodifiziert, teils durch Rechtsprechung und Praxis entwickelte Grundsaetze:

| Grundsatz | Rechtsgrundlage | Relevanz fuer Software |
|-----------|----------------|----------------------|
| Klarheit und Uebersichtlichkeit | s 243 Abs. 2 HGB | Strukturierte Darstellung der Buchfuehrung |
| Vollstaendigkeit | s 246 Abs. 1 HGB | Alle Vermoegensgegenstaende und Schulden erfassen |
| Verrechnungsverbot | s 246 Abs. 2 HGB | Keine Saldierung von Aktiva/Passiva oder Aufwendungen/Ertraegen |
| Einzelbewertung | s 252 Abs. 1 Nr. 3 HGB | Jeder Posten ist einzeln zu bewerten |
| Vorsichtsprinzip | s 252 Abs. 1 Nr. 4 HGB | Vorhersehbare Risiken und Verluste beruecksichtigen |
| Periodenabgrenzung | s 252 Abs. 1 Nr. 5 HGB | Zeitraumrichtige Zuordnung |
| Bewertungsstetigkeit | s 252 Abs. 1 Nr. 6 HGB | Konsistente Bewertungsmethoden |
| Going Concern | s 252 Abs. 1 Nr. 2 HGB | Fortfuehrungsprinzip beachten |

### 2.3 Anforderungen an den Jahresabschluss (ss 242-263 HGB)

Eine GmbH muss gemaess s 264 Abs. 1 HGB den Jahresabschluss um einen **Anhang** erweitern (Bilanz, GuV und Anhang bilden eine Einheit). Die Software muss folgende Abschlussarbeiten unterstuetzen:

**a) Bilanz (s 266 HGB):**
- Gliederung nach vorgeschriebenem Schema (s 266 Abs. 2 und 3 HGB)
- Groessenabhaengige Erleichterungen (s 266 Abs. 1 S. 3 HGB fuer kleine GmbHs)

**b) Gewinn- und Verlustrechnung (s 275 HGB):**
- Gesamtkostenverfahren oder Umsatzkostenverfahren
- Wahlrecht des Unternehmens, aber Stetigkeit erforderlich

**c) Anhang (ss 284-288 HGB):**
- Angaben zu Bilanzierungs- und Bewertungsmethoden
- Erlaeuterungen zu einzelnen Posten

**d) Groessenklassen (s 267 HGB):**

| Merkmal | Klein | Mittel | Gross |
|---------|-------|--------|-------|
| Bilanzsumme | <= 7,5 Mio. EUR | <= 25 Mio. EUR | > 25 Mio. EUR |
| Umsatzerloese | <= 15 Mio. EUR | <= 50 Mio. EUR | > 50 Mio. EUR |
| Arbeitnehmer | <= 50 | <= 250 | > 250 |

(Schwellenwerte nach DiRUG/BilRUG, aktualisiert 2024)

Die Groessenklasse bestimmt den Umfang der Offenlegungspflichten und beeinflusst, welche Vereinfachungen die Software anbieten kann.

### 2.4 Aufbewahrungspflichten (s 257 HGB)

| Dokumentenart | Aufbewahrungsfrist | Rechtsgrundlage |
|---------------|-------------------|-----------------|
| Handelsbuechern, Inventare, Eroeffnungsbilanzen, Jahresabschluesse, Lageberichte, Konzernabschluesse | **10 Jahre** | s 257 Abs. 4 HGB |
| Empfangene Handelsbriefe | 6 Jahre | s 257 Abs. 4 HGB |
| Wiedergaben abgesandter Handelsbriefe | 6 Jahre | s 257 Abs. 4 HGB |
| **Buchungsbelege** | **8 Jahre** (seit 01.01.2025) | s 257 Abs. 4 HGB i.V.m. BEG IV |

**Wichtig:** Die Aufbewahrungsfrist fuer Buchungsbelege wurde durch das **Vierte Buerokratieentlastungsgesetz (BEG IV)** von zehn auf acht Jahre verkuerzt (ab 01.01.2025).

Die Software muss sicherstellen, dass:
- Aufbewahrte Unterlagen waehrend der gesamten Frist **lesbar und maschinell auswertbar** bleiben
- Ein **Formatwechsel** (z.B. bei Softwareupdates) die Lesbarkeit nicht beeintraechtigt
- Die Daten auch bei Systemmigrationen erhalten bleiben

---

## 3. Abgabenordnung

### 3.1 Buchfuehrungs- und Aufzeichnungspflichten (ss 140-148 AO)

#### s 140 AO - Buchfuehrungspflicht aus anderen Gesetzen

Wer nach **anderen Gesetzen als den Steuergesetzen** Buecher und Aufzeichnungen zu fuehren hat (insbesondere nach HGB), hat diese Verpflichtungen auch fuer die Besteuerung zu erfuellen. Dies betrifft jede GmbH unmittelbar.

#### s 141 AO - Buchfuehrungspflicht bestimmter Steuerpflichtiger

Unabhaengig von der handelsrechtlichen Pflicht werden Gewerbetreibende steuerrechtlich buchfuehrungspflichtig, wenn:
- **Umsaetze** > 800.000 EUR im Kalenderjahr (seit 01.01.2024, zuvor 600.000 EUR) oder
- **Gewinn** > 80.000 EUR im Wirtschaftsjahr (seit 01.01.2024, zuvor 60.000 EUR)

**Hinweis:** Fuer GmbHs hat s 141 AO geringe praktische Bedeutung, da die Buchfuehrungspflicht bereits aus s 238 HGB i.V.m. s 140 AO folgt.

#### s 145 AO - Allgemeine Anforderungen an Buchfuehrung und Aufzeichnungen

- Buchfuehrung muss so beschaffen sein, dass sie einem sachverstaendigen Dritten einen **Ueberblick ueber die Geschaeftsvorfaelle und die Lage des Unternehmens** vermitteln kann.
- Geschaeftsvorfaelle muessen sich in ihrer **Entstehung und Abwicklung** verfolgen lassen.
- Die Aufzeichnungen muessen **vollstaendig, richtig, zeitgerecht und geordnet** vorgenommen werden.

#### s 146 AO - Ordnungsvorschriften fuer die Buchfuehrung und fuer Aufzeichnungen

Zentrale Vorschrift mit folgenden Kernaussagen:

- **s 146 Abs. 1 AO:** Buchungen und Aufzeichnungen sind **vollstaendig, richtig, zeitgerecht und geordnet** vorzunehmen.
- **s 146 Abs. 2 AO:** Kasseneinnahmen und -ausgaben sollen **taeglich** festgehalten werden.
- **s 146 Abs. 4 AO:** Buchungen duerfen nicht in einer Weise veraendert werden, dass der **urspruengliche Inhalt nicht mehr feststellbar** ist.
- **s 146 Abs. 5 AO:** Die Buecher und Aufzeichnungen sind im **Geltungsbereich der AO** zu fuehren und aufzubewahren (Ausnahmen moeglich auf Antrag). Elektronische Buchfuehrung im EU/EWR-Ausland ist unter bestimmten Voraussetzungen zulaessig (s 146 Abs. 2a AO).

### 3.2 s 146a AO - Technische Sicherheitseinrichtung (TSE)

#### Anwendungsbereich

s 146a AO richtet sich primaer an **elektronische Aufzeichnungssysteme**, insbesondere Kassensysteme. Fuer eine reine Buchfuehrungssoftware ohne Kassenmodul hat die TSE-Pflicht nur dann Relevanz, wenn die Software auch als Aufzeichnungssystem fuer Bargeschaefte eingesetzt wird.

#### Kernpflichten (soweit einschlaegig):

- **Einzelaufzeichnungspflicht:** Jeder Geschaeftsvorfall ist einzeln zu erfassen (s 146a Abs. 1 S. 1 AO).
- **Zertifizierte TSE:** Wenn die Software als elektronisches Aufzeichnungssystem qualifiziert, muss eine vom **BSI zertifizierte technische Sicherheitseinrichtung** (bestehend aus Sicherheitsmodul, Speichermedium und einheitlicher digitaler Schnittstelle) eingesetzt werden.
- **Belegausgabepflicht:** Bei Verwendung eines Aufzeichnungssystems muss fuer jeden Geschaeftsvorfall ein Beleg ausgegeben werden koennen.
- **Meldepflicht:** Seit 01.01.2025 muessen elektronische Aufzeichnungssysteme an das Finanzamt gemeldet werden (Name, Steuernummer, Anzahl der Systeme, Seriennummern, Anschaffungsdatum). Frist: **31. Juli 2025**.

#### Kassensicherungsverordnung (KassenSichV)

Ergaenzt s 146a AO durch detaillierte technische Anforderungen:
- **BSI TR-03153:** Technische Richtlinie fuer die TSE
- **DSFinV-K:** Digitale Schnittstelle der Finanzverwaltung fuer Kassensysteme

**Umsetzungshinweis fuer die Software:**
Wenn ein Kassenmodul integriert werden soll, muss eine BSI-zertifizierte TSE (hardware- oder cloudbasiert) angebunden werden. Die Schnittstelle DSFinV-K muss implementiert werden.

### 3.3 s 147 AO - Aufbewahrungspflichten

#### Aufbewahrungsfristen (steuerrechtlich):

| Dokumentenart | Frist | Rechtsgrundlage |
|--------------|-------|-----------------|
| Buecher, Aufzeichnungen, Inventare, Jahresabschluesse, Lageberichte, Eroeffnungsbilanz | **10 Jahre** | s 147 Abs. 1 Nr. 1, Abs. 3 AO |
| Buchungsbelege | **8 Jahre** (seit BEG IV) | s 147 Abs. 1 Nr. 4, Abs. 3 AO |
| Sonstige Unterlagen (z.B. Handels-/Geschaeftsbriefe) | **6 Jahre** | s 147 Abs. 1 Nr. 2, 3, 5, Abs. 3 AO |

#### Elektronische Aufbewahrung:

- Originaer digitale Dokumente muessen **digital aufbewahrt** werden (keine Auslagerung auf Papier).
- Die Daten muessen waehrend der gesamten Aufbewahrungsfrist **maschinell auswertbar** sein.
- Bei Systemwechseln: Zugriff auf Alt-Daten muss fuer **mindestens 5 Jahre** nach Archivierung gewaehrleistet bleiben (s 147 Abs. 6 S. 5 AO).

### 3.4 Digitale Betriebspruefung und Datenzugriff (s 147 Abs. 6 AO)

Die Finanzverwaltung hat drei Zugriffsmoeglichkeiten auf die steuerrelevanten Daten:

#### Z1 - Unmittelbarer Zugriff (Nur-Lese-Zugriff)

- Der Betriebspruefer erhaelt **direkten Nur-Lese-Zugriff** auf das DV-System des Steuerpflichtigen.
- Die Software muss einen **Betriebspruefer-Modus** bereitstellen koennen.
- Zugriff auf alle steuerrelevanten Daten mit Sortier-, Filter- und Auswertungsmoeglichkeiten.
- Das Unternehmen muss einen Arbeitsplatz mit Systemzugang bereitstellen.

#### Z2 - Mittelbarer Zugriff (Auswertungszugriff)

- Der Steuerpflichtige oder ein Beauftragter fuehrt die vom Pruefer gewuenschten **Auswertungen** selbst durch.
- Die Software muss flexible Auswertungen und Berichte generieren koennen.
- Daten muessen in verschiedenen Formaten exportiert werden koennen.

#### Z3 - Datenueberlassung (Datentraegerueberlassung)

- Der Steuerpflichtige stellt die steuerrelevanten Daten auf einem **maschinell lesbaren Datentraeger** zur Verfuegung.
- Die Daten muessen im **GoBD-konformen Exportformat** bereitgestellt werden.
- Praxisrelevant: Export im Format fuer **IDEA/AIS TaxAudit** (Pruefungssoftware der Finanzverwaltung).
- Unterstuetzte Formate: CSV, XML, SAF-T (Standard Audit File for Tax).

**Umsetzungshinweis fuer die Software:**
```
Pflicht-Exportformate:
- CSV mit definierter Struktur (Feldtrenner, Zeichensatz)
- XML nach vorgegebenem Schema
- Indexdatei (index.xml) mit Beschreibung der Dateien
- Beschreibungsdatei mit Feldstrukturen
- Unterstuetzung von IDEA/AIS TaxAudit Importformaten
```

Die Software muss **alle drei Zugriffsarten** (Z1, Z2, Z3) unterstuetzen. Dies ist eine harte Anforderung, die bei Betriebspruefungen kontrolliert wird.

---

## 4. Datenschutz (DSGVO)

### 4.1 Anwendbarkeit auf Finanzdaten

Finanzdaten in der Buchfuehrung enthalten regelmaessig **personenbezogene Daten** im Sinne der DSGVO:

- Namen, Adressen und Kontaktdaten von Geschaeftspartnern (Kunden, Lieferanten)
- Bankverbindungen (IBAN, BIC)
- Steuernummern und Umsatzsteuer-Identifikationsnummern
- Verguetungsdaten von Mitarbeitern
- Zahlungsverhalten und Bonituetsinformationen

Die DSGVO findet daher **vollumfaenglich Anwendung** auf die Verarbeitung dieser Daten in der Buchhaltungssoftware.

### 4.2 Rechtsgrundlagen der Verarbeitung

Fuer die Verarbeitung personenbezogener Daten in der Buchhaltung kommen folgende Rechtsgrundlagen in Betracht:

| Rechtsgrundlage | Anwendungsfall |
|----------------|----------------|
| **Art. 6 Abs. 1 lit. b DSGVO** | Vertragliche Verpflichtungen (Rechnungsstellung, Zahlungsabwicklung) |
| **Art. 6 Abs. 1 lit. c DSGVO** | Gesetzliche Buchfuehrungspflichten (HGB, AO, UStG) |
| **Art. 6 Abs. 1 lit. f DSGVO** | Berechtigtes Interesse (z.B. Forderungsmanagement) |

### 4.3 Loeschkonzept vs. Aufbewahrungspflicht

Einer der **zentralen Konflikte** in der Finanzbuchhaltungssoftware:

#### Das Problem:
- **DSGVO Art. 17:** Recht auf Loeschung ("Recht auf Vergessenwerden")
- **DSGVO Art. 5 Abs. 1 lit. e:** Speicherbegrenzung (Daten nur so lange wie noetig)
- **HGB s 257, AO s 147:** Aufbewahrungspflichten von 6-10 Jahren
- **GoBD:** Unveraenderbarkeit der Buchfuehrungsdaten

#### Die Loesung:

Die Software muss ein **abgestuftes Loeschkonzept** implementieren:

1. **Waehrend der Aufbewahrungsfrist:**
   - Personenbezogene Daten in der Buchfuehrung duerfen **nicht geloescht** werden (gesetzliche Pflicht gemaess Art. 6 Abs. 1 lit. c DSGVO ueberwiegt).
   - Zugriff auf diese Daten kann jedoch **eingeschraenkt** werden (Zugriffsbeschraenkung statt Loeschung).
   - Daten sollten als **gesperrt** markiert werden, wenn der operative Zweck entfaellt, aber die Aufbewahrungsfrist noch laeuft.

2. **Nach Ablauf der Aufbewahrungsfrist:**
   - Personenbezogene Daten muessen **geloescht oder anonymisiert** werden.
   - Die Software muss automatisierte Loeschroutinen implementieren.
   - Die statistische/analytische Nutzbarkeit der Daten kann durch **Anonymisierung** erhalten bleiben.

3. **Automatische Fristueberwachung:**
   - Die Software muss die **Aufbewahrungsfristen automatisch ueberwachen** und nach deren Ablauf die Loeschung ausloesen oder den Anwender informieren.
   - Unterschiedliche Fristen fuer verschiedene Dokumentenarten muessen beruecksichtigt werden.

**Hinweis:** Die **Datenschutzkonferenz (DSK)** hat fuer 2025/2026 einen besonderen Pruefungsschwerpunkt auf die Einhaltung von Loeschkonzepten gelegt. Insgesamt 32 Datenschutzaufsichtsbehoerden in Europa beteiligen sich an koordinierten Pruefaktionen.

### 4.4 Datenschutz bei KI-Verarbeitung von Finanzdaten

#### 4.4.1 Datenschutz-Folgenabschaetzung (DSFA)

Gemaess **Art. 35 DSGVO** ist bei Einsatz von KI fuer die Verarbeitung von Finanzdaten in der Regel eine DSFA erforderlich, da:
- **Neue Technologien** eingesetzt werden (KI/ML)
- **Umfangreiche Verarbeitung** personenbezogener Daten stattfindet
- **Profiling** oder automatisierte Entscheidungsfindung moeglich ist
- Die Aufsichtsbehoerden bei KI-Anwendungen **haeufig von einem hohen Risiko** ausgehen

Die DSFA muss dokumentieren:
- Beschreibung der Verarbeitungsvorgaenge und Zwecke
- Bewertung der Notwendigkeit und Verhaeltnismaessigkeit
- Bewertung der Risiken fuer Betroffene
- Geplante Abhilfemassnahmen

#### 4.4.2 Auftragsverarbeitung (Art. 28 DSGVO)

Wenn die KI ueber eine externe API (z.B. Claude API von Anthropic) betrieben wird:
- Es liegt eine **Auftragsverarbeitung** vor.
- Ein **Auftragsverarbeitungsvertrag (AVV)** mit dem KI-Anbieter ist **zwingend erforderlich**.
- Der AVV muss den Anforderungen von Art. 28 Abs. 3 DSGVO genuegen.
- **Drittlandtransfer:** Wenn Daten ausserhalb des EWR verarbeitet werden (z.B. Server in den USA), sind zusaetzliche Garantien erforderlich (Standardvertragsklauseln, Angemessenheitsbeschluss, Transfer Impact Assessment).

#### 4.4.3 Automatisierte Einzelentscheidung (Art. 22 DSGVO)

- Buchungsentscheidungen, die rein KI-gesteuert ohne menschliche Intervention getroffen werden, koennten unter **Art. 22 DSGVO** fallen.
- Betroffene haben das Recht, **nicht einer ausschliesslich auf automatisierter Verarbeitung beruhenden Entscheidung** unterworfen zu werden, die ihnen gegenueber rechtliche Wirkung entfaltet.
- **Empfehlung:** Implementierung eines **Human-in-the-Loop-Ansatzes**, bei dem jede KI-Buchung von einem Menschen freigegeben wird.

#### 4.4.4 Technisch-Organisatorische Massnahmen (TOMs)

Die Software muss gemaess **Art. 32 DSGVO** geeignete TOMs implementieren:

- **Verschluesselung:** AES-256 fuer Daten at rest, TLS 1.3 fuer Daten in transit
- **Pseudonymisierung:** Soweit moeglich, Trennung identifizierender Merkmale
- **Zugriffskontrolle:** Rollenbasiertes Berechtigungssystem (RBAC)
- **Protokollierung:** Zugriffe auf personenbezogene Daten loggen
- **Datentrennung:** Mandantentrennung bei Multi-Tenant-Architektur
- **Belastbarkeit:** Redundanz und Ausfallsicherheit

### 4.5 Informationspflichten (Art. 13, 14 DSGVO)

Die Software muss Funktionen bereitstellen, um den Informationspflichten nachzukommen:
- **Datenschutzerklaerung** fuer Geschaeftspartner
- **Auskunftsrecht** (Art. 15 DSGVO): Export aller gespeicherten Daten einer Person
- **Berichtigung** (Art. 16 DSGVO): Korrekturfunktion fuer Stammdaten
- **Datenportabilitaet** (Art. 20 DSGVO): Export in maschinenlesbarem Format

---

## 5. Zertifizierungen und Pruefungen

### 5.1 IDW PS 880 (Softwarepruefung)

#### Ueberblick

Der **IDW Pruefungsstandard 880** (Die Pruefung von Softwareprodukten) des Instituts der Wirtschaftspruefer (IDW) ist der massgebliche Standard fuer die Pruefung von Buchhaltungssoftware in Deutschland. Die aktuelle Neufassung stammt aus **01/2022**.

#### Pruefungsgegenstand

Die Pruefung nach IDW PS 880 umfasst:

1. **Programmfunktionen:**
   - Korrekte Verarbeitung von Buchungen
   - Einhaltung der GoB
   - Korrekte Saldierung und Abschlussarbeiten
   - Steuerrechtliche Berechnungen (USt, KSt, GewSt)

2. **Programmkontrollen:**
   - Eingabekontrollen (Plausibilitaetspruefungen)
   - Verarbeitungskontrollen
   - Ausgabekontrollen

3. **Programmdokumentation:**
   - Benutzerdokumentation
   - Technische Dokumentation
   - Verfahrensdokumentation

4. **IT-Sicherheit:**
   - Zugriffsschutz
   - Datensicherung
   - Protokollierung

#### Pruefungsergebnis

- Bei positivem Ergebnis wird ein **Softwaretestat** erteilt.
- Das Testat bestaetigt, dass die Software bei ordnungsgemaesser Anwendung eine **GoB-konforme Buchfuehrung** ermoeglicht.
- Abschlusspruefer koennen bei IDW PS 880-zertifizierter Software ihren **Pruefungsaufwand reduzieren**.

#### Bedeutung fuer die Entwicklung

- Ein IDW PS 880 Testat ist **nicht gesetzlich vorgeschrieben**, aber **marktrelevant und vertrauensbildend**.
- Ohne Testat muessen Nutzer und deren Wirtschaftspruefer die GoBD-Konformitaet der Software **selbst nachweisen**.
- **Empfehlung:** Anstrebung eines IDW PS 880 Testats als wichtiger Wettbewerbsvorteil gegenueber DATEV.

#### KI-spezifische Pruefungsaspekte

Fuer eine KI-gestuetzte Software ergeben sich zusaetzliche Pruefungsfragen:
- Nachvollziehbarkeit der KI-Entscheidungen
- Qualitaetssicherung der KI-generierten Buchungen
- Auswirkungen von Modellaenderungen auf die Buchungsqualitaet
- Robustheit und Zuverlaessigkeit der KI-Kontierung

### 5.2 BSI-Grundschutz

#### Relevanz

Der **BSI IT-Grundschutz** bietet einen umfassenden Rahmen fuer die Informationssicherheit. Fuer Buchhaltungssoftware sind insbesondere relevant:

- **Baustein APP.4.2 - SAP-ERP-System** (analog anwendbar auf andere ERP/Finanzsysteme)
- **Baustein OPS.1.1.4 - Schutz vor Schadprogrammen**
- **Baustein CON.1 - Kryptokonzept**
- **Baustein OPS.1.2.4 - Telearbeit** (bei Cloud-Nutzung)
- **Baustein APP.4.3 - Relationale Datenbanksysteme**

#### Umsetzungsempfehlungen

| Massnahme | Beschreibung |
|-----------|-------------|
| Verschluesselung | AES-256 fuer Daten, TLS 1.3 fuer Kommunikation |
| Zugriffsmanagement | RBAC mit Least-Privilege-Prinzip |
| Protokollierung | Umfassende Audit-Logs mit Integritaetsschutz |
| Schwachstellenmanagement | Regelmaessige Security-Audits und Penetrationstests |
| Notfallmanagement | Dokumentierter Disaster-Recovery-Plan |
| Patch-Management | Regelmaessige und zeitnahe Sicherheitsupdates |

### 5.3 ISO-Normen

#### ISO/IEC 27001 (Informationssicherheit)

- Internationaler Standard fuer Informationssicherheits-Managementsysteme (ISMS)
- **Empfohlen** fuer den Betrieb der Buchhaltungssoftware (insbesondere bei Cloud-Bereitstellung)
- Eine **ISO 27001-Zertifizierung auf Basis von IT-Grundschutz** wird vom BSI angeboten und ist als gleichwertig anerkannt

#### ISO 25010 (Softwarequalitaet)

- Qualitaetsmodell fuer Softwareprodukte
- Relevante Qualitaetsmerkmale: Funktionalitaet, Zuverlaessigkeit, Benutzbarkeit, Effizienz, Wartbarkeit, Portabilitaet, Sicherheit

#### ISO/IEC 27701 (Datenschutz-Erweiterung)

- Erweiterung von ISO 27001 um Datenschutzaspekte
- Unterstuetzt die Compliance mit DSGVO-Anforderungen

#### ISO 22301 (Business Continuity)

- Relevant fuer die Gewaehrleistung der Verfuegbarkeit der Buchhaltungssoftware
- Besonders wichtig bei Cloud-basierter Bereitstellung

### 5.4 SOC 2 Type II

- Relevant, wenn die Software als **SaaS/Cloud-Dienst** angeboten wird
- Prueft: Sicherheit, Verfuegbarkeit, Verarbeitungsintegritaet, Vertraulichkeit, Datenschutz
- Wird zunehmend von Geschaeftskunden gefordert

---

## 6. Haftungsrisiken

### 6.1 Haftung bei fehlerhafter Buchfuehrung

#### 6.1.1 Steuerliche Konsequenzen

Bei fehlerhafter Buchfuehrung (auch durch KI-Fehler) drohen:

- **Verwerfung der Buchfuehrung** durch das Finanzamt (s 158 AO)
- **Schaetzung der Besteuerungsgrundlagen** (s 162 AO) -- im Regelfall zu Lasten des Steuerpflichtigen
- **Bussgeld** von bis zu **25.000 EUR** bei Verstoessen gegen GoBD (s 379 AO)
- **Steuerhinterziehung** (s 370 AO) oder **leichtfertige Steuerverkuerzung** (s 378 AO) bei Vorsatz bzw. Leichtfertigkeit
- **Verzoegerungsgeld** von 2.500 bis 250.000 EUR bei verweigertem Datenzugriff (s 146 Abs. 2c AO)

#### 6.1.2 Zivilrechtliche Konsequenzen

- **Schadensersatzansprueche** von Gesellschaftern oder Glaeubigern
- **Insolvenzantragspflicht:** Fehlerhafte Buchfuehrung kann dazu fuehren, dass eine Ueberschuldung oder Zahlungsunfaehigkeit nicht rechtzeitig erkannt wird (s 15a InsO)

### 6.2 Geschaeftsfuehrerhaftung (s 43 GmbHG)

#### Sorgfaltspflicht

Gemaess **s 43 Abs. 1 GmbHG** haben Geschaeftsfuehrer die **Sorgfalt eines ordentlichen Geschaeftsmannes** anzuwenden. Dies umfasst:

- Die **Pflicht zur ordnungsgemaessen Buchfuehrung** (s 41 GmbHG)
- Die **Ueberwachungspflicht** -- auch wenn die Buchfuehrung an Dritte (Steuerberater) oder an Software delegiert wird
- Die **Informationspflicht** -- der Geschaeftsfuehrer muss sich laufend ueber die wirtschaftliche Lage informieren

#### Haftungsmassstab bei KI-Einsatz

- Der Geschaeftsfuehrer kann die **operative Buchfuehrung delegieren** (an Steuerberater, Buchhalter oder Software), aber **nicht die Verantwortung**.
- Bei Einsatz einer KI-gestuetzten Software muss der Geschaeftsfuehrer:
  - Sicherstellen, dass die Software **fachlich geeignet** ist
  - **Regelmaessige Stichproben** der KI-generierten Buchungen durchfuehren (lassen)
  - Ein **Kontrollsystem** implementieren (z.B. Vier-Augen-Prinzip, Human-in-the-Loop)
  - Sich ueber die **Funktionsweise und Grenzen** der KI informieren
  - Bei **erkennbaren Fehlern** unverzueglich eingreifen

#### Schadensersatz

- **s 43 Abs. 2 GmbHG:** Geschaeftsfuehrer, die ihre Pflichten verletzen, haften der Gesellschaft **gesamtschuldnerisch** fuer den entstandenen Schaden.
- Die **Beweislast** liegt beim Geschaeftsfuehrer: Er muss beweisen, dass er die Sorgfalt eines ordentlichen Geschaeftsmannes beachtet hat.
- Haftungsbegrenzung ist nur eingeschraenkt moeglich (z.B. durch D&O-Versicherung).

### 6.3 Organhaftung und Delegation

#### Business Judgement Rule (s 43 Abs. 1 GmbHG analog s 93 Abs. 1 S. 2 AktG)

Eine **Pflichtverletzung liegt nicht vor**, wenn der Geschaeftsfuehrer bei einer unternehmerischen Entscheidung:
- Vernuenftigerweise annehmen durfte, auf der Grundlage **angemessener Informationen** gehandelt zu haben
- Zum **Wohle der Gesellschaft** gehandelt hat
- Keinen **Interessenkonflikt** hatte

**Fuer den KI-Einsatz bedeutet dies:**
- Die Auswahl einer KI-gestuetzten Software ist eine unternehmerische Entscheidung.
- Der Geschaeftsfuehrer muss sich **angemessen informieren** (Marktvergleich, Referenzen, Testlaeufe).
- Die Entscheidung muss **dokumentiert** werden.
- Ein IDW PS 880 Testat der Software staerkt die Position des Geschaeftsfuehrers erheblich.

#### Haftung des Softwareherstellers

- Der Hersteller der Buchhaltungssoftware kann **vertraglich und deliktisch** fuer Softwarefehler haften.
- **Produkthaftung** (ProdHaftG) ist grundsaetzlich anwendbar, aber Finanzdaten sind keine Koerperschaeden.
- **Vertragliche Haftung:** AGB koennen die Haftung begrenzen, aber nicht fuer Vorsatz und grobe Fahrlaessigkeit.
- **Empfehlung fuer den Softwareanbieter:** Angemessene Haftpflichtversicherung (IT-Haftpflicht, E&O-Versicherung).

### 6.4 Steuerberaterhaftung und KI

- Wenn ein Steuerberater die KI-gestuetzte Software einsetzt, bleibt er **berufsrechtlich verantwortlich** fuer die Ergebnisse.
- Der Steuerberater muss die KI-Ergebnisse **eigenverantwortlich pruefen** und darf sie nicht ungepreuft uebernehmen.
- Die **Berufshaftpflichtversicherung** des Steuerberaters deckt grundsaetzlich auch Fehler ab, die auf fehlerhafter Software beruhen -- vorausgesetzt, der Steuerberater hat seiner Pruefpflicht genuegt.

---

## 7. KI-spezifische rechtliche Fragen

### 7.1 EU AI Act (KI-Verordnung) und Einordnung

#### 7.1.1 Zeitplan der Umsetzung

| Datum | Pflicht |
|-------|---------|
| Seit 02.02.2025 | Verbot verbotener KI-Praktiken (Art. 5) |
| Ab 02.08.2025 | KI-Kompetenz-Pflicht (Art. 4), Pflichten fuer GPAI-Modelle |
| **Ab 02.08.2026** | **Pflichten fuer Hochrisiko-KI-Systeme (Anhang III)** |
| Ab 02.08.2027 | Pflichten fuer Hochrisiko-KI-Systeme als Sicherheitskomponenten |

#### 7.1.2 Risikoklassifizierung einer KI-Buchfuehrungssoftware

Die Einordnung einer KI-gestuetzten Buchhaltungssoftware haengt von ihrem konkreten Einsatzzweck ab:

**a) Hochrisiko-KI (Anhang III Nr. 5b):**
KI-Systeme, die fuer die **Kreditwuerdigkeitspruefung und Bonitaetsbewertung natuerlicher Personen** eingesetzt werden, sind Hochrisiko-KI. Eine reine Buchhaltungssoftware faellt **nicht automatisch** in diese Kategorie, es sei denn, sie wird fuer Bonitaetspruefungen eingesetzt.

**b) Begrenztes Risiko (Art. 50):**
Eine KI-gestuetzte Buchfuehrungssoftware wird voraussichtlich in die Kategorie **begrenztes Risiko** einzuordnen sein, wenn sie:
- Buchungsvorschlaege generiert, die von Menschen geprueft werden
- Keine Entscheidungen mit unmittelbarer rechtlicher Wirkung gegenueber natuerlichen Personen trifft
- Kein erhebliches Risiko fuer Gesundheit, Sicherheit oder Grundrechte birgt

**c) Minimales Risiko:**
Wenn die KI lediglich als **Assistenz** fungiert (z.B. Vorschlag von Kontierungen, die manuell bestaetigt werden), kann eine Einordnung als minimales Risiko in Betracht kommen.

#### 7.1.3 Pflichten bei begrenztem Risiko

- **Transparenzpflicht (Art. 50):** Nutzer muessen darauf hingewiesen werden, dass sie mit einem KI-System interagieren.
- **Kennzeichnungspflicht:** KI-generierte Inhalte (z.B. Buchungsvorschlaege) muessen als solche erkennbar sein.

#### 7.1.4 Pflichten bei Hochrisiko-Einstufung (falls einschlaegig)

Sollte die Software als Hochrisiko-KI eingestuft werden, greifen umfangreiche Pflichten:

- **Risikomanagement-System** (Art. 9)
- **Datenqualitaets-Anforderungen** (Art. 10)
- **Technische Dokumentation** (Art. 11)
- **Transparenz fuer Nutzer** (Art. 13)
- **Menschliche Aufsicht** (Art. 14)
- **Genauigkeit, Robustheit, Cybersicherheit** (Art. 15)
- **Konformitaetsbewertung** vor Inbetriebnahme
- **CE-Kennzeichnung**
- **Registrierung in EU-Datenbank**

**Empfehlung:** Selbst wenn die Software als begrenztes Risiko eingestuft wird, sollten die Hochrisiko-Anforderungen **freiwillig so weit wie moeglich umgesetzt** werden (insbesondere Dokumentation, Risikomanagement und menschliche Aufsicht), um:
- Regulatorische Sicherheit zu schaffen
- Vertrauen bei Kunden aufzubauen
- Auf moegliche zukuenftige Verschaerfungen vorbereitet zu sein

### 7.2 Rechtliche Zulaessigkeit von KI-gestuetzter Buchfuehrung

#### 7.2.1 Grundsaetzliche Zulaessigkeit

Es gibt **kein gesetzliches Verbot** des Einsatzes von KI in der Buchfuehrung. Die GoBD sprechen technologieneutral von "DV-gestuetzten Buchfuehrungssystemen". Entscheidend ist, dass die **materiellen Anforderungen** (Richtigkeit, Vollstaendigkeit, Nachvollziehbarkeit etc.) erfuellt sind -- unabhaengig davon, ob Buchungen manuell oder KI-gestuetzt erstellt werden.

#### 7.2.2 Grenzen der KI-Nutzung

- Die **Letztverantwortung** fuer die Buchfuehrung verbleibt beim Menschen (Geschaeftsfuehrer, Steuerberater).
- KI-generierte Buchungen muessen **nachpruefbar und erklaerbar** sein.
- Die KI darf keine **Black Box** sein -- die Entscheidungsfindung muss dokumentiert und nachvollziehbar sein.

### 7.3 Transparenz und Erklaerbarkeit

#### 7.3.1 Anforderungen an die Erklaerbarkeit

Fuer eine GoBD-konforme KI-Buchfuehrung muss die Software erklaeren koennen:

- **Warum** ein bestimmtes Konto gewaehlt wurde
- **Auf welcher Grundlage** (Beleg, Regelwerk, Mustererkennung) die Buchung erstellt wurde
- **Welche Alternativen** die KI in Betracht gezogen hat
- **Mit welcher Konfidenz** die KI die Buchung vorschlaegt

#### 7.3.2 Umsetzung der Transparenz

Die Software sollte fuer jede KI-generierte Buchung einen **Erklaerungsbericht** bereitstellen:

```
Buchungsvorschlag #12345
========================
Beleg: Rechnung RE-2026-001 von Lieferant XY
KI-Analyse:
  - Erkannter Geschaeftsvorfall: Wareneinkauf
  - Vorgeschlagene Kontierung: Soll 5000 (Wareneinkauf) / Haben 1200 (Bank)
  - Konfidenz: 97,3%
  - Begruendung: Lieferant XY ist im Stamm als Warenlieferant hinterlegt.
    Rechnungsbetrag und Zahlungsbedingungen entsprechen Standardkonditionen.
    Vorsteuer 19% wurde separat erfasst.
  - Alternative: Soll 5100 (Fremdleistungen) - Konfidenz: 2,1%
  - Regelwerk-Referenz: SKR03, Kontenplan v2026.1
Status: Freigabe durch Buchhalter ausstehend
```

### 7.4 Human-in-the-Loop Anforderungen

#### 7.4.1 Rechtliche Notwendigkeit

Ein **Human-in-the-Loop (HITL)** Ansatz ist aus mehreren Gruenden **dringend empfohlen** bzw. teilweise **rechtlich erforderlich**:

1. **GoBD:** Die Verantwortung fuer die Richtigkeit der Buchfuehrung liegt beim Steuerpflichtigen -- nicht bei der Software.
2. **s 43 GmbHG:** Der Geschaeftsfuehrer muss die Buchfuehrung ueberwachen.
3. **Art. 22 DSGVO:** Verbot ausschliesslich automatisierter Entscheidungen mit rechtlicher Wirkung (soweit personenbezogene Daten betroffen).
4. **EU AI Act:** Menschliche Aufsicht ist bei Hochrisiko-KI Pflicht (Art. 14) und generell als Best Practice empfohlen.
5. **Berufsrecht:** Steuerberater muessen KI-Ergebnisse eigenverantwortlich pruefen.

#### 7.4.2 Empfohlene HITL-Stufen

| Stufe | Anwendungsfall | KI-Rolle | Mensch-Rolle |
|-------|---------------|----------|-------------|
| **1 - Vollautonome Vorerfassung** | Standardbuchungen (z.B. Bankumsaetze mit klarer Zuordnung, Konfidenz > 99%) | Erstellt Buchung | Prueft in regelmaessigen Stichproben |
| **2 - Vorschlag mit Freigabe** | Normale Buchungen (Konfidenz 80-99%) | Erstellt Vorschlag | Prueft und gibt frei |
| **3 - Assistenz** | Komplexe Buchungen (z.B. Abgrenzungen, Rueckstellungen, Konfidenz < 80%) | Liefert Kontierungsvorschlag mit Erklaerung | Entscheidet eigenstaendig |
| **4 - Manuelle Buchung** | Sonderfaelle (z.B. steuerliche Gestaltungen, einmalige Vorgaenge) | Kein Eingriff oder allgemeine Hinweise | Erstellt Buchung komplett manuell |

**Empfehlung:** Mindestens **Stufe 2** als Standard implementieren. Stufe 1 nur fuer klar definierte, repetitive Standardbuchungen mit nachtraeglicher Stichprobenkontrolle.

### 7.5 KI-Kompetenz-Pflicht (Art. 4 EU AI Act)

Seit **02.08.2025** gilt die Pflicht, dass Anbieter und Betreiber von KI-Systemen sicherstellen muessen, dass ihr Personal ueber ein **ausreichendes Mass an KI-Kompetenz** verfuegt. Dies bedeutet:

- Die Software muss **Schulungsmaterialien** fuer Nutzer bereitstellen
- Es sollten **Onboarding-Prozesse** integriert werden
- Die Dokumentation muss die **Moeglichkeiten und Grenzen** der KI transparent darstellen

---

## 8. ELSTER und Finanzverwaltung

### 8.1 ELSTER-Schnittstelle (ERiC)

#### 8.1.1 Grundlagen

Die elektronische Uebermittlung steuerlicher Daten an die Finanzverwaltung erfolgt ueber das **ELSTER-Verfahren** (Elektronische Steuererklaerung). Die technische Schnittstelle ist **ERiC** (ELSTER Rich Client).

#### 8.1.2 Technische Anforderungen

- **ERiC-Bibliothek:** Integration der offiziellen ERiC-Bibliothek (bereitgestellt durch die Finanzverwaltung) als DLL/Shared Library
- **Zertifikat:** ELSTER-Zertifikat (Organisationszertifikat fuer Softwareanbieter) erforderlich
- **Registrierung:** Als Softwarehersteller bei ELSTER registrieren (Herstellerbescheinigung erforderlich)
- **Testumgebung:** ELSTER stellt eine Testumgebung fuer die Entwicklung bereit
- **Versionen:** ERiC wird jaehrlich aktualisiert -- die Software muss mit Updates Schritt halten

#### 8.1.3 Uebermittlungspflichten

| Steuerart | Pflicht | Rechtsgrundlage |
|-----------|---------|-----------------|
| Umsatzsteuer-Voranmeldung | Monatlich/vierteljaehrlich | s 18 Abs. 1 UStG |
| Zusammenfassende Meldung (ZM) | Monatlich/vierteljaehrlich | s 18a UStG |
| Lohnsteuer-Anmeldung | Monatlich/vierteljaehrlich/jaehrlich | s 41a EStG |
| Koerperschaftsteuer-Erklaerung | Jaehrlich | s 31 Abs. 1a KStG |
| Gewerbesteuer-Erklaerung | Jaehrlich | s 14a GewStG |
| E-Bilanz | Jaehrlich | s 5b EStG |
| Umsatzsteuer-Jahreserklaerung | Jaehrlich | s 18 Abs. 3 UStG |

### 8.2 E-Bilanz (s 5b EStG)

#### 8.2.1 Pflicht und Umfang

Seit 2013 muessen **alle bilanzierenden Steuerpflichtigen** (also auch GmbHs) den Inhalt der Bilanz sowie der GuV in **elektronischer Form** an das Finanzamt uebermitteln (s 5b EStG).

#### 8.2.2 XBRL-Taxonomie

Die E-Bilanz muss im **XBRL-Format** (eXtensible Business Reporting Language) uebermittelt werden. Die Finanzverwaltung veroeffentlicht jaehrlich aktualisierte Taxonomien:

- **Kerntaxonomie:** Gilt fuer alle bilanzierenden Unternehmen
- **Ergaenzungstaxonomien:** Branchenspezifische Ergaenzungen
- **Spezialtaxonomien:** Z.B. fuer Banken, Versicherungen

**Aktuelle Versionen:**
- Taxonomie **6.8** fuer Wirtschaftsjahre ab 01.01.2025
- Taxonomie **6.9** fuer Wirtschaftsjahre ab 01.01.2026

#### 8.2.3 Mindestumfang der E-Bilanz

Die Taxonomie unterscheidet zwischen:
- **Mussfeldern:** Zwingend zu uebermitteln (z.B. Eigenkapital, Verbindlichkeiten)
- **Mussfeldern mit Auffangposition:** Differenzierung erwuenscht, Sammelbetrag zulaessig
- **Freiwilligen Feldern:** Optionale Detailinformationen
- **Kontennachweis:** Detaillierte Aufschluesselung der Kontensalden (freiwillig aber empfohlen)

#### 8.2.4 Umsetzungshinweis fuer die Software

```
Anforderungen E-Bilanz-Modul:
1. Mapping: Kontenrahmen (SKR03/SKR04) -> XBRL-Taxonomie-Positionen
2. Validierung: Pruefung auf Vollstaendigkeit der Mussfelder
3. XBRL-Generierung: Erstellung des XBRL-Instanzdokuments
4. Uebermittlung: Versand ueber ERiC-Schnittstelle
5. Empfangsbestaetigung: Verarbeitung der Rueckmeldung
6. Taxonomie-Updates: Jaehrliche Aktualisierung der Mappings
```

### 8.3 Weitere elektronische Schnittstellen

#### 8.3.1 DATEV-Schnittstelle

- Obwohl die Software als Alternative zu DATEV positioniert ist, ist eine **DATEV-kompatible Export/Import-Schnittstelle** wichtig.
- **DATEV-Buchungsstapel-Format:** Standard fuer den Datenaustausch zwischen Buchfuehrungssoftware und Steuerberatern.
- **DATEV-Connect:** API fuer den Datenaustausch.
- Viele Steuerberater arbeiten mit DATEV -- Kompatibilitaet erleichtert den Wechsel.

#### 8.3.2 Bankenanbindung

- **FinTS/HBCI:** Standard fuer den elektronischen Bankverkehr in Deutschland
- **PSD2/Open Banking:** EU-Richtlinie fuer den Zugang zu Bankkonten ueber APIs
- **MT940/CAMT:** Standardformate fuer elektronische Kontoauszuege

#### 8.3.3 Meldepflicht fuer elektronische Aufzeichnungssysteme

Seit 01.01.2025 muessen elektronische Aufzeichnungssysteme an das Finanzamt gemeldet werden:
- Uebermittlung ueber **"Mein ELSTER"** oder **ERiC-Schnittstelle**
- Frist: **31. Juli 2025**
- Inhalt: Name, Steuernummer, Anzahl der Systeme, Seriennummern, Anschaffungsdatum, ggf. Ausserbetriebnahme

---

## 9. Zusammenfassung und Handlungsempfehlungen

### 9.1 Kritische rechtliche Anforderungen (Must-Have)

Die folgenden Anforderungen sind **zwingend** zu erfuellen, um eine rechtskonforme Software bereitzustellen:

| Nr. | Anforderung | Rechtsgrundlage | Prioritaet |
|-----|-------------|-----------------|------------|
| 1 | Unveraenderbarkeit der Buchungen (Append-Only, Storno statt Loeschung) | GoBD Rz. 58-67, s 239 Abs. 3 HGB | KRITISCH |
| 2 | Lueckenloser Audit Trail / Protokollierung | GoBD Rz. 58-67 | KRITISCH |
| 3 | Verfahrensdokumentation (automatisch generiert) | GoBD Rz. 151-155 | KRITISCH |
| 4 | Z1/Z2/Z3 Datenzugriff fuer Betriebspruefung | s 147 Abs. 6 AO | KRITISCH |
| 5 | E-Bilanz im XBRL-Format | s 5b EStG | KRITISCH |
| 6 | ELSTER/ERiC-Anbindung | Diverse Steuergesetze | KRITISCH |
| 7 | E-Rechnungsverarbeitung (XRechnung, ZUGFeRD) | s 14 UStG, EN 16931 | KRITISCH |
| 8 | DSGVO-konformes Loeschkonzept mit Fristueberwachung | Art. 5, 17 DSGVO | HOCH |
| 9 | Auftragsverarbeitungsvertrag fuer KI-API | Art. 28 DSGVO | HOCH |
| 10 | Human-in-the-Loop fuer KI-Buchungen | Art. 22 DSGVO, s 43 GmbHG, EU AI Act | HOCH |
| 11 | Datensicherungskonzept | GoBD Rz. 103-107 | HOCH |
| 12 | Rollenbasiertes Berechtigungskonzept | GoBD, DSGVO Art. 32 | HOCH |
| 13 | KI-Transparenz und Erklaerbarkeit | EU AI Act Art. 50, GoBD | HOCH |
| 14 | Aufbewahrungsfristen korrekt implementieren (8/10 Jahre) | s 257 HGB, s 147 AO | HOCH |

### 9.2 Empfohlene Zertifizierungen (Should-Have)

| Zertifizierung | Nutzen | Zeitpunkt |
|---------------|--------|-----------|
| **IDW PS 880** | Marktvertrauen, Prueferakzeptanz | Vor Markteinfuehrung |
| **ISO 27001** | IT-Sicherheitsnachweis, Kundenvertrauen | Innerhalb 12 Monaten nach Launch |
| **SOC 2 Type II** | Cloud-Sicherheitsnachweis | Bei SaaS-Bereitstellung |

### 9.3 Architekturempfehlungen aus rechtlicher Sicht

1. **Append-Only-Datenbankschema** fuer Buchungsdaten -- keine UPDATE/DELETE-Operationen auf festgeschriebene Buchungen
2. **Kryptografische Integritaetssicherung** (Hashverkettung) des Buchungsjournals
3. **Mandantentrennung** auf Datenbankebene (DSGVO, Vertraulichkeit)
4. **Datenresidenz in der EU** -- keine Verarbeitung von Finanzdaten ausserhalb des EWR (oder nur mit Angemessenheitsbeschluss/Standardvertragsklauseln)
5. **Exportschnittstellen** fuer Z1/Z2/Z3-Zugriff und DATEV-Kompatibilitaet
6. **Konfidenzbasierte Freigabe-Workflows** fuer KI-Buchungen (HITL)
7. **Umfassende Erklaerungskomponente** fuer jede KI-Entscheidung
8. **Versionierte Verfahrensdokumentation** (automatisch aktualisiert bei Softwareaenderungen)
9. **Automatisierte Fristueberwachung** fuer Aufbewahrungs- und Loeschfristen
10. **Getrennte Speicherung** von Buchhaltungsdaten und KI-Modelldaten

### 9.4 Risikomatrix

| Risiko | Eintrittswahrscheinlichkeit | Schadenshoehe | Massnahme |
|--------|---------------------------|---------------|-----------|
| Verwerfung der Buchfuehrung bei Betriebspruefung | Mittel | Hoch | IDW PS 880, Z1/Z2/Z3-Support, Verfahrensdokumentation |
| DSGVO-Bussgeld (bis 4% des Jahresumsatzes) | Niedrig-Mittel | Sehr hoch | DSFA, AVV, Loeschkonzept, TOMs |
| Geschaeftsfuehrerhaftung (s 43 GmbHG) | Niedrig | Hoch | HITL, Dokumentation, Stichprobenkonzept |
| KI-Fehlbuchungen mit steuerlichen Folgen | Mittel | Mittel-Hoch | HITL, Konfidenz-Schwellwerte, Validierungsregeln |
| EU AI Act Verstoss | Niedrig (bis 08/2026) | Hoch (bis 35 Mio. EUR oder 7% Umsatz) | Transparenz, Dokumentation, Risikomanagement |
| Verlust des GoBD-Konformitaetsnachweis | Mittel | Hoch | Audit Trail, Unveraenderbarkeit, Verfahrensdokumentation |

### 9.5 Zeitlicher Fahrplan aus rechtlicher Sicht

| Phase | Zeitraum | Rechtliche Meilensteine |
|-------|----------|------------------------|
| **MVP** | Monate 1-6 | GoBD-Konformitaet (Unveraenderbarkeit, Audit Trail, Verfahrensdokumentation), DSGVO-Grundlagen (TOMs, Loeschkonzept) |
| **Beta** | Monate 7-12 | ELSTER/ERiC-Integration, E-Bilanz (XBRL), Z1/Z2/Z3-Datenzugriff, HITL-Workflows |
| **Launch** | Monate 13-18 | IDW PS 880 Pruefung, DSFA fuer KI-Komponente, AVV mit KI-Anbieter, E-Rechnungsverarbeitung |
| **Ausbau** | Monate 19-24 | ISO 27001 Zertifizierung, EU AI Act Compliance (vor 08/2026), SOC 2 Type II |

---

## Anhang: Rechtsquellenverzeichnis

### Gesetze und Verordnungen

- **HGB** -- Handelsgesetzbuch (ss 238-263, 264-289, insbesondere s 257)
- **AO** -- Abgabenordnung (ss 140-148, insbesondere ss 145, 146, 146a, 147)
- **GmbHG** -- Gesetz betreffend die Gesellschaften mit beschraenkter Haftung (ss 41, 43)
- **UStG** -- Umsatzsteuergesetz (ss 14, 14a, 18)
- **EStG** -- Einkommensteuergesetz (s 5b)
- **KStG** -- Koerperschaftsteuergesetz
- **GewStG** -- Gewerbesteuergesetz
- **InsO** -- Insolvenzordnung (s 15a)
- **DSGVO** -- Datenschutz-Grundverordnung (Art. 5, 6, 13, 14, 15, 17, 22, 28, 32, 35)
- **KI-VO** -- Verordnung (EU) 2024/1689 (EU AI Act)
- **KassenSichV** -- Kassensicherungsverordnung
- **ProdHaftG** -- Produkthaftungsgesetz
- **BEG IV** -- Viertes Buerokratieentlastungsgesetz (2024)

### Verwaltungsanweisungen und Standards

- **GoBD** -- BMF-Schreiben vom 14.07.2025 (aktualisierte Fassung)
- **IDW PS 880** -- Die Pruefung von Softwareprodukten (Neufassung 01/2022)
- **BSI TR-03153** -- Technische Richtlinie fuer TSE
- **DSFinV-K** -- Digitale Schnittstelle der Finanzverwaltung fuer Kassensysteme
- **BSI IT-Grundschutz** -- IT-Grundschutz-Kompendium
- **ISO/IEC 27001** -- Informationssicherheits-Managementsysteme
- **ISO 25010** -- Softwarequalitaet
- **EN 16931** -- Europaeische Norm fuer elektronische Rechnungen
- **XBRL-Taxonomie 6.8/6.9** -- E-Bilanz Taxonomie der Finanzverwaltung

### Weitergehende Quellen

- Lexware: GoBD-Anforderungen (https://www.lexware.de/wissen/buchhaltung-finanzen/gobd/)
- IHK Muenchen: GoBD Ratgeber (https://www.ihk-muenchen.de/ratgeber/steuern/finanzverwaltung/grundsaetze-elektronische-buchfuehrung-gobd/)
- d.velop: IDW PS 880 (https://www.d-velop.de/blog/compliance/idw-ps-880/)
- dataguard: GoBD und DSGVO Kombination (https://www.dataguard.de/blog/gobd-und-dsgvo)
- ai-act-law.eu: EU AI Act Anhang III (https://ai-act-law.eu/de/anhang/3/)
- Lexware Office: KI in der Steuerkanzlei - Human in the Loop (https://office.lexware.de/blog/ki-in-der-steuerkanzlei-human-in-the-loop/)
- esteuer.de: E-Bilanz Taxonomie FAQ (https://www.esteuer.de/download/taxonomie/FAQ_Version_2025-01.pdf)
- BSI: IT-Grundschutz (https://www.bsi.bund.de/DE/Themen/Unternehmen-und-Organisationen/Standards-und-Zertifizierung/IT-Grundschutz/it-grundschutz_node.html)

---

*Dieser Report dient der Information und Orientierung bei der Softwareentwicklung. Er ersetzt keine individuelle Rechtsberatung. Fuer die konkrete Umsetzung wird empfohlen, einen auf IT-Recht und Steuerrecht spezialisierten Rechtsanwalt hinzuzuziehen.*
