# Steuerliche Anforderungen und Buchungslogik fuer eine GmbH-Buchhaltungssoftware

## Umfassender Report fuer die Entwicklung einer KI-gestuetzten Buchhaltungssoftware

**Erstellt:** 07.02.2026
**Zielgruppe:** Entwicklerteam, Produktdesign, Steuerberater
**Fokus:** Vermoegensverwaltende und operative GmbHs in Deutschland

---

## Inhaltsverzeichnis

1. [Kontenrahmen und Kontensystematik](#1-kontenrahmen-und-kontensystematik)
2. [Doppelte Buchfuehrung - Kernmechanismen](#2-doppelte-buchfuehrung---kernmechanismen)
3. [Umsatzsteuer (UStG)](#3-umsatzsteuer-ustg)
4. [Koerperschaftsteuer (KStG)](#4-koerperschaftsteuer-kstg)
5. [Gewerbesteuer (GewStG)](#5-gewerbesteuer-gewstg)
6. [Jahresabschluss nach HGB](#6-jahresabschluss-nach-hgb)
7. [E-Bilanz](#7-e-bilanz)
8. [Vermoegensverwaltende GmbH - Besonderheiten](#8-vermoegensverwaltende-gmbh---besonderheiten)
9. [Operative GmbH - Besonderheiten](#9-operative-gmbh---besonderheiten)
10. [Besondere Buchungsvorfaelle](#10-besondere-buchungsvorfaelle)
11. [Holding-Strukturen](#11-holding-strukturen)
12. [GoBD-Compliance und Betriebspruefung](#12-gobd-compliance-und-betriebspruefung)

---

## 1. Kontenrahmen und Kontensystematik

### 1.1 SKR03 vs. SKR04: Aufbau und Unterschiede

Die DATEV-Standard-Kontenrahmen SKR03 und SKR04 sind die am weitesten verbreiteten Kontenrahmen in Deutschland. Beide sind vierstellig aufgebaut (Konten 0000-9999).

#### SKR03 - Prozessgliederungsprinzip

Der SKR03 orientiert sich am **betrieblichen Ablauf** (Beschaffung -> Produktion -> Absatz). Die Konten sind nach dem Wertschoepfungsprozess geordnet.

**Kontenklassen SKR03:**

| Klasse | Bezeichnung | Beispiele |
|--------|-------------|-----------|
| 0 | Anlage- und Kapitalkonten | 0200 Geschaefts-/Firmenwert, 0400 Maschinen, 0800 Gezeichnetes Kapital |
| 1 | Finanz- und Privatkonten | 1200 Bank, 1000 Kasse, 1400 Forderungen aus LuL |
| 2 | Abgrenzungskonten | 2000 Rueckstellungen, 2100 Verbindlichkeiten |
| 3 | Wareneingangs-/Bestandskonten | 3000 Einsatzstoffe, 3300 Waren-Eingang 7% USt, 3400 Waren-Eingang 19% USt |
| 4 | Betriebliche Aufwendungen | 4100 Loehne, 4200 Gehaelter, 4500 Kfz-Kosten, 4600 Werbekosten |
| 7 | Bestaende an Erzeugnissen | 7000 Bestandsveraenderungen |
| 8 | Erloeskonten | 8000 Erloese 19% USt, 8300 Erloese 7% USt, 8400 Erloese steuerfrei |
| 9 | Vortrags-/statistische Konten | 9000 Saldenvortraege, 9800 Einkommen-/Koerperschaftsteuer |

#### SKR04 - Abschlussgliederungsprinzip

Der SKR04 orientiert sich am **Aufbau des Jahresabschlusses** (Bilanz und GuV). Dadurch ist die Zuordnung der Konten zur Bilanz bzw. GuV direkt ersichtlich.

**Kontenklassen SKR04:**

| Klasse | Bezeichnung | Beispiele |
|--------|-------------|-----------|
| 0 | Anlagevermoegen | 0100 Grundstuecke, 0440 Maschinen, 0027 EDV-Software |
| 1 | Umlaufvermoegen | 1200 Forderungen aus LuL, 1800 Bank, 1600 Kasse |
| 2 | Eigenkapital / Rueckstellungen | 2000 Gezeichnetes Kapital, 2900 Rueckstellungen |
| 3 | Verbindlichkeiten / RAP | 3300 Verbindlichkeiten aus LuL, 3500 Sonstige Verbindlichkeiten |
| 4 | Erloese / Betriebliche Ertraege | 4000 Umsatzerloese 19%, 4300 Umsatzerloese 7% |
| 5 | Material-/Personalaufwand | 5000 Materialaufwand, 5200 Personalaufwand |
| 6 | Sonstige betriebliche Aufwendungen | 6300 Sonstige betriebl. Aufwendungen, 6600 Abschreibungen |
| 7 | Weitere Aufwendungen/Ertraege | 7000 Zinsertraege, 7300 Zinsaufwendungen |
| 8 | Ausserordentliche Ertraege/Aufwendungen | 8000-8999 |
| 9 | Vortrags-/statistische Konten | 9000 Saldenvortraege |

#### Empfehlung fuer die Software

**Die Software sollte beide Kontenrahmen (SKR03 und SKR04) vollstaendig unterstuetzen.** Da SKR04 didaktisch naheliegender ist (Bilanzorientierung) und in der Ausbildung dominiert, empfiehlt es sich, SKR04 als Standard vorzuschlagen. Die Wahl muss jedoch beim Nutzer liegen.

**Implementierungshinweis:** Intern sollte eine Mapping-Tabelle SKR03 <-> SKR04 hinterlegt werden, um bei Kontenrahmenwechsel automatisch umstellen zu koennen.

### 1.2 Individualisierung des Kontenrahmens

Die Software muss dem Nutzer erlauben:
- Eigene Konten innerhalb der vorgegebenen Kontenklassen anzulegen
- Konten zu deaktivieren (nicht zu loeschen, da historische Buchungen darauf verweisen)
- Konten mit individuellen Bezeichnungen zu versehen
- Konten mit Steuerautomatiken (Steuerschluessel) zu verknuepfen
- Konten als Automatikkonten zu kennzeichnen (automatische USt-Berechnung)

### 1.3 Automatische Kontenzuordnung (Kontierungslogik)

Die KI-Komponente der Software muss in der Lage sein, aus Belegen (Rechnungen, Kontoauszuegen) automatisch den passenden Buchungssatz abzuleiten. Dazu benoetigt sie:

1. **Regelbasierte Zuordnung:**
   - Wiederkehrende Geschaeftsvorfaelle anhand von Empfaenger/Absender, IBAN, Verwendungszweck erkennen
   - Standardbuchungen: Bank an Erloese, Aufwand an Bank, etc.
   - Steuerautomatik: USt-Satz aus Konto ableiten

2. **KI-gestuetzte Zuordnung:**
   - Textanalyse von Rechnungspositionen
   - Lernfaehige Zuordnung basierend auf historischen Buchungen
   - Confidence Score fuer automatische vs. manuelle Freigabe

3. **Pflichtfelder bei der Buchung:**
   - Belegdatum
   - Buchungsdatum
   - Buchungstext
   - Sollkonto / Habenkonto
   - Betrag (netto, brutto, USt)
   - Steuerschluessel
   - Belegnummer
   - Optional: Kostenstelle, Kostentraeger, Projekt

---

## 2. Doppelte Buchfuehrung - Kernmechanismen

### 2.1 Soll/Haben-Logik

**Fundamentale Regel:** Jede Buchung besteht aus mindestens einer Soll- und einer Habenbuchung. Die Summe der Soll-Betraege muss immer gleich der Summe der Haben-Betraege sein.

#### Aktivkonten (Vermoegen)
- **Zugang** = Soll (Mehrung)
- **Abgang** = Haben (Minderung)

#### Passivkonten (Kapital/Schulden)
- **Zugang** = Haben (Mehrung)
- **Abgang** = Soll (Minderung)

#### Aufwandskonten
- **Aufwand** = Soll (Mehrung)
- **Korrektur** = Haben (Minderung)

#### Ertragskonten
- **Ertrag** = Haben (Mehrung)
- **Korrektur** = Soll (Minderung)

### 2.2 Buchungssaetze und deren Struktur

#### Einfacher Buchungssatz
Ein Sollkonto an ein Habenkonto.

**Beispiel: Barkauf von Bueromaterial (SKR03)**
```
Soll: 4930 Buerokosten                     50,00 EUR
Haben: 1000 Kasse                          50,00 EUR
```

#### Zusammengesetzter Buchungssatz
Mehrere Soll- und/oder Habenkonten.

**Beispiel: Eingangsrechnung mit Vorsteuer (SKR03)**
```
Soll: 3400 Wareneinkauf 19% USt          1.000,00 EUR
Soll: 1576 Abziehbare Vorsteuer 19%        190,00 EUR
Haben: 1600 Verbindlichkeiten aus LuL     1.190,00 EUR
```

**Implementierungshinweis:** Die Software muss Splitbuchungen (1:n und n:m) unterstuetzen. Die Soll-Haben-Gleichheit ist als harte Validierungsregel zu implementieren.

### 2.3 Eroeffnungs- und Abschlussbuchungen

#### Eroeffnungsbilanz (EB)

Zu Beginn jedes Geschaeftsjahres werden die Schlusssalden des Vorjahres als Anfangsbestaende vorgetragen:

```
Aktive Bestandskonten:
Soll: [jeweiliges Aktivkonto]      an Haben: 9000 Saldenvortraege Sachkonten

Passive Bestandskonten:
Soll: 9000 Saldenvortraege Sachkonten an Haben: [jeweiliges Passivkonto]
```

**Beispiel SKR03:**
```
Soll: 1200 Bank                         50.000,00 EUR
Haben: 9000 Saldenvortraege             50.000,00 EUR

Soll: 9000 Saldenvortraege              25.000,00 EUR
Haben: 0800 Gezeichnetes Kapital        25.000,00 EUR
```

#### Abschlussbuchungen (Jahresende)

1. **Aufwands- und Ertragskonten abschliessen** (GuV-Abschluss):
```
Soll: 8000 Erloese (Saldo)      an Haben: 9999 GuV-Konto
Soll: 9999 GuV-Konto            an Haben: 4000 Aufwendungen (Saldo)
```

2. **GuV-Saldo ins Eigenkapital ueberleiten:**
```
bei Gewinn:
Soll: 9999 GuV-Konto            an Haben: 0860 Gewinnvortrag

bei Verlust:
Soll: 0868 Verlustvortrag       an Haben: 9999 GuV-Konto
```

3. **Bestandskonten abschliessen** (Schlussbilanz):
```
Soll: 9998 Schlussbilanzkonto    an Haben: [Aktivkonten]
Soll: [Passivkonten]            an Haben: 9998 Schlussbilanzkonto
```

**Implementierungshinweis:** Die Software muss einen automatisierten Jahresabschluss-Assistenten bereitstellen, der diese Buchungen korrekt generiert. Ein manuelles Buchen des Jahresabschlusses sollte dennoch moeglich sein.

### 2.4 Nebenbuecher

#### Debitorenbuchhaltung (Kundenbuch)

- Eigener Kontenkreis: z.B. 10000-69999 (DATEV-Standard) oder frei definierbar
- Jeder Debitor erhaelt ein eigenes Personenkonto
- Buchungen auf Debitorenkonten werden automatisch auf das Sammelkonto "Forderungen aus Lieferungen und Leistungen" (SKR03: 1400 / SKR04: 1200) uebernommen
- OP-Verwaltung (Offene Posten): Zuordnung von Zahlungseingaengen zu offenen Rechnungen

**Beispiel: Ausgangsrechnung an Debitor (SKR03)**
```
Soll: 10001 Debitor Meier GmbH        1.190,00 EUR
Haben: 8400 Erloese 19%               1.000,00 EUR
Haben: 1776 Umsatzsteuer 19%            190,00 EUR
```

**Beispiel: Zahlungseingang Debitor (SKR03)**
```
Soll: 1200 Bank                        1.190,00 EUR
Haben: 10001 Debitor Meier GmbH        1.190,00 EUR
```

#### Kreditorenbuchhaltung (Lieferantenbuch)

- Eigener Kontenkreis: z.B. 70000-99999 (DATEV-Standard) oder frei definierbar
- Jeder Kreditor erhaelt ein eigenes Personenkonto
- Buchungen werden auf das Sammelkonto "Verbindlichkeiten aus Lieferungen und Leistungen" (SKR03: 1600 / SKR04: 3300) uebernommen

**Beispiel: Eingangsrechnung Kreditor (SKR03)**
```
Soll: 4980 Sonstige Aufwendungen       1.000,00 EUR
Soll: 1576 Vorsteuer 19%                 190,00 EUR
Haben: 70001 Kreditor Mueller AG        1.190,00 EUR
```

#### Anlagenbuchhaltung

- Jedes Wirtschaftsgut erhaelt einen eigenen Datensatz (Anlagenstamm)
- Pflichtangaben: Bezeichnung, Anschaffungsdatum, AK/HK, Nutzungsdauer, AfA-Methode
- Automatische Berechnung der Abschreibungen (monatlich/jaehrlich)
- Verwaltung von Zu-/Abgaengen, Umbuchungen, Teilwertabschreibungen
- Erstellung des Anlagenspiegels (Pflicht in E-Bilanz)

**Implementierungshinweis:** Die Anlagenbuchhaltung muss als eigenes Nebenbuch gefuehrt werden, das automatisch Buchungssaetze fuer die Hauptbuchhaltung generiert (insbesondere Abschreibungsbuchungen).

---

## 3. Umsatzsteuer (UStG)

### 3.1 Grundlagen und Steuersaetze

**Aktuelle Steuersaetze (Stand 2026):**
- Regelsteuersatz: **19%** (ss 12 Abs. 1 UStG)
- Ermaessigter Steuersatz: **7%** (ss 12 Abs. 2 UStG)
- Seit 01.01.2026: Speisen in der Gastronomie dauerhaft 7% (Getraenke weiterhin 19%)

**Steuerbefreiungen** (Auswahl, ss 4 UStG):
- Ausfuhrlieferungen (ss 4 Nr. 1a UStG)
- Innergemeinschaftliche Lieferungen (ss 4 Nr. 1b UStG)
- Finanz- und Versicherungsleistungen (ss 4 Nr. 8, 10, 11 UStG)
- Vermietung und Verpachtung von Grundstuecken (ss 4 Nr. 12 UStG, Option zur USt moeglich)
- Aerztliche Leistungen (ss 4 Nr. 14 UStG)

### 3.2 Umsatzsteuer-Voranmeldung (UStVA)

**Abgabefristen:**
- **Monatlich:** Wenn die USt-Zahllast im Vorjahr > 7.500 EUR
- **Vierteljaehrlich:** Wenn die USt-Zahllast im Vorjahr zwischen 1.000 EUR und 7.500 EUR
- **Keine UStVA (nur Jahreserklaerung):** Wenn die USt-Zahllast im Vorjahr < 1.000 EUR
- **Im Gruendungsjahr und Folgejahr:** Immer monatlich

**Abgabetermin:** 10. des Folgemonats (bzw. Folgequartals). Dauerfristverlaengerung moeglich (ss 46-48 UStDV): +1 Monat, erfordert bei Monatszahlern Sondervorauszahlung von 1/11 der Vorjahres-USt.

**Struktur der UStVA (wesentliche Kennzahlen):**

| KZ | Bezeichnung | Quelle |
|----|-------------|--------|
| 81 | Steuerpflichtige Lieferungen/Leistungen 19% (Bemessungsgrundlage) | Erloeskonten 19% |
| 86 | Steuerpflichtige Lieferungen/Leistungen 7% (Bemessungsgrundlage) | Erloeskonten 7% |
| 21 | Nicht steuerbare sonstige Leistungen (ss 18b S.1 Nr. 2 UStG) | EU-Dienstleistungen |
| 41 | Innergemeinschaftliche Lieferungen (ss 4 Nr. 1b) an Abnehmer mit USt-IdNr | igl. Lieferungen |
| 46 | Nicht steuerbare Umsaetze (Leistungsort nicht im Inland) | Drittland-Leistungen |
| 67 | Vorsteuerbetraege aus Rechnungen | Vorsteuerkonto |
| 61 | Vorsteuer aus innergemeinschaftlichem Erwerb | VSt aus igE |
| 66 | USt-Vorauszahlung (Zahllast) | Berechnet: USt - VSt |
| 83 | Verbleibende Vorauszahlung | Nettobetrag |

**Implementierungshinweis:** Die Software muss automatisch aus den Buchungen die UStVA-Kennzahlen ermitteln und das ELSTER-XML generieren. Die Zuordnung der Konten zu den Kennzahlen erfolgt ueber die Steuerschluessel (USt-Codes).

### 3.3 USt-Jahreserklaerung

Die Jahreserklaerung fasst alle UStVA-Zeitraeume zusammen und enthaelt zusaetzlich:
- Jahressummen aller Umsaetze nach Steuersaetzen
- Gesamte abziehbare Vorsteuer
- Differenz zu den geleisteten Vorauszahlungen (Nachzahlung/Erstattung)
- Angaben zu steuerfreien Umsaetzen
- Angaben zu innergemeinschaftlichen Erwerben

**Implementierungshinweis:** Die Software muss einen Abgleich der Jahreserklaerung mit den Summen der UStVA ermoeglchen. Differenzen muessen erkannt und erklaert werden.

### 3.4 Vorsteuerabzug und Voraussetzungen

**Voraussetzungen (ss 15 UStG):**
1. Ordnungsgemaesse Rechnung nach ss 14, 14a UStG vorhanden
2. Leistung fuer das Unternehmen bezogen
3. Leistung wurde ausgefuehrt
4. Keine Vorsteuer-Ausschlusstatbestaende (ss 15 Abs. 1a, 2 UStG)

**Pflichtangaben einer Rechnung (ss 14 Abs. 4 UStG):**
- Name und Anschrift des leistenden Unternehmers
- Name und Anschrift des Leistungsempfaengers
- Steuernummer oder USt-IdNr. des leistenden Unternehmers
- Ausstellungsdatum
- Fortlaufende Rechnungsnummer
- Menge und Art der Lieferung / Umfang der Leistung
- Zeitpunkt der Lieferung/Leistung
- Entgelt (netto)
- Steuersatz und Steuerbetrag / Hinweis auf Steuerbefreiung
- Im Voraus vereinbarte Minderungen (Skonti, Rabatte)

**Kleinbetragsrechnungen (ss 33 UStDV, bis 250 EUR brutto):**
- Vereinfachte Anforderungen (kein Empfaenger, keine Rechnungsnummer noetig)

**Implementierungshinweis:** Die Software sollte eingehende Rechnungen auf Pflichtangaben pruefen (OCR + KI-Analyse), bevor ein Vorsteuerabzug gewahrt wird. Fehlende Pflichtangaben muessen dem Nutzer gemeldet werden.

### 3.5 ss 13b UStG - Reverse Charge (Umkehr der Steuerschuldnerschaft)

Bei bestimmten Umsaetzen schuldet nicht der leistende Unternehmer die USt, sondern der **Leistungsempfaenger**:

**Anwendungsfaelle (ss 13b Abs. 2 UStG):**
- Werklieferungen/Dienstleistungen eines im Ausland ansaessigen Unternehmers
- Bauleistungen (ss 13b Abs. 2 Nr. 4 UStG)
- Gebaeude-/Grundstuecksreinigung (ss 13b Abs. 2 Nr. 8 UStG)
- Lieferungen von Mobilfunkgeraeten, Tablets, Spielekonsolen (>= 5.000 EUR, ss 13b Abs. 2 Nr. 10 UStG)
- Lieferung von Metallen (Anlage 4 UStG)
- Lieferung von Gold (ss 13b Abs. 2 Nr. 9 UStG)

**Buchungslogik Reverse Charge (SKR03):**

Eingangsrechnung aus EU ohne USt (z.B. Beratung aus Frankreich, 5.000 EUR netto):
```
Soll: 3123 Sonstige Leistungen EU ss 13b (netto)    5.000,00 EUR
Soll: 1577 Abziehbare Vorsteuer ss 13b 19%            950,00 EUR
Haben: 1787 Umsatzsteuer ss 13b 19%                    950,00 EUR
Haben: 70xxx Kreditor (Lieferant Frankreich)          5.000,00 EUR
```

**Ergebnis:** USt und VSt heben sich auf (sofern voller VSt-Abzug). In der UStVA erscheint der Betrag bei KZ 46 (Bemessungsgrundlage) und KZ 47 (Steuer) sowie bei KZ 67 (Vorsteuer).

**Implementierungshinweis:** Die Software muss automatisch erkennen, wann ss 13b anzuwenden ist (basierend auf Herkunftsland des Lieferanten, Leistungsart, Rechnungsbetrag). Es muessen spezielle Automatikkonten mit ss13b-Steuerschluessel bereitgestellt werden.

### 3.6 Innergemeinschaftliche Lieferungen und Erwerbe

#### Innergemeinschaftliche Lieferung (igL)

Eine Lieferung an einen anderen EU-Unternehmer ist steuerfrei, wenn:
- Lieferung gelangt physisch in einen anderen EU-Staat
- Erwerber hat gueltige USt-IdNr.
- Gelangensbestaetigung / Frachtnachweis liegt vor

**Buchung igL (SKR03):**
```
Soll: 10xxx Debitor EU                            10.000,00 EUR
Haben: 8125 Steuerfreie igL (ss 4 Nr. 1b)         10.000,00 EUR
```

**UStVA:** KZ 41 (Bemessungsgrundlage)
**Zusammenfassende Meldung:** Pflicht!

#### Innergemeinschaftlicher Erwerb (igE)

Beim Erwerb aus einem anderen EU-Staat schuldet der Erwerber die Erwerbsteuer:

**Buchung igE (SKR03):**
```
Soll: 3425 Innergemeinschaftlicher Erwerb 19%     10.000,00 EUR
Soll: 1574 Abziehbare Vorsteuer igE 19%            1.900,00 EUR
Haben: 1774 Umsatzsteuer igE 19%                   1.900,00 EUR
Haben: 70xxx Kreditor EU-Lieferant                 10.000,00 EUR
```

**UStVA:** KZ 89 (Bemessungsgrundlage igE), KZ 93 (Steuer)

### 3.7 One-Stop-Shop (OSS)

Seit 01.07.2021 gilt das OSS-Verfahren fuer:
- **Fernverkauf an Endverbraucher** in andere EU-Staaten (B2C)
- Schwelle: 10.000 EUR Gesamtumsatz EU-weit (darunter Besteuerung im Inland)
- Registrierung in Deutschland beim BZSt

**Buchungslogik OSS:**
```
Soll: 10xxx Debitor Endkunde Frankreich           1.000,00 EUR
Haben: 8338 Erloese OSS Frankreich 20%              833,33 EUR
Haben: 1779 Umsatzsteuer OSS Frankreich 20%         166,67 EUR
```

**Implementierungshinweis:** Die Software muss pro EU-Land den jeweils geltenden USt-Satz hinterlegen und eigene Erloeskonten je Land fuehren. Die OSS-Meldung muss vierteljaehrlich ueber das BZSt-Portal (oder API) uebermittelt werden.

### 3.8 Zusammenfassende Meldung (ZM)

**Meldepflicht bei:**
- Innergemeinschaftlichen Lieferungen
- Innergemeinschaftlichen Dreiecksgeschaeften
- Grenzueberschreitenden Dienstleistungen (ss 3a Abs. 2 UStG) in der EU

**Abgabefrist:** Bis zum 25. des Folgemonats (bei monatlicher Abgabe)

**Inhalt:**
- USt-IdNr. jedes Kunden
- Summe der Lieferungen/Leistungen an diesen Kunden im Meldezeitraum

**Implementierungshinweis:** Die Software muss USt-IdNr. der Kunden validieren (ueber das VIES-System der EU-Kommission) und automatisch die ZM aus den Buchungsdaten generieren.

### 3.9 Vorbereitung auf Umsatzsteuer-Sonderpruefung

Die Software sollte folgende Pruefungshilfen bereitstellen:
- Vollstaendige Belegliste aller Vorsteuerbetraege mit Rechnungsnachweisen
- Abstimmung USt-Konten zu UStVA-Summen
- Nachweis der Vorsteuer-Abzugsberechtigung
- Gelangensbestaetigungen fuer igL
- USt-IdNr.-Validierungsprotokolle
- Reverse-Charge-Pruefprotokolle

---

## 4. Koerperschaftsteuer (KStG)

### 4.1 Ermittlung des zu versteuernden Einkommens

Die GmbH ist als Kapitalgesellschaft **unbeschraenkt koerperschaftsteuerpflichtig** (ss 1 Abs. 1 Nr. 1 KStG).

**Steuersaetze (Stand 2026):**
- Koerperschaftsteuer: **15%** (ss 23 Abs. 1 KStG)
- Solidaritaetszuschlag: **5,5% auf die KSt** = effektiv 0,825% des Einkommens
- **Geplante Absenkung ab 2028:** Schrittweise von 15% auf 10% bis 2032

**Schema zur Einkommensermittlung:**

```
  Jahresueberschuss/-fehlbetrag laut Handelsbilanz (GuV)
+ Nicht abzugsfaehige Betriebsausgaben (ss 10 KStG)
+ Verdeckte Gewinnausschuettungen (ss 8 Abs. 3 S. 2 KStG)
- Verdeckte Einlagen
+/- Andere ausserbilanzmaeessige Korrekturen
- Steuerfreie Beteiligungsertraege (ss 8b KStG)
- Investitionsabzugsbetraege (ss 7g EStG)
+/- Hinzurechnungen/Kuerzungen (GewSt-Ermittlung separat)
= Einkommen vor Verlustabzug
- Verlustabzug (ss 10d EStG i.V.m. ss 8 Abs. 1 KStG)
= Zu versteuerndes Einkommen
x 15% KSt
+ 5,5% SolZ auf KSt
= Koerperschaftsteuer-Belastung
```

### 4.2 Nicht abzugsfaehige Betriebsausgaben (ss 10 KStG)

Folgende Aufwendungen duerfen den Gewinn **nicht mindern**:

| Aufwand | Gesetzesgrundlage | Erlaeuterung |
|---------|-------------------|--------------|
| Aufwendungen fuer die Erfuellung satzungsmaessiger Zwecke | ss 10 Nr. 1 KStG | Bei gemeinnuetzigen Koerperschaften |
| Koerperschaftsteuer selbst | ss 10 Nr. 2 KStG | KSt ist keine Betriebsausgabe |
| Umsatzsteuer auf verdeckte Gewinnausschuettungen | ss 10 Nr. 2 KStG | |
| 50% der Aufsichtsratverguetungen | ss 10 Nr. 4 KStG | Nur halber Abzug |
| Zuwendungen aller Art an Gesellschafter (soweit vGA) | ss 8 Abs. 3 S. 2 KStG | Verdeckte Gewinnausschuettungen |
| Geldbussen, Ordnungs-/Verwarnungsgelder | ss 10 Nr. 3 KStG | Kein BA-Abzug |
| Zuwendungen i.S.d. ss 9 Abs. 1 Nr. 2 KStG | ss 9 Abs. 1 Nr. 2 KStG | Spenden (mit Hoechstbetrag) |

**Buchungsbeispiel - Nicht abzugsfaehige Aufwendungen (SKR03):**
```
Gebucht als Aufwand:
Soll: 4396 Steuerlich nicht abziehbare Betriebsausgaben    10.000,00 EUR
Haben: 1200 Bank                                           10.000,00 EUR

Alternativ: Geldbusse
Soll: 4398 Nicht abzugsfaehige Geldbussen                   5.000,00 EUR
Haben: 1200 Bank                                            5.000,00 EUR
```

**Implementierungshinweis:** Die Software muss Konten markieren, die steuerlich nicht abzugsfaehig sind. Bei der Steuerberechnung werden diese Betraege dem Gewinn ausserbilanziell wieder hinzugerechnet.

### 4.3 Verdeckte Gewinnausschuettungen (vGA) - ss 8 Abs. 3 S. 2 KStG

Eine vGA liegt vor, wenn die GmbH einem Gesellschafter einen Vorteil zuwendet, den ein ordentlicher Geschaeftsleiter einem Nichtgesellschafter nicht gewaehrt haette.

**Typische Faelle:**
- Ueberhoeetes Geschaeftsfuehrergehalt
- Zu niedriger Mietzins bei Anmietung vom Gesellschafter
- Zu hohe Zinsen auf Gesellschafterdarlehen
- Private Nutzung von GmbH-Vermoegen ohne Versteuerung
- Ueberhoeete Beraterverguetung an nahestehende Personen

**Buchungsbeispiel - Erkannte vGA:**

Nachtraegliche Korrektur eines ueberhoeeten GF-Gehalts (10.000 EUR zu viel):
```
Die vGA wird NICHT in der Handelsbilanz korrigiert, sondern nur
ausserbilanziell bei der Steuerberechnung hinzugerechnet.

Steuerliches Schema:
Handelsbilanzergebnis:         100.000 EUR
+ vGA (ueberhoeetes GF-Gehalt):  10.000 EUR
= Einkommen KSt-Ermittlung:    110.000 EUR
```

**Implementierungshinweis:** Die Software sollte Warnhinweise geben, wenn:
- Geschaeftsuehergehalter ueber Vergleichswerten liegen
- Transaktionen mit nahestehenden Personen gebucht werden
- Zinssaetze fuer Gesellschafterdarlehen deutlich vom Marktniveau abweichen

### 4.4 Verdeckte Einlagen

Eine verdeckte Einlage liegt vor, wenn ein Gesellschafter der GmbH einen Vermoegensvorteil zuwendet, ohne dass eine offene Einlage beschlossen wurde.

**Typische Faelle:**
- Gesellschafter ueberlaesst der GmbH ein Wirtschaftsgut unter Wert
- Gesellschafter verzichtet auf eine werthaltige Forderung gegen die GmbH
- Gesellschafter zahlt Aufwendungen der GmbH

**Buchung bei Forderungsverzicht des Gesellschafters (SKR03):**
```
Soll: 0740 Verbindlichkeiten gg. Gesellschaftern     50.000,00 EUR
Haben: 0855 Einlagen (verdeckte Einlagen)             50.000,00 EUR
```

**Steuerliche Behandlung:**
- Bei der GmbH: Erhoehung des steuerlichen Einlagekontos (ss 27 KStG)
- Keine Erhoehung des zu versteuernden Einkommens
- Beim Gesellschafter: Erhoehung der Anschaffungskosten der Beteiligung

### 4.5 Verlustabzug (ss 8c, ss 8d KStG)

#### ss 8c KStG - Verlustabzugsbeschraenkung bei Anteilsuebertragungen

| Anteilsuebertragung | Rechtsfolge |
|---------------------|-------------|
| > 50% innerhalb von 5 Jahren an einen Erwerber | **Vollstaendiger Wegfall** des Verlustvortrags, Verlustabzugs und lfd. Verlustes |

**Hinweis:** Die fruehere Regelung zum anteiligen Wegfall bei 25-50% Uebertragung wurde vom BVerfG fuer verfassungswidrig erklaert (BVerfG, Beschluss vom 29.03.2017 - 2 BvL 6/11).

**Ausnahmen (Konzernklausel, Stille-Reserven-Klausel):**
- Konzernklausel (ss 8c Abs. 1 S. 4 KStG): Uebertragung innerhalb eines Konzerns
- Stille-Reserven-Klausel (ss 8c Abs. 1 S. 5-8 KStG): Soweit stille Reserven vorhanden sind, bleibt der Verlust erhalten

#### ss 8d KStG - Fortfuehrungsgebundener Verlustvortrag

Auf Antrag bleibt der Verlustvortrag trotz schaedlichem Beteiligungserwerb erhalten, wenn:
- Der Geschaeftsbetrieb seit mindestens 3 Jahren besteht
- Der Geschaeftsbetrieb **unveraendert fortgefuehrt** wird
- Keine schaedlichen Ereignisse eintreten (Einstellung, Ruhendstellung, Branchenwechsel, Zugang Betriebsvermoegen > Eigenkapital, Beteiligung an Mitunternehmerschaft)

**Implementierungshinweis:** Die Software muss bei Gesellschafterwechseln automatisch pruefen, ob die 50%-Grenze ueberschritten wird, und den Nutzer auf moegliche Verlustgefahren hinweisen.

### 4.6 Organschaft (ss 14-19 KStG)

Bei einer koerperschaftsteuerlichen Organschaft wird das Einkommen der Organgesellschaft (Tochter) der Organtraegerin (Mutter) zugerechnet.

**Voraussetzungen:**
- Gewinnabfuehrungsvertrag (GAV) nach ss 291 AktG
- Finanzielle Eingliederung (Stimmrechtsmehrheit)
- GAV mindestens 5 Jahre Laufzeit
- Tatsaechliche Durchfuehrung

**Buchung Gewinnabfuehrung bei der Organgesellschaft (SKR03):**
```
Soll: 2600 Aufwendungen aus Gewinnabfuehrung       100.000,00 EUR
Haben: 0740 Verbindlichkeiten gg. Organtraeger      100.000,00 EUR
```

**Buchung bei der Organtraegerin (SKR03):**
```
Soll: 1300 Forderungen gg. Organgesellschaft       100.000,00 EUR
Haben: 2710 Ertraege aus Gewinnabfuehrung           100.000,00 EUR
```

---

## 5. Gewerbesteuer (GewStG)

### 5.1 Berechnung der Gewerbesteuer

**GmbHs sind stets gewerbesteuerpflichtig** (ss 2 Abs. 2 GewStG). Ein Freibetrag von 24.500 EUR gilt nur fuer Personenunternehmen, **nicht** fuer Kapitalgesellschaften.

**Berechnungsschema:**

```
  Gewinn aus Gewerbebetrieb (ss 7 GewStG)
  = Gewinn lt. KSt-Veranlagung (handelsrechtlicher Gewinn mit stl. Korrekturen)
+ Hinzurechnungen (ss 8 GewStG)
- Kuerzungen (ss 9 GewStG)
= Gewerbeertrag (abgerundet auf volle 100 EUR)
x Steuermesszahl: 3,5% (ss 11 Abs. 2 GewStG)
= Steuermessbetrag
x Hebesatz der Gemeinde (mindestens 200%, typisch 300-500%)
= Gewerbesteuer
```

**Beispielrechnung:**
```
Gewinn:                           500.000 EUR
+ Hinzurechnungen (z.B. Zinsen):   30.000 EUR
- Kuerzungen (z.B. Grundbesitz):  -10.000 EUR
= Gewerbeertrag:                   520.000 EUR (abgerundet)
x Steuermesszahl 3,5%:              18.200 EUR
x Hebesatz 400%:                     72.800 EUR Gewerbesteuer
```

**Effektiver GewSt-Satz:** Hebesatz x 3,5% / 100
- Bei 400% Hebesatz: 14,0%
- Bei 450% Hebesatz: 15,75%
- Bei 200% Hebesatz: 7,0%

### 5.2 Hinzurechnungen (ss 8 GewStG)

Die Hinzurechnungen sollen eine "objektive" Besteuerung der Ertragskraft sicherstellen, unabhaengig von der Finanzierungsform.

**ss 8 Nr. 1 GewStG - Finanzierungsentgelte:**

Summe aller Finanzierungsentgelte:
| Art | Hinzurechnungsquote |
|-----|---------------------|
| a) Entgelte fuer Schulden (Zinsen) | 100% |
| b) Renten und dauernde Lasten | 100% |
| c) Gewinnanteile stiller Gesellschafter | 100% |
| d) Miet-/Pachtzinsen fuer bewegliche WG | 20% |
| e) Miet-/Pachtzinsen fuer unbewegliche WG | 50% |
| f) Lizenzen | 25% |

Von der **Summe** wird ein Freibetrag von **200.000 EUR** abgezogen. Vom verbleibenden Betrag werden **25%** hinzugerechnet.

**Berechnungsbeispiel Finanzierungsentgelte:**
```
Zinsen (100%):                     100.000 EUR
Miete Buero (50% von 60.000):      30.000 EUR
Leasing Kfz (20% von 12.000):       2.400 EUR
Lizenzen (25% von 20.000):          5.000 EUR
                                   ___________
Summe Finanzierungsentgelte:       137.400 EUR
- Freibetrag:                     -200.000 EUR
= Hinzurechnungsbetrag:                 0 EUR (da < 200.000 EUR)
```

Wenn die Summe den Freibetrag uebersteigt:
```
Summe Finanzierungsentgelte:       300.000 EUR
- Freibetrag:                     -200.000 EUR
= Ueberschreitungsbetrag:         100.000 EUR
x 25% Hinzurechnungsquote:         25.000 EUR = Hinzurechnung
```

**Weitere Hinzurechnungen:**
- ss 8 Nr. 5 GewStG: Gewinnanteile aus Beteiligungen (soweit nach ss 9 Nr. 2a/7/8 GewStG gekuerzt)
- ss 8 Nr. 8 GewStG: Spenden (soweit als Kuerzung nach ss 9 Nr. 5 beruecksichtigt)
- ss 8 Nr. 12 GewStG: Gewinnminderungen aus Beteiligungen an Personengesellschaften

### 5.3 Kuerzungen (ss 9 GewStG)

| Nr. | Kuerzung | Erlaeuterung |
|-----|----------|--------------|
| 1 S.1 | 1,2% des Einheitswerts des Grundbesitzes | Pauschale Kuerzung fuer betrieblichen Grundbesitz |
| 1 S.2 | Erweiterte Kuerzung | Bei reiner Grundstuecksverwaltung: kompletter GewSt-Anteil |
| 2a | Gewinnanteile aus Beteiligungen (>= 15%) | An anderen Koerperschaften |
| 7 | Gewinnanteile aus ausl. Betriebsstaetten | |
| 5 | Spenden (mit Hoechstbetrag) | |

**Besonders wichtig fuer vermoegensverwaltende GmbHs:**
Die **erweiterte Kuerzung** (ss 9 Nr. 1 S. 2 GewStG) ist fuer rein vermoegensverwaltende Immobilien-GmbHs relevant: Hier kann die komplette Gewerbesteuer auf Mieteinkuenfte entfallen, sofern die GmbH **ausschliesslich** eigenen Grundbesitz verwaltet und nutzt.

### 5.4 Zerlegung (ss 28 GewStG)

Unterhalt eine GmbH Betriebsstaetten in mehreren Gemeinden, wird der Steuermessbetrag auf die Gemeinden aufgeteilt (Zerlegung). Zerlegungsmassstab ist grundsaetzlich das Verhaeltnis der Arbeitslohn-Summen.

### 5.5 Gewerbesteuerrueckstellung

Die GewSt ist fuer KSt-Zwecke **nicht** als Betriebsausgabe abzugsfaehig (ss 4 Abs. 5b EStG). Handelsrechtlich ist sie aber als Aufwand zu buchen und eine Rueckstellung zu bilden.

**Buchung GewSt-Rueckstellung (SKR03):**
```
Soll: 4320 Gewerbesteuer                           72.800,00 EUR
Haben: 0956 Gewerbesteuerrueckstellung              72.800,00 EUR
```

**Nachzahlung/Erstattung bei Bescheid:**
```
Nachzahlung:
Soll: 0956 Gewerbesteuerrueckstellung              72.800,00 EUR
Haben: 1200 Bank                                    73.500,00 EUR
Soll: 4320 Gewerbesteuer (Nachzahlung)                700,00 EUR

Erstattung:
Soll: 0956 Gewerbesteuerrueckstellung              72.800,00 EUR
Soll: 1200 Bank                                      1.000,00 EUR
Haben: 4320 Gewerbesteuer (Erstattung)                1.000,00 EUR
(Aufloesung der zu hohen Rueckstellung = Ertrag)
```

**Implementierungshinweis:** Die Software muss den Hebesatz der jeweiligen Gemeinde kennen und die GewSt automatisch berechnen koennen. Die Berechnung ist iterativ, da die GewSt-Rueckstellung den handelsrechtlichen Gewinn und damit die Bemessungsgrundlage beeinflusst.

---

## 6. Jahresabschluss nach HGB

### 6.1 Groessenklassen (ss 267, 267a HGB)

| Kriterium | Kleinst | Klein | Mittelgross | Gross |
|-----------|---------|-------|-------------|-------|
| Bilanzsumme | <= 450.000 EUR | <= 7.500.000 EUR | <= 25.000.000 EUR | > 25.000.000 EUR |
| Umsatzerloese | <= 900.000 EUR | <= 15.000.000 EUR | <= 50.000.000 EUR | > 50.000.000 EUR |
| Arbeitnehmer | <= 10 | <= 50 | <= 250 | > 250 |

Mindestens 2 von 3 Kriterien muessen an 2 aufeinanderfolgenden Bilanzstichtagen ueber-/unterschritten werden.

**Auswirkungen auf Pflichten:**

| Pflicht | Kleinst | Klein | Mittelgross | Gross |
|---------|---------|-------|-------------|-------|
| Bilanz | Verkuerzt | Verkuerzt | Erweitert | Vollstaendig |
| GuV | Verkuerzt | Verkuerzt | Erweitert | Vollstaendig |
| Anhang | Entfaellt | Verkuerzt | Erweitert | Vollstaendig |
| Lagebericht | Entfaellt | Entfaellt | Entfaellt* | Pflicht |
| Pruefungspflicht | Nein | Nein | Ja | Ja |
| Offenlegung | Hinterlegung | Verkuerzt | Erweitert | Vollstaendig |

*Mittelgrosse GmbHs: Lagebericht kann entfallen, wenn bestimmte Angaben im Anhang gemacht werden (ss 326 Abs. 1 HGB)

### 6.2 Bilanz - Aufbau und Gliederung (ss 266 HGB)

**AKTIVA:**
```
A. Anlagevermoegen
   I. Immaterielle Vermoegensgegenstaende
      1. Selbst geschaffene gewerbliche Schutzrechte
      2. Entgeltlich erworbene Konzessionen, Lizenzen, Rechte
      3. Geschaefts- oder Firmenwert
      4. Geleistete Anzahlungen
   II. Sachanlagen
      1. Grundstuecke, grundstuecksgleiche Rechte und Bauten
      2. Technische Anlagen und Maschinen
      3. Andere Anlagen, Betriebs- und Geschaeftsausstattung
      4. Geleistete Anzahlungen und Anlagen im Bau
   III. Finanzanlagen
      1. Anteile an verbundenen Unternehmen
      2. Ausleihungen an verbundene Unternehmen
      3. Beteiligungen
      4. Ausleihungen an Unternehmen, mit denen ein Beteiligungsverhaeltnis besteht
      5. Wertpapiere des Anlagevermoegens
      6. Sonstige Ausleihungen

B. Umlaufvermoegen
   I. Vorraete
      1. Roh-, Hilfs- und Betriebsstoffe
      2. Unfertige Erzeugnisse/Leistungen
      3. Fertige Erzeugnisse und Waren
      4. Geleistete Anzahlungen
   II. Forderungen und sonstige Vermoegensgegenstaende
      1. Forderungen aus Lieferungen und Leistungen
      2. Forderungen gegen verbundene Unternehmen
      3. Forderungen gegen Unternehmen mit Beteiligungsverhaeltnis
      4. Sonstige Vermoegensgegenstaende
   III. Wertpapiere
      1. Anteile an verbundenen Unternehmen
      2. Sonstige Wertpapiere
   IV. Kassenbestand, Bundesbankguthaben, Guthaben bei Kreditinstituten

C. Rechnungsabgrenzungsposten

D. Aktive latente Steuern

E. Aktiver Unterschiedsbetrag aus Vermoegesverrechnung
```

**PASSIVA:**
```
A. Eigenkapital
   I.   Gezeichnetes Kapital (Stammkapital)
   II.  Kapitalruecklage
   III. Gewinnruecklagen
        1. Gesetzliche Ruecklage
        2. Ruecklage fuer Anteile an einem herrschenden Unternehmen
        3. Satzungsmaessige Ruecklagen
        4. Andere Gewinnruecklagen
   IV.  Gewinnvortrag / Verlustvortrag
   V.   Jahresueberschuss / Jahresfehlbetrag

B. Rueckstellungen
   1. Rueckstellungen fuer Pensionen und aehnliche Verpflichtungen
   2. Steuerrueckstellungen
   3. Sonstige Rueckstellungen

C. Verbindlichkeiten
   1. Anleihen
   2. Verbindlichkeiten gegenueber Kreditinstituten
   3. Erhaltene Anzahlungen auf Bestellungen
   4. Verbindlichkeiten aus Lieferungen und Leistungen
   5. Verbindlichkeiten aus Wechselzahlungen
   6. Verbindlichkeiten gegenueber verbundenen Unternehmen
   7. Verbindlichkeiten gegenueber Unternehmen mit Beteiligungsverhaeltnis
   8. Sonstige Verbindlichkeiten

D. Rechnungsabgrenzungsposten

E. Passive latente Steuern
```

### 6.3 GuV - Gesamtkostenverfahren vs. Umsatzkostenverfahren (ss 275 HGB)

#### Gesamtkostenverfahren (GKV) - ss 275 Abs. 2 HGB

Das GKV ist in Deutschland am weitesten verbreitet und gliedert die Aufwendungen nach **Aufwandsarten**:

```
 1. Umsatzerloese
 2. Erhoehung/Verminderung des Bestands an fertigen und unfertigen Erzeugnissen
 3. Andere aktivierte Eigenleistungen
 4. Sonstige betriebliche Ertraege
 5. Materialaufwand
    a) Aufwendungen fuer Roh-, Hilfs- und Betriebsstoffe
    b) Aufwendungen fuer bezogene Leistungen
 6. Personalaufwand
    a) Loehne und Gehaelter
    b) Soziale Abgaben und Aufwendungen fuer Altersversorgung
 7. Abschreibungen
    a) Auf immaterielle VG und Sachanlagen
    b) Auf VG des Umlaufvermoegens
 8. Sonstige betriebliche Aufwendungen
 9. Ertraege aus Beteiligungen
10. Ertraege aus anderen Wertpapieren und Ausleihungen des Finanzvermoegens
11. Sonstige Zinsen und aehnliche Ertraege
12. Abschreibungen auf Finanzanlagen und Wertpapiere des UV
13. Zinsen und aehnliche Aufwendungen
14. Steuern vom Einkommen und vom Ertrag
15. Ergebnis nach Steuern
16. Sonstige Steuern
17. Jahresueberschuss / Jahresfehlbetrag
```

#### Umsatzkostenverfahren (UKV) - ss 275 Abs. 3 HGB

Das UKV gliedert die Aufwendungen nach **Funktionsbereichen** (Herstellung, Vertrieb, Verwaltung):

```
 1. Umsatzerloese
 2. Herstellungskosten der zur Erzielung der Umsatzerloese erbrachten Leistungen
 3. Bruttoergebnis vom Umsatz
 4. Vertriebskosten
 5. Allgemeine Verwaltungskosten
 6. Sonstige betriebliche Ertraege
 7. Sonstige betriebliche Aufwendungen
 ... (Finanzergebnis wie GKV)
```

**Empfehlung fuer die Software:** Primaer das **GKV** implementieren, da dieses in Deutschland Standard ist. SKR03 und SKR04 sind auf das GKV ausgelegt.

### 6.4 Anhang und Lagebericht

**Anhang (ss 284-288 HGB):**
- Erlaeuterung der Bilanzierungs- und Bewertungsmethoden
- Angaben zu Haftungsverhaeltnissen
- Angaben zu verbundenen Unternehmen
- Angaben zu Gesamtbezuegen der Organe
- Grundlagen der Fremdwaehrungsumrechnung

**Lagebericht (ss 289 HGB) - nur grosse GmbHs:**
- Darstellung des Geschaeftsverlaufs
- Lage der Gesellschaft
- Risikobericht
- Prognosebericht
- F&E-Bericht

### 6.5 Offenlegungspflichten (Bundesanzeiger)

Alle GmbHs muessen ihren Jahresabschluss im Bundesanzeiger offenlegen (ss 325 HGB). Fristen:
- **Kleine GmbH:** 12 Monate nach Geschaeftsjahresende
- **Mittelgrosse/Grosse GmbH:** 12 Monate nach Geschaeftsjahresende (frueher kuerzere Frist)
- Bei Verstoessen: Ordnungsgeld-Verfahren durch das BfJ (Bundesamt fuer Justiz)

**Implementierungshinweis:** Die Software sollte die Offenlegungsfristen ueberwachen und den Nutzer rechtzeitig erinnern. Idealerweise wird eine XBRL-Schnittstelle zum Bundesanzeiger bereitgestellt.

---

## 7. E-Bilanz

### 7.1 Grundlagen

Seit dem Wirtschaftsjahr 2013 muessen alle bilanzierenden Steuerpflichtigen ihre Bilanz und GuV elektronisch an die Finanzverwaltung uebermitteln (ss 5b EStG).

**Aktueller Stand:**
- **Taxonomie 6.8:** Pflichtanwendung fuer WJ 2025
- **Taxonomie 6.9:** Pflichtanwendung ab WJ 2026 (freiwillige Anwendung fuer WJ 2025 moeglich)
- Verfuegbar unter www.esteuer.de

### 7.2 HGB-Taxonomie - Aufbau

Die Taxonomie gliedert sich in:

| Bereich | Inhalt |
|---------|--------|
| GCD (Global Common Document) | Stammdaten des Unternehmens |
| GAAP | Bilanz, GuV, EK-Spiegel, Kapitalkontenentwicklung |
| Anlagespiegel | Pflicht seit WJ 2017 |
| Kontennachweis | Ab Taxonomie 6.8/6.9: Pflicht fuer alle werthaltig uebermittelten Positionen |

### 7.3 Pflichtfelder und Auffangpositionen

**Mussfelder:** Muessen zwingend uebermittelt werden. Wenn kein Wert vorhanden: Leerfeld oder NIL-Wert.

**Mussfelder mit Auffangpositionen:** Wenn eine detailliertere Aufgliederung nicht moeglich ist, kann der Gesamtbetrag in der Auffangposition ("davon nicht zuordenbar") gemeldet werden.

**Kannfelder:** Freiwillige Uebermittlung.

**Wichtige Aenderung ab Taxonomie 6.9:**
- Fuer **jede werthaltig uebermittelte Position** der Bilanz und GuV muss ein **Kontennachweis** uebermittelt werden
- Der Kontennachweis umfasst alle Sachkonten mit Saldo am Bilanzstichtag
- Dies bedeutet: Die Software muss die vollstaendige Zuordnung aller Konten zu Taxonomiepositionen sicherstellen

### 7.4 Uebermittlung

- Format: **XBRL** (eXtensible Business Reporting Language)
- Schnittstelle: **ERiC** (ELSTER Rich Client) / ELSTER
- Authentifizierung: ELSTER-Zertifikat

**Implementierungshinweis:** Die Software muss:
1. Alle Konten einer Taxonomieposition zuordnen (Mapping-Tabelle)
2. Die XBRL-Datei automatisch aus den Buchungsdaten generieren
3. Den Kontennachweis erstellen (alle Konten mit Saldo)
4. Die Uebermittlung ueber ERiC/ELSTER durchfuehren
5. Validierung der Pflichtfelder vor Uebermittlung

---

## 8. Vermoegensverwaltende GmbH - Besonderheiten

### 8.1 Wertpapierbuchungen

#### Kauf von Wertpapieren

**Aktien als Finanzanlage (Anlagevermoegen) - SKR03:**
```
Soll: 0525 Beteiligungen                           50.000,00 EUR
Soll: 4970 Nebenkosten des Geldverkehrs (Provision)    250,00 EUR
Haben: 1200 Bank                                    50.250,00 EUR
```

**Hinweis:** Anschaffungsnebenkosten (Provisionen, Maklercourtage) werden in die AK einbezogen!

Korrekte Buchung mit Einbeziehung der Nebenkosten:
```
Soll: 0525 Beteiligungen                           50.250,00 EUR
Haben: 1200 Bank                                    50.250,00 EUR
```

**Aktien als Umlaufvermoegen (kurzfristiger Handel) - SKR03:**
```
Soll: 1340 Sonstige Wertpapiere (UV)               50.250,00 EUR
Haben: 1200 Bank                                    50.250,00 EUR
```

#### Verkauf von Wertpapieren

**Mit Gewinn (SKR03):**
```
AK = 50.250 EUR, Veraeusserungserloess = 65.000 EUR, Provision = 300 EUR

Soll: 1200 Bank                                    64.700,00 EUR
Haben: 0525 Beteiligungen                          50.250,00 EUR
Haben: 2740 Ertraege aus Abgang Finanzanlagen       14.450,00 EUR
```

**Mit Verlust (SKR03):**
```
AK = 50.250 EUR, Veraeusserungserloess = 40.000 EUR, Provision = 300 EUR

Soll: 1200 Bank                                    39.700,00 EUR
Soll: 2310 Verluste aus Abgang Finanzanlagen        10.550,00 EUR
Haben: 0525 Beteiligungen                          50.250,00 EUR
```

### 8.2 Realisierte vs. unrealisierte Gewinne/Verluste

**Grundregel:** Nur **realisierte** Gewinne werden ertragsmaessig erfasst. Unrealisierte Gewinne duerfen nach dem **Realisationsprinzip** (ss 252 Abs. 1 Nr. 4 HGB) nicht aktiviert werden.

**Unrealisierte Verluste** muessen dagegen nach dem **Impaaritaetsprinzip** bei dauerhafter Wertminderung als Abschreibung erfasst werden:

**Finanzanlagen (Anlagevermoegen):**
- **Gemildertes Niederstwertprinzip** (ss 253 Abs. 3 S. 5-6 HGB): Abschreibung nur bei **voraussichtlich dauerhafter** Wertminderung (Pflicht)
- Bei voruebergehender Wertminderung: Abschreibung moeglich (Wahlrecht)

**Wertpapiere des Umlaufvermoegens:**
- **Strenges Niederstwertprinzip** (ss 253 Abs. 4 HGB): Abschreibung bei **jedem** niedrigeren Wert am Bilanzstichtag (Pflicht)

**Buchung Teilwertabschreibung (SKR03):**
```
Soll: 2310 Abschreibungen auf Finanzanlagen         5.000,00 EUR
Haben: 0525 Beteiligungen                           5.000,00 EUR
```

**Wertaufholung (Zuschreibung, ss 253 Abs. 5 HGB):**
```
Soll: 0525 Beteiligungen                            3.000,00 EUR
Haben: 2740 Ertraege aus Zuschreibungen Finanzanlagen 3.000,00 EUR
```
Zuschreibung maximal bis zu den urspruenglichen Anschaffungskosten.

### 8.3 Dividendenertraege und ss 8b KStG (95% Freistellung)

Dividenden, die eine GmbH von einer anderen Kapitalgesellschaft erhaelt, sind nach ss 8b Abs. 1 KStG bei der Einkommensermittlung **steuerfrei** zu stellen. Allerdings gelten 5% als nicht abziehbare Betriebsausgaben (ss 8b Abs. 5 KStG).

**Voraussetzungen (seit 2013):**
- Mindestbeteiligungsquote: **10%** zu Beginn des Kalenderjahres (ss 8b Abs. 4 KStG)
- Unter 10%: Dividenden sind **voll steuerpflichtig** ("Streubesitzdividenden")

**Buchung Dividendeneingang (SKR03):**
```
Dividende brutto: 10.000 EUR
KapESt (25%): -2.500 EUR
SolZ (5,5% auf KapESt): -137,50 EUR
Netto-Zufluss: 7.362,50 EUR

Soll: 1200 Bank                                     7.362,50 EUR
Soll: 1548 Anrechenbare KapESt                       2.500,00 EUR
Soll: 1549 Anrechenbarer SolZ auf KapESt               137,50 EUR
Haben: 2610 Ertraege aus Beteiligungen               10.000,00 EUR
```

**Steuerliche Behandlung (ausserbilanziell):**
```
Beteiligungsertraege:              10.000,00 EUR
- 95% steuerfrei (ss 8b Abs. 1):   -9.500,00 EUR
+ 5% nicht abz. BA (ss 8b Abs. 5):      (implizit)
= Steuerpflichtig:                    500,00 EUR (= 5% von 10.000)
```

**Effektive Steuerbelastung auf Dividenden:**
```
500 EUR x 15% KSt = 75 EUR
500 EUR x 15,75% GewSt (bei 450% Hebesatz) = 78,75 EUR
= Gesamt ca. 153,75 EUR = ca. 1,54% auf 10.000 EUR
```

**Implementierungshinweis:** Die Software muss:
1. Beteiligungsquoten je Beteiligung speichern
2. Automatisch zwischen Streubesitz (< 10%) und qualifizierter Beteiligung (>= 10%) unterscheiden
3. Die ss 8b-Korrektur bei der Steuerberechnung automatisch vornehmen
4. Anrechenbare KapESt/SolZ fuer die KSt-Erklaerung erfassen

### 8.4 Teileinkuenfteverfahren

Das Teileinkuenfteverfahren ist fuer die GmbH selbst **nicht** relevant (gilt nur fuer natuerliche Personen als Gesellschafter). Relevant wird es bei der **Gewinnausschuettung an natuerliche Personen** mit Beteiligung im Betriebsvermoegen:
- 60% der Ausschuettung steuerpflichtig
- 40% steuerfrei (ss 3 Nr. 40 EStG)

Die Software sollte dies bei der Erstellung von Steuerbescheinigungen fuer Gesellschafter beruecksichtigen.

### 8.5 Zinsertraege und Quellensteuer

**Buchung Zinsertraege (SKR03):**
```
Soll: 1200 Bank                                     1.000,00 EUR
Haben: 2650 Zinsertraege                             1.000,00 EUR
```

**Bei auslaendischen Zinsertraegen mit Quellensteuer:**
```
Zinsen brutto: 1.000 EUR, ausl. Quellensteuer 15%: 150 EUR

Soll: 1200 Bank                                       850,00 EUR
Soll: 1548 Anrechenbare auslaendische Quellensteuer    150,00 EUR
Haben: 2650 Zinsertraege                             1.000,00 EUR
```

### 8.6 Kryptowaehrungen in der GmbH

**Bilanzielle Einordnung:**
Kryptowaehrungen werden bilanziell als **immaterielle Vermoegensgegenstaende** oder als **sonstige Vermoegensgegenstaende** klassifiziert. Die herrschende Meinung behandelt sie als **immaterielle Wirtschaftsgueter** des Anlage- oder Umlaufvermoegens.

**Kauf von Kryptowaehrungen (SKR03):**
```
Soll: 0027 Sonstige immaterielle VG (AV)          10.000,00 EUR
oder
Soll: 1370 Sonstige Vermoegensgegenstaende (UV)    10.000,00 EUR
Haben: 1200 Bank                                   10.000,00 EUR
```

**Hinweis:** Transaktionsgebuehren (Gas-Fees, Boersengebuehren) gehoeren zu den Anschaffungsnebenkosten.

**Verkauf mit Gewinn (SKR03):**
```
AK: 10.000 EUR, Veraeusserungserloess: 18.000 EUR, Gebuehren: 100 EUR

Soll: 1200 Bank                                    17.900,00 EUR
Haben: 0027 Sonstige immaterielle VG               10.000,00 EUR
Haben: 2700 Sonstige betriebliche Ertraege           7.900,00 EUR
```

**Steuerliche Besonderheiten:**
- **Kein ss 8b KStG anwendbar** (keine Anteile an Kapitalgesellschaften)
- **Keine Haltefrist** (anders als bei Privatpersonen > 1 Jahr steuerfrei)
- Volle Besteuerung mit KSt + GewSt (ca. 30%)
- Bewertung: Anschaffungskostenprinzip, Niederstwertprinzip am Bilanzstichtag
- **FIFO/LIFO-Methode** fuer die Zuordnung bei Teilveraeusserungen zulnaessig (Stetigkeitsgebot beachten)

**Staking-Ertraege:**
```
Soll: 0027 Sonstige immaterielle VG (Zeitwert)      500,00 EUR
Haben: 2700 Sonstige betriebliche Ertraege            500,00 EUR
```

### 8.7 Immobilien in der GmbH

**Kauf einer Immobilie (SKR03):**
```
Kaufpreis: 500.000 EUR (davon Grund: 100.000, Gebaeude: 400.000)
Grunderwerbsteuer (6% in NRW): 30.000 EUR
Notar/Grundbuch: 10.000 EUR

Soll: 0050 Grundstuecke                            100.000,00 EUR
Soll: 0090 Gebaeude                                 440.000,00 EUR
Haben: 1200 Bank (oder 1600 Verbindlichkeiten)      540.000,00 EUR
```

**Hinweis:** Grunderwerbsteuer und Notarkosten sind **Anschaffungsnebenkosten** und werden auf Grund/Gebaeude verteilt.

**Laufende Mietertraege (SKR03):**
```
Soll: 1200 Bank                                     2.000,00 EUR
Haben: 8590 Ertraege aus Vermietung/Verpachtung      2.000,00 EUR
```

**Abschreibung Gebaeude (ss 7 Abs. 4 EStG):**
- Wohngebaeude (Bauantrag nach 31.12.2022): **3% linear** (33,33 Jahre)
- Gewerbliche Gebaeude: **3% linear** (33,33 Jahre)
- Aeltere Gebaeude (vor 1925): 2,5% linear

```
Soll: 4830 Abschreibungen auf Sachanlagen           13.200,00 EUR
Haben: 0090 Gebaeude (440.000 x 3%)                  13.200,00 EUR
```

**Erweiterte Kuerzung Gewerbesteuer (ss 9 Nr. 1 S. 2 GewStG):**
Wenn die GmbH **ausschliesslich** eigenen Grundbesitz verwaltet und nutzt, kann sie die erweiterte Kuerzung beantragen. Dann faellt **keine Gewerbesteuer** auf die Mietertraege an. Steuerbelastung nur: KSt 15% + SolZ = **15,825%**.

**Achtung:** Jede gewerbliche Taetigkeit (auch Nebentaetigkeiten) kann die erweiterte Kuerzung gefaehrden!

---

## 9. Operative GmbH - Besonderheiten

### 9.1 Rechnungsstellung (ss 14, 14a UStG)

**Pflichtangaben einer Rechnung (ss 14 Abs. 4 UStG):**

1. Vollstaendiger Name und Anschrift des leistenden Unternehmers
2. Vollstaendiger Name und Anschrift des Leistungsempfaengers
3. Steuernummer oder USt-IdNr. des leistenden Unternehmers
4. Ausstellungsdatum
5. Fortlaufende Rechnungsnummer (einmalig vergeben)
6. Menge und Art der Lieferung / Umfang der sonstigen Leistung
7. Zeitpunkt der Lieferung/Leistung (oder Vereinnahmung Entgelt)
8. Nach Steuersaetzen aufgeschluesseltes Entgelt
9. Anzuwendender Steuersatz und Steuerbetrag
10. Im Voraus vereinbarte Entgeltminderungen

**Zusaetzlich bei igL:**
- USt-IdNr. des Leistenden UND des Empfaengers
- Hinweis "Steuerfreie innergemeinschaftliche Lieferung"

**Bei Reverse Charge:**
- Hinweis "Steuerschuldnerschaft des Leistungsempfaengers" / "Reverse Charge"

**Ab 01.01.2025: E-Rechnung Pflicht (B2B):**
- Alle inlaendischen B2B-Rechnungen muessen in einem strukturierten elektronischen Format (ZUGFeRD, XRechnung) ausgestellt werden
- Uebergangsfrist bis 31.12.2026 fuer Papierrechnungen / PDF (fuer Rechnungsaussteller)
- Ab 01.01.2027 bzw. 01.01.2028: Weitere Einschraenkungen

**Implementierungshinweis:** Die Software muss E-Rechnungen im ZUGFeRD- oder XRechnungs-Format erstellen und empfangen koennen. Die Validierung der Pflichtangaben muss automatisch erfolgen.

### 9.2 Debitoren- und Kreditorenbuchhaltung

**Debitorenverwaltung:**
- Stammdaten: Name, Adresse, USt-IdNr., Zahlungsbedingungen, Kreditlimit
- OP-Verwaltung (Offene Posten): Zuordnung Zahlungen zu Rechnungen
- Mahnwesen: Automatisierte Mahnstufen
- Forderungsbewertung: Einzelwertberichtigung, Pauschalwertberichtigung

**Kreditorenverwaltung:**
- Stammdaten: Name, Adresse, Steuernummer, Zahlungsbedingungen, IBAN
- OP-Verwaltung
- Zahlungsvorschlaege: Automatische Skonto-Berechnung
- Kontierung: Zuweisung der richtigen Aufwandskonten

### 9.3 Mahnwesen und Forderungsmanagement

**Typische Mahnstufen:**

| Stufe | Zeitpunkt | Aktion |
|-------|-----------|--------|
| Zahlungserinnerung | 7-14 Tage nach Faelligkeit | Freundliche Erinnerung |
| 1. Mahnung | 14-21 Tage nach Faelligkeit | Aufforderung mit Fristsetzung |
| 2. Mahnung | 7-14 Tage nach 1. Mahnung | Letzte Aufforderung, Mahngebuehren |
| 3. Mahnung | 7-14 Tage nach 2. Mahnung | Androhung gerichtliches Mahnverfahren |

**Buchung Mahngebuehren (SKR03):**
```
Soll: 10xxx Debitor                                    5,00 EUR
Haben: 8630 Sonstige betriebliche Ertraege (Mahngebuehren) 5,00 EUR
```

**Einzelwertberichtigung (EWB) - SKR03:**
```
Forderung 10.000 EUR, voraussichtlicher Ausfall 50%:

Soll: 2451 Einstellung EWB auf Forderungen           5.000,00 EUR
Haben: 0993 Einzelwertberichtigungen zu Forderungen   5.000,00 EUR
```

**Pauschalwertberichtigung (PWB) - SKR03:**
```
Gesamtbestand Forderungen 500.000 EUR, PWB-Satz 2%:

Soll: 2452 Einstellung PWB auf Forderungen           10.000,00 EUR
Haben: 0994 Pauschalwertberichtigungen zu Forderungen 10.000,00 EUR
```

**Uneinbringliche Forderung (Forderungsausfall) - SKR03:**
```
Soll: 2405 Forderungsverluste (steuerpflichtiger Umsatz)  10.000,00 EUR
Soll: 1776 Umsatzsteuer 19% (Korrektur)                     1.900,00 EUR
Haben: 10xxx Debitor                                        11.900,00 EUR
```

**Wichtig:** Bei Forderungsausfall ist die bereits abgefuehrte USt zurueckzufordern (ss 17 Abs. 2 UStG).

### 9.4 Abschreibungen (AfA)

#### Lineare AfA (ss 7 Abs. 1 EStG)

Gleichmaessige Verteilung der AK/HK ueber die Nutzungsdauer:

```
AfA-Betrag = AK / Nutzungsdauer (Jahre)

Beispiel: Maschine, AK 60.000 EUR, ND 10 Jahre
AfA/Jahr: 60.000 / 10 = 6.000 EUR

Soll: 4830 Abschreibungen auf Sachanlagen            6.000,00 EUR
Haben: 0400 Maschinen                                6.000,00 EUR
```

#### Degressive AfA (ss 7 Abs. 2 EStG)

Fuer bewegliche Wirtschaftsgueter, die nach dem 30.09.2023 und vor dem 01.01.2025 angeschafft wurden: max. das **Zweifache** der linearen AfA, hoechstens **20%**.

**Fuer ab 2025 angeschaffte WG:** Degressive AfA wurde verlaengert (Wachstumschancengesetz).

```
Beispiel: Maschine, AK 60.000 EUR, ND 10 Jahre, degressiver Satz 20%

Jahr 1: 60.000 x 20% = 12.000 EUR
Jahr 2: 48.000 x 20% =  9.600 EUR
Jahr 3: 38.400 x 20% =  7.680 EUR
...
Wechsel zur linearen AfA, wenn diese hoeher wird.
```

#### Sonder-AfA (ss 7g Abs. 5 EStG)

- 40% der AK/HK im Jahr der Anschaffung (zusaetzlich zur normalen AfA)
- Voraussetzung: Bilanzsumme des Vorjahres <= 235.000 EUR (fuer GmbH: Summe des abnutzbaren Anlagevermoegens)
- Nur fuer **neue oder neuwertige bewegliche Wirtschaftsgueter** des Anlagevermoegens

#### GWG - Geringwertige Wirtschaftsgueter (ss 6 Abs. 2 EStG)

| AK netto | Behandlung |
|----------|------------|
| <= 250 EUR | Sofort als Aufwand buchen (duerfen, muessen nicht aktiviert werden) |
| 250,01 - 800 EUR | Sofortabschreibung im Jahr der Anschaffung (Wahlrecht) ODER Sammelposten |
| 800,01 - 1.000 EUR | Sammelposten (5 Jahre Abschreibung) ODER regulaere AfA |
| > 1.000 EUR | Regulaere AfA ueber Nutzungsdauer |

**Buchung GWG-Sofortabschreibung (SKR03):**
```
Soll: 4855 Sofortabschreibung GWG                     500,00 EUR
Soll: 1576 Vorsteuer 19%                               95,00 EUR
Haben: 1200 Bank                                      595,00 EUR
```

### 9.5 Rueckstellungen (ss 249 HGB)

**Pflichtrueckstellungen:**
1. Ungewisse Verbindlichkeiten
2. Drohende Verluste aus schwebenden Geschaeften
3. Gewaehrleistungen ohne rechtliche Verpflichtung (Kulanz)
4. Unterlassene Aufwendungen fuer Instandhaltung (Nachholung innerhalb 3 Monaten)
5. Abraumbeseitigung (Nachholung im folgenden Geschaeftsjahr)

**Typische Rueckstellungen einer GmbH:**

| Art | SKR03 Konto | Beispiel |
|-----|-------------|---------|
| Steuerrueckstellungen | 0955-0965 | KSt, GewSt |
| Pensionsrueckstellungen | 0970 | Betriebsrenten |
| Urlaubsrueckstellungen | 0980 | Resturlaub MA |
| Prozesskosten-RS | 0976 | Laufende Rechtsstreitigkeiten |
| Jahresabschlusskosten-RS | 0974 | Steuerberater, Wirtschaftspruefer |
| Garantie-/Gewaehrleistungs-RS | 0971 | Produkthaftung |

**Buchung Bildung einer Rueckstellung (SKR03):**
```
Urlaubsrueckstellung:
Soll: 4130 Gesetzliche soziale Aufwendungen          15.000,00 EUR
Haben: 0980 Rueckstellungen fuer Urlaub              15.000,00 EUR

Jahresabschlusskostenrueckstellung:
Soll: 4955 Abschluss- und Pruefungskosten             8.000,00 EUR
Haben: 0974 RS fuer Abschluss und Pruefung             8.000,00 EUR
```

**Buchung Aufloesung/Inanspruchnahme (SKR03):**
```
Inanspruchnahme (Rechnung Steuerberater):
Soll: 0974 RS fuer Abschluss und Pruefung             8.000,00 EUR
Soll: 1576 Vorsteuer 19%                              1.520,00 EUR
Haben: 70xxx Kreditor Steuerberater                    9.520,00 EUR

Aufloesung (nicht benoetigt):
Soll: 0974 RS fuer Abschluss und Pruefung             2.000,00 EUR
Haben: 2735 Ertraege aus Aufloesung von Rueckstellungen 2.000,00 EUR
```

### 9.6 Rechnungsabgrenzungsposten (RAP)

**Aktive RAP (ss 250 Abs. 1 HGB):**
Ausgaben vor dem Bilanzstichtag, die Aufwand fuer die Zeit danach darstellen.

```
Beispiel: Versicherungspraemie 01.10.-30.09. gezahlt am 01.10., 12.000 EUR
Davon betrifft naechstes Jahr (01.01.-30.09.): 9.000 EUR

Zahlung:
Soll: 4360 Versicherungen                           12.000,00 EUR
Haben: 1200 Bank                                    12.000,00 EUR

Abgrenzung zum 31.12.:
Soll: 0980 Aktive RAP                                9.000,00 EUR
Haben: 4360 Versicherungen                           9.000,00 EUR

Aufloesung am 01.01. des Folgejahres:
Soll: 4360 Versicherungen                            9.000,00 EUR
Haben: 0980 Aktive RAP                               9.000,00 EUR
```

**Passive RAP (ss 250 Abs. 2 HGB):**
Einnahmen vor dem Bilanzstichtag, die Ertrag fuer die Zeit danach darstellen.

```
Beispiel: Mieteinnahme Januar des Folgejahres, erhalten am 28.12.

Eingang:
Soll: 1200 Bank                                     2.000,00 EUR
Haben: 0990 Passive RAP                              2.000,00 EUR

Aufloesung am 01.01.:
Soll: 0990 Passive RAP                               2.000,00 EUR
Haben: 8590 Mietertraege                              2.000,00 EUR
```

### 9.7 Lohn- und Gehaltsbuchungen (Ueberblick)

**Grundstruktur der Gehaltsbuchung (SKR03):**

```
Bruttogehalt Angestellter: 5.000 EUR

Soll: 4120 Gehaelter                                5.000,00 EUR
Haben: 1741 Verbindlichkeiten aus LSt                  500,00 EUR (Lohnsteuer)
Haben: 1742 Verbindlichkeiten aus KiSt                  45,00 EUR (Kirchensteuer)
Haben: 1743 Verbindlichkeiten aus SolZ                  27,50 EUR (Solidaritaetszuschlag)
Haben: 1740 Verbindlichkeiten Sozialversicherung       1.000,00 EUR (AN-Anteil SV)
Haben: 1200 Bank (Nettogehalt)                        3.427,50 EUR
```

**AG-Anteil Sozialversicherung (SKR03):**
```
Soll: 4130 Gesetzl. soziale Aufwendungen (AG-Anteil)  1.050,00 EUR
Haben: 1740 Verbindlichkeiten Sozialversicherung       1.050,00 EUR
```

**Abfuehrung an Finanzamt/Krankenkasse:**
```
Soll: 1741 Verbindl. LSt                              500,00 EUR
Soll: 1742 Verbindl. KiSt                              45,00 EUR
Soll: 1743 Verbindl. SolZ                              27,50 EUR
Haben: 1200 Bank                                       572,50 EUR

Soll: 1740 Verbindl. SV (AN + AG)                    2.050,00 EUR
Haben: 1200 Bank                                     2.050,00 EUR
```

**Implementierungshinweis:** Die Lohnbuchhaltung ist extrem komplex (SV-Beitraege, Steuerklassen, Freibetraege, Grenzwerte). Es wird empfohlen, die eigentliche Lohnberechnung an ein spezialisiertes System zu delegieren (z.B. DATEV Lohn und Gehalt, Personio, Sage) und nur die **Sammelbuchungen** in die FiBu zu uebernehmen.

---

## 10. Besondere Buchungsvorfaelle

### 10.1 Gesellschafterdarlehen

**Darlehen des Gesellschafters an die GmbH (SKR03):**
```
Auszahlung:
Soll: 1200 Bank                                   100.000,00 EUR
Haben: 0740 Verbindlichkeiten gg. Gesellschaftern  100.000,00 EUR

Zinszahlung (Marktueblicher Zins!):
Soll: 2110 Zinsaufwendungen gg. Gesellschaftern      5.000,00 EUR
Haben: 1200 Bank                                      5.000,00 EUR
```

**Darlehen der GmbH an den Gesellschafter (SKR03):**
```
Auszahlung:
Soll: 1360 Forderungen gg. Gesellschaftern          50.000,00 EUR
Haben: 1200 Bank                                    50.000,00 EUR

Zinsforderung:
Soll: 1360 Forderungen gg. Gesellschaftern           2.500,00 EUR
Haben: 2650 Zinsertraege                              2.500,00 EUR
```

**Achtung vGA-Risiko:** Wenn Zinsen nicht marktueblich sind, liegt eine verdeckte Gewinnausschuettung vor!
- Zu hohe Zinsen an Gesellschafter-Darlehensgeber = vGA
- Zu niedrige Zinsen bei Darlehen an Gesellschafter = vGA
- Zinslos = idR vGA (mindestens 3-6% marktueblich, abhaengig vom Risiko)

### 10.2 Eigenkapitalveraenderungen

**Kapitalerhoehung (SKR03):**
```
Bareinlage 50.000 EUR auf Stammkapital:
Soll: 1200 Bank                                    50.000,00 EUR
Haben: 0800 Gezeichnetes Kapital                    50.000,00 EUR

Mit Agio (Aufgeld):
Soll: 1200 Bank                                    75.000,00 EUR
Haben: 0800 Gezeichnetes Kapital                    50.000,00 EUR
Haben: 0835 Kapitalruecklage (ss 272 Abs. 2 Nr. 1)   25.000,00 EUR
```

**Kapitalherabsetzung (SKR03):**
```
Ordentliche Kapitalherabsetzung mit Auszahlung:
Soll: 0800 Gezeichnetes Kapital                    25.000,00 EUR
Haben: 1200 Bank                                    25.000,00 EUR
```

### 10.3 Gewinnverwendungsbeschluss

Nach Feststellung des Jahresabschlusses beschliessen die Gesellschafter ueber die Verwendung des Jahresueberschusses.

**Buchung Gewinnausschuettung (SKR03):**
```
Beschluss:
Soll: 0860 Gewinnvortrag                            50.000,00 EUR
Haben: 0755 Verbindlichkeiten gg. Gesellschaftern
       aus Gewinnausschuettung                       50.000,00 EUR

Auszahlung (nach KapESt-Einbehalt):
Soll: 0755 Verbindl. gg. Gesellschaftern            50.000,00 EUR
Haben: 1200 Bank                                    36.862,50 EUR (Nettobetrag)
Haben: 1743 Verbindl. KapESt (25%)                  12.500,00 EUR
Haben: 1744 Verbindl. SolZ auf KapESt (5,5%)           637,50 EUR
```

**Abfuehrung KapESt:**
```
Soll: 1743 Verbindl. KapESt                         12.500,00 EUR
Soll: 1744 Verbindl. SolZ                              637,50 EUR
Haben: 1200 Bank                                    13.137,50 EUR
```

**Frist:** KapESt-Anmeldung bis zum 10. des Folgemonats nach Ausschuettung.

**Thesaurierung (Einstellung in Gewinnruecklagen):**
```
Soll: 0860 Gewinnvortrag                            50.000,00 EUR
Haben: 0855 Andere Gewinnruecklagen                  50.000,00 EUR
```

### 10.4 Tantiemen und Geschaeftsfuehrergehalt

**Geschaeftsfuehrergehalt (SKR03):**
```
GF-Gehalt wird wie normales Gehalt gebucht:
Soll: 4124 Geschaeftsfuehrergehaelter GmbH           15.000,00 EUR
Haben: 1741 Verbindl. LSt                             3.000,00 EUR
Haben: 1740 Verbindl. SV                              1.500,00 EUR
Haben: 1200 Bank (Netto)                              10.500,00 EUR
```

**Hinweis fuer Gesellschafter-GF:** Kein SV-Beitrag bei beherrschendem GF (>= 50% Beteiligung). Hier wird idR nur LSt einbehalten.

**Tantiemen (variable Verguetung):**
```
Bildung Tantiemenrueckstellung (Jahresende):
Soll: 4124 Geschaeftsfuehrergehaelter               30.000,00 EUR
Haben: 0977 Rueckstellungen fuer Tantiemen            30.000,00 EUR

Auszahlung im Folgejahr:
Soll: 0977 Rueckstellungen fuer Tantiemen             30.000,00 EUR
Haben: 1741 Verbindl. LSt                             6.000,00 EUR
Haben: 1200 Bank                                     24.000,00 EUR
```

**vGA-Risiko:** Gesamtverguetung (Gehalt + Tantieme + Sachbezuege) muss angemessen sein. Richtwerte:
- Feste Verguetung: 75% des Gesamtpakets
- Tantieme: maximal 25% des Gesamtpakets
- Gesamtverguetung: Abhaengig von Branche, Unternehmensgroesse, Umsatz

### 10.5 Privatnutzung Firmenwagen

#### 1%-Regelung (ss 6 Abs. 1 Nr. 4 S. 2 EStG)

```
Monatlicher Privatanteil = 1% des Bruttolistenpreises (BLP)

Beispiel: BLP 50.000 EUR, Entfernung Wohnung-Arbeit 20 km
Privatnutzung/Monat: 500 EUR (1% x 50.000)
Fahrten Wohnung-Arbeit/Monat: 150 EUR (0,03% x 50.000 x 20 km)
Gesamt: 650 EUR/Monat
```

**Buchung Privatnutzung Kfz - Gesellschafter-GF (SKR03):**
```
Wenn ueber Lohnabrechnung (Sachbezug):
Soll: 4124 GF-Gehaelter (Sachbezug)                   650,00 EUR
Haben: 8921 Verwendung von Gegenstaenden
       fuer Zwecke ausserhalb des Unternehmens 19%     546,22 EUR
Haben: 1776 Umsatzsteuer 19%                          103,78 EUR
```

**Oder als Eigenverbrauch/Entnahme (bei Nicht-Arbeitnehmer):**
```
Soll: 1800 Privatentnahmen / Verrechnungskonto GF     773,50 EUR
Haben: 8921 Verwendung von Gegenstaenden
       fuer Zwecke ausserhalb des Unternehmens 19%     650,00 EUR
Haben: 1776 Umsatzsteuer 19%                          123,50 EUR
```

#### Fahrtenbuchmethode

Alternative zur 1%-Regelung. Erfordert:
- Lueckenloses Fahrtenbuch (Datum, km-Stand, Ziel, Zweck, km)
- Trennung private/betriebliche Fahrten
- Berechnung: Tatsaechliche Kfz-Kosten x Privatanteil (%)

**Implementierungshinweis:** Die Software sollte beide Methoden unterstuetzen und einen Vorteilhaftigkeitsvergleich ermoeglichen.

### 10.6 Bewirtungskosten (ss 4 Abs. 5 Nr. 2 EStG)

**Steuerliche Abzugsfaehigkeit:**
- **70%** der angemessenen Bewirtungskosten sind als BA abzugsfaehig
- **30%** sind **nicht** abzugsfaehig
- **100%** der Vorsteuer sind abziehbar (sofern ordnungsgemaesser Beleg)

**Anforderungen an den Bewirtungsbeleg:**
- Ort der Bewirtung
- Tag der Bewirtung
- Teilnehmer (namentlich)
- Anlass der Bewirtung
- Hoehe der Aufwendungen
- Trinkgeld (gesondert)
- Maschinell erstellte und registrierte Rechnung (seit 01.07.2021)

**Buchung Bewirtungskosten (SKR03):**
```
Restaurantrechnung 119,00 EUR brutto (inkl. 19% USt):

Soll: 4650 Bewirtungskosten (70% abzugsfaehig)        70,00 EUR
Soll: 4654 Nicht abzugsfaehige Bewirtungskosten (30%)  30,00 EUR
Soll: 1576 Vorsteuer 19% (100% abziehbar!)             19,00 EUR
Haben: 1200 Bank                                      119,00 EUR
```

### 10.7 Reisekosten

**Uebernachtungskosten:**
```
Hotelrechnung 130,90 EUR (110,00 netto + 7% USt Uebernachtung, Fruehstueck separat):

Soll: 4674 Reisekosten Uebernachtung                 110,00 EUR
Soll: 1571 Vorsteuer 7%                                 7,70 EUR
Soll: 4664 Reisekosten Verpflegung (Fruehstueck)       12,00 EUR
Soll: 1576 Vorsteuer 19% (Fruehstueck)                  2,28 EUR
Haben: 1200 Bank                                      131,98 EUR
```

**Verpflegungsmehraufwand (ss 9 Abs. 4a EStG):**
- Abwesenheit > 8 Stunden: **14 EUR** Pauschale
- Abwesenheit 24 Stunden: **28 EUR** Pauschale
- An-/Abreisetag: **14 EUR** Pauschale

```
Dienstreise 3 Tage (Anreise + 1 voller Tag + Abreise):
2 x 14 EUR + 1 x 28 EUR = 56 EUR

Soll: 4668 Verpflegungsmehraufwand                    56,00 EUR
Haben: 1200 Bank                                      56,00 EUR
```

**Fahrtkosten:**
- Tatsaechliche Kosten (bei Firmenwagen)
- Oder km-Pauschale: **0,30 EUR/km** (Pkw)
- Ab dem 21. km: **0,38 EUR/km** (fuer Fahrten Wohnung-Arbeit, nicht fuer Dienstreisen)

---

## 11. Holding-Strukturen

### 11.1 Gewinnabfuehrungsvertrag (GAV)

Der GAV ist die Grundlage der ertragsteuerlichen Organschaft (ss 14 KStG):

**Voraussetzungen:**
- Notarielle Beurkundung
- Handelsregistereintragung
- Mindestlaufzeit: 5 Jahre
- Finanzielle Eingliederung (Stimmrechtsmehrheit)
- Tatsaechliche Durchfuehrung (Abfuehrung/Verlustuebernahme)

**Buchungen bei Organschaft:**

**Organgesellschaft (Tochter) - Gewinnfall (SKR03):**
```
Soll: 2600 Aufwendungen aus Gewinnabfuehrungsvertrag   200.000,00 EUR
Haben: 0740 Verbindlichkeiten gg. Organtraeger          200.000,00 EUR
```

**Organgesellschaft (Tochter) - Verlustfall (SKR03):**
```
Soll: 1300 Forderungen gg. Organtraeger                  80.000,00 EUR
Haben: 2710 Ertraege aus Verlustuebernahme                80.000,00 EUR
```

**Organtraeger (Mutter) - Gewinnfall (SKR03):**
```
Soll: 1300 Forderungen gg. Organgesellschaft            200.000,00 EUR
Haben: 2710 Ertraege aus Gewinnabfuehrung                200.000,00 EUR
```

**Organtraeger (Mutter) - Verlustfall (SKR03):**
```
Soll: 2600 Aufwendungen aus Verlustuebernahme            80.000,00 EUR
Haben: 0740 Verbindlichkeiten gg. Organgesellschaft       80.000,00 EUR
```

**Steuerliche Wirkung:**
- Organgesellschaft: Einkommen = 0 (da abgefuehrt/uebernommen)
- Organtraeger: Versteuert eigenes Einkommen + zugerechnetes Organ-Einkommen
- KSt: Volle Verrechnung (Gewinne und Verluste saldierbar)
- GewSt: Ebenfalls Organschaft moeglich (gleiche Voraussetzungen)

### 11.2 Schachtelprivileg (ss 8b KStG)

Das "Schachtelprivileg" ergibt sich aus ss 8b KStG:

**Fuer Dividenden (ss 8b Abs. 1 KStG):**
- Mindestbeteiligung >= 10% (ss 8b Abs. 4 KStG)
- 95% der Dividenden steuerfrei (5% als nicht abziehbare BA)
- Effektive Steuerbelastung: ca. 1,5%

**Fuer Veraeusserungsgewinne (ss 8b Abs. 2 KStG):**
- **Keine** Mindestbeteiligungsquote erforderlich!
- 95% des Veraeusserungsgewinns steuerfrei
- 5% als nicht abziehbare Betriebsausgaben
- **Achtung:** Veraeusserungsverluste sind zu 100% nicht abziehbar (ss 8b Abs. 3 KStG)!

**Gewerbesteuerliches Schachtelprivileg (ss 9 Nr. 2a GewStG):**
- Kuerzung fuer Gewinnanteile aus Beteiligungen >= 15% (hoeherer Schwellenwert als KSt!)
- Bei < 15%: Volle GewSt auf die Beteiligungsertraege

**Zusammenfassung Beteiligungsschwellen:**

| Steuerart | Mindestbeteiligung | Freistellung |
|-----------|--------------------|--------------|
| KSt (Dividenden) | >= 10% | 95% |
| KSt (Veraeusserungsgewinne) | 0% (keine) | 95% |
| GewSt (Kuerzung) | >= 15% | 95% |
| KSt (Streubesitz < 10%) | - | 0% (voll steuerpflichtig) |

### 11.3 Konsolidierung (Ueberblick)

Bei Konzernen mit Beherrschungsverhaeltnissen ist ggf. ein Konzernabschluss zu erstellen (ss 290 HGB).

**Befreiung fuer kleine Konzerne (ss 293 HGB):**
- Bilanzsumme <= 24.000.000 EUR (brutto) / 20.000.000 EUR (netto)
- Umsatzerloese <= 48.000.000 EUR (brutto) / 40.000.000 EUR (netto)
- Arbeitnehmer <= 250

**Konsolidierungsschritte:**
1. **Kapitalkonsolidierung** (ss 301 HGB): Beteiligungsbuchwert gegen Eigenkapital der Tochter aufrechnen
2. **Schuldenkonsolidierung** (ss 303 HGB): Konzerninterne Forderungen/Verbindlichkeiten eliminieren
3. **Aufwands-/Ertragskonsolidierung** (ss 305 HGB): Konzerninterne Ertraege/Aufwendungen eliminieren
4. **Zwischenergebniseliminierung** (ss 304 HGB): Unrealisierte Gewinne aus konzerninternen Geschaeften eliminieren

**Implementierungshinweis:** Eine vollstaendige Konsolidierungsloesung ist komplex. Fuer die erste Version der Software empfiehlt es sich, die Konsolidierung als spaeteren Entwicklungsschritt zu planen und zunaechst die Einzelabschluss-Funktionalitaet zu perfektionieren.

---

## 12. GoBD-Compliance und Betriebspruefung

*(Ergaenzt auf Basis der rechtlichen Analyse des Rechtsanwalts, vgl. /Users/hendrikhemken/Desktop/Tax/research/recht/rechtliche-anforderungen.md)*

### 12.1 Aufbewahrungsfristen (differenziert nach BEG IV, seit 01.01.2025)

**Wichtig:** Die Fristen sind differenzierter als oft vereinfacht dargestellt:

| Dokumentenart | Aufbewahrungsfrist | Rechtsgrundlage |
|---------------|-------------------|-----------------|
| Buecher, Inventare, Jahresabschluesse, Eroeffnungsbilanzen | **10 Jahre** | ss 147 Abs. 1 Nr. 1, Abs. 3 AO |
| Buchungsbelege (Rechnungen, Kontoauszuege, etc.) | **8 Jahre** (NEU, vorher 10!) | ss 257 Abs. 4 HGB i.V.m. BEG IV |
| Handelsbriefe, sonstige Unterlagen | **6 Jahre** | ss 257 Abs. 4 HGB |

**Implementierungshinweis:** Die Software muss unterschiedliche Aufbewahrungsfristen je Dokumentenart verwalten. Loeschung vor Ablauf ist unzulaessig. Nach Ablauf: DSGVO-konforme Loeschung/Anonymisierung personenbezogener Daten anbieten. Grundregel: Soft-Delete implementieren, keine physische Loeschung.

### 12.2 GoBD - Grundsaetze zur ordnungsmaessigen Fuehrung und Aufbewahrung

Die GoBD wurden am **14.07.2025** primaer wegen der E-Rechnungspflicht aktualisiert.

**Zeitgerechte Erfassung:**
- Bargeschaefte: **taeglich** erfassen
- Unbare Geschaeftsvorfaelle: innerhalb von **10 Tagen**
- Periodenfremd erfasste Vorgaenge: Nachvollziehbarkeit gewaehrleisten

**Unveraenderbarkeit:**
- Buchungen duerfen nach Festschreibung **nicht** geloescht oder ueberschrieben werden
- Aenderungen nur durch Stornobuchungen/Gegenbuchungen
- Jede Aenderung muss protokolliert werden (Aenderungshistorie/Audit Trail)

**Nachvollziehbarkeit:**
- Belegprinzip: Keine Buchung ohne Beleg
- Progressive und retrograde Pruefbarkeit (vom Beleg zur Buchung und umgekehrt)
- Kontierungsvermerk auf dem Beleg (oder elektronischer Verweis)

### 12.3 Digitale Betriebspruefung (GDPdU / ss 147 Abs. 6 AO)

Die Software **muss** alle drei Datenzugriffsarten unterstuetzen:

| Zugriffsart | Bezeichnung | Anforderung |
|-------------|-------------|-------------|
| **Z1** | Unmittelbarer Zugriff | Pruefer arbeitet direkt im System (Nur-Lese-Zugang) |
| **Z2** | Mittelbarer Zugriff | Pruefer gibt Auswertungswuensche vor, die vom Steuerpflichtigen im System ausgefuehrt werden |
| **Z3** | Datentraegerueberlassung | Export aller steuerlich relevanten Daten im maschinell auswertbaren Format |

**Z3-Export-Format:**
- Kompatibel mit IDEA/AIS TaxAudit (CSV/XML mit Indexdatei/Beschreibungsdatei)
- Alle Buchungssaetze mit Pflichtfeldern: Datum, Belegnummer, Buchungstext, Konto, Gegenkonto, Betrag, Steuer
- Stammdaten: Kontenplan, Debitoren, Kreditoren
- Journaldaten: Chronologische Buchungsreihenfolge

**Bei Systemwechsel:** Alt-Daten-Zugriff muss 5 Jahre nach Archivierung erhalten bleiben.

### 12.4 E-Rechnungspflicht - Detaillierte Uebergangsfristen

| Zeitraum | Pflicht |
|----------|---------|
| Ab 01.01.2025 | **Empfangspflicht** fuer alle Unternehmen (B2B inlaendisch) |
| Ab 01.01.2027 | **Ausstellungspflicht** fuer Unternehmen mit > 800.000 EUR Vorjahresumsatz |
| Ab 01.01.2028 | **Ausstellungspflicht** fuer **alle** Unternehmen |
| Bis 31.12.2026 | Papierrechnungen und einfache PDF noch zulaessig (fuer Ausstellung) |

**Formate:**
- **XRechnung:** Reines XML-Format (strukturiert, kein visuelles PDF)
- **ZUGFeRD 2.x:** Hybrid-Format (PDF + eingebettetes XML nach EN 16931)
- **Einfache PDF gilt NICHT als E-Rechnung!**

---

## Zusammenfassung: Anforderungen an die Software-Logik

### Kernfunktionen der Buchungsengine

1. **Doppelte Buchfuehrung:** Strikte Soll-Haben-Gleichheit, Splitbuchungen, automatische Gegenkontierung
2. **Kontenrahmen:** SKR03 + SKR04 vollstaendig hinterlegt, mit Mapping-Tabellen und Taxonomie-Zuordnung
3. **Steuerautomatik:** Automatische USt-Berechnung auf Automatikkonten, Steuerschluessel-System
4. **Nebenbuecher:** Debitoren, Kreditoren, Anlagen als separate Module mit Hauptbuch-Integration
5. **Periodenabgrenzung:** Automatische RAP-Buchungen, Rueckstellungen, Abschreibungen

### Steuerberechnung und Meldewesen

6. **UStVA:** Automatische Berechnung und ELSTER-Uebermittlung (monatlich/vierteljaehrlich)
7. **USt-Jahreserklaerung:** Generierung aus Buchungsdaten, ELSTER-Uebermittlung
8. **Zusammenfassende Meldung:** Automatische Generierung, USt-IdNr.-Validierung
9. **KSt-Berechnung:** Einkommensermittlung mit ausserbilanziellen Korrekturen (vGA, ss 8b, nicht abz. BA)
10. **GewSt-Berechnung:** Hinzurechnungen, Kuerzungen, Hebesatz-Verwaltung, Rueckstellungsberechnung
11. **E-Bilanz:** Taxonomie-Mapping, XBRL-Generierung, Kontennachweis, ERiC-Uebermittlung

### Spezialmodule

12. **Vermoegensverwaltende GmbH:** Wertpapier-/Krypto-/Immobilienverwaltung, ss 8b-Automatik, Beteiligungsquoten
13. **Operative GmbH:** E-Rechnung (ZUGFeRD/XRechnung), Mahnwesen, AfA-Verwaltung
14. **Holding:** Organschaftsbuchungen, Schachtelprivileg-Pruefung, Verlustabzugsueberwachung

### GoBD-Compliance und Betriebspruefung

15. **Unveraenderbarkeit:** Keine Loeschung von Buchungen, nur Stornos. Vollstaendiger Audit Trail.
16. **Aufbewahrungsfristen:** Differenzierte Fristen (10/8/6 Jahre je Dokumentenart), automatische Ueberwachung
17. **Zeitgerechte Erfassung:** Warnung bei verspaeteter Buchungserfassung (Bar: taeglich, Unbar: 10 Tage)
18. **Digitale Betriebspruefung:** Z1/Z2/Z3-Zugriff, IDEA/AIS-kompatibler Datenexport
19. **GoBD-Verfahrensdokumentation:** Automatische Protokollierung aller Systemaenderungen

### Schnittstellen (API)

20. **ELSTER/ERiC:** UStVA, USt-Jahreserklaerung, KSt, GewSt, E-Bilanz, LStA
21. **Bundesanzeiger:** Offenlegung Jahresabschluss
22. **VIES:** USt-IdNr.-Validierung
23. **BZSt:** OSS-Meldung, ZM
24. **Banken:** Kontoumsaetze (FinTS/HBCI), SEPA-Zahlungen
25. **E-Rechnung:** ZUGFeRD 2.x, XRechnung (Empfang ab 2025, Versand ab 2027/2028)

---

## Quellen und Rechtsgrundlagen

### Gesetze
- Handelsgesetzbuch (HGB), insbesondere ss 238-342a
- Einkommensteuergesetz (EStG), insbesondere ss 4-7g
- Koerperschaftsteuergesetz (KStG), insbesondere ss 1-8d, ss 14-19, ss 23, ss 27
- Gewerbesteuergesetz (GewStG), insbesondere ss 2, ss 7-9, ss 11, ss 28
- Umsatzsteuergesetz (UStG), insbesondere ss 1-4, ss 12-15, ss 18
- Abgabenordnung (AO), insbesondere ss 140-148

### Verordnungen und Richtlinien
- Umsatzsteuer-Durchfuehrungsverordnung (UStDV)
- Einkommensteuer-Durchfuehrungsverordnung (EStDV)
- AfA-Tabellen des BMF
- E-Bilanz-Taxonomie (www.esteuer.de)

### GoBD und Betriebspruefung
- GoBD (Grundsaetze zur ordnungsmaessigen Fuehrung und Aufbewahrung von Buechern, Aufzeichnungen und Unterlagen in elektronischer Form sowie zum Datenzugriff) - aktualisiert 14.07.2025
- Buerokratieentlastungsgesetz IV (BEG IV) - seit 01.01.2025
- GDPdU / ss 147 Abs. 6 AO (Datenzugriff bei Betriebspruefung)

### Aktuelle Aenderungen 2025/2026
- Taxonomie 6.9: Pflichtanwendung ab WJ 2026 (mit erweiterten Kontennachweisen)
- E-Rechnung: Empfangspflicht B2B ab 01.01.2025, Ausstellungspflicht ab 01.01.2027/2028
- Gastronomie-USt: Dauerhaft 7% fuer Speisen ab 01.01.2026
- Degressive AfA: Verlaengerung durch Wachstumschancengesetz
- Geplante KSt-Senkung ab 2028 (schrittweise auf 10% bis 2032)
- BEG IV: Aufbewahrungsfristen fuer Buchungsbelege von 10 auf 8 Jahre reduziert
- GoBD: Aktualisierung primaer wegen E-Rechnungspflicht (14.07.2025)
- Buchfuehrungspflicht-Schwellenwerte (seit 01.01.2024): Umsatz > 800.000 EUR, Gewinn > 80.000 EUR

### Verweis auf ergaenzende Reports
- Rechtliche Anforderungen: `/Users/hendrikhemken/Desktop/Tax/research/recht/rechtliche-anforderungen.md`
