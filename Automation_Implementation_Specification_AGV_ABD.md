
# Automation Implementation Specification
## Archivierung von Austrittsnachweisen (AGV) und Ausfuhrbegleitdokumenten (ABD)

**Organisation:** NovaTaste Austria GmbH — Trade Compliance
**Stand:** Juli 2026 · Version 0.9 (Review-Fassung)
**Status:** Entwurf zur Freigabe durch den Fachbereich. Noch nicht implementierungsfrei.

---

## 0. Über dieses Dokument

### 0.1 Zweck

Dieses Dokument beschreibt den fachlichen Soll-Prozess und die technische Spezifikation für die Automatisierung der Archivierung von Austrittsnachweisen. Es ist so angelegt, dass

- der **Fachbereich** die fachlichen Regeln prüfen, korrigieren und freigeben kann, ohne technische Vorkenntnisse zu benötigen,
- ein **Entwickler** die Umsetzung beginnen kann, ohne die Originaldokumente erneut lesen zu müssen.

### 0.2 Quellen

| Kürzel | Dokument | Rolle |
|---|---|---|
| **[KD]** | `Klick-Doku_AGV_ABD_03-07-2026` (117 Seiten, 180 Schritte, mit Screenshots) | **Offizielle Basis** des Ist-Prozesses. Aufzeichnung einer Arbeitssitzung vom 03.07.2026 |
| **[SOP]** | `Archivierung von Austrittsnachweisen.docx` | Fachliche Regeln, Randfälle, Fristen, Rechtsgrundlagen |
| **[PPT]** | `Automatisierung.pptx` | Zielbild, Definition der „Kern-Daten" mit Formularfeld-Nummern, Pain Points |
| **[SCR]** | Screenshots aus [KD], im Rahmen der Analyse ausgewertet | Belegt Feldnamen, Spaltennamen, Query-Bezeichnung |
| **[FB]** | Entscheidungen und Feedback des Fachbereichs im Analyseverlauf | siehe Abschnitt 0.4 |

### 0.3 Lesehinweise und Kennzeichnungen

| Kennzeichnung | Bedeutung |
|---|---|
| **BELEGT** | Aus [KD], [SOP], [PPT] oder [SCR] nachweisbar. Fundstelle ist angegeben |
| **HYPOTHESE** | Fachlich plausible Annahme, **nicht** belegt. Muss bestätigt werden |
| **OP-nn** | Offener Punkt. Siehe Abschnitt 11. Nicht implementieren, bevor geklärt |
| **Aufwand: gering / mittel / hoch** | Mehraufwand gegenüber dem in [KD] beschriebenen Ist-Prozess |

**Es wurden bewusst keine stillschweigenden Annahmen getroffen.** Wo Informationen fehlen, steht ein offener Punkt. Insgesamt sind **37 offene Punkte** dokumentiert, davon **14 blockierend** für den Start der Implementierung.

### 0.4 Bereits getroffene Entscheidungen des Fachbereichs

Diese Punkte sind im Analyseverlauf entschieden und gelten als gesetzt:

| # | Entscheidung | Konsequenz |
|---|---|---|
| E-1 | Die Excel-Masterliste darf als **führendes System** entfallen | Prozesszustand wandert in einen eigenen Speicher (Abschnitt 5) |
| E-2 | Ein **Monitoring ist zwingend** erforderlich | Teilprozess P7, Sichten in Abschnitt 8.4 |
| E-3 | Der Fachbereich behält **Schreibzugriff** für Rand- und Sonderfälle | Overlay-Muster, Tabelle `T_MANUELL` |
| E-4 | AGV und ABD werden künftig **als XML** bereitgestellt werden können | Verarbeitung primär XML, Ablage weiterhin PDF (Abschnitt 6.1) |
| E-5 | **SAP bleibt gesetzt** — als Datenquelle und als Ablageort der Nachweis-Anlage | Bausteine A und B, Abschnitt 8.2 |
| E-6 | Lesen aus SAP möglichst über **OData**, nicht über UI-Automation | Abschnitt 8.2 |

---

## 1. Zusammenfassung für den Fachbereich

### 1.1 Worum es geht

Bei jeder Ausfuhr in ein Drittland muss NovaTaste nachweisen, dass die Ware das Zollgebiet der EU tatsächlich verlassen hat. Der Nachweis heißt **Ausgangsvermerk (AGV)** und ist Voraussetzung für die Umsatzsteuerfreiheit der Lieferung. Er trifft nach der Ausfuhr per E-Mail vom Zolldienstleister ein und muss der richtigen Rechnung zugeordnet, geprüft und revisionssicher abgelegt werden. Fehlt er, muss er nachträglich beschafft werden.

Heute passiert das vollständig manuell: pro Monat 80–100 Vorgänge, je Vorgang 15–25 Klicks über Outlook, SAP, ein Netzlaufwerk und eine Excel-Liste.

### 1.2 Was die Automatisierung übernehmen soll

1. **Erkennen**, welche Rechnungen überhaupt einen Nachweis brauchen — täglich aus SAP
2. **Einlesen und zuordnen** der eintreffenden Zolldokumente zur passenden Rechnung
3. **Prüfen**, ob die Angaben auf dem Zolldokument mit den Rechnungsdaten übereinstimmen
4. **Ablegen** — am Laufwerk unter der MRN und als Anlage am SAP-Fakturabeleg
5. **Überwachen**, für welche Ausfuhr noch kein Nachweis vorliegt, und die Nachverfolgung anstoßen
6. **Melden**, wenn etwas nicht zusammenpasst — als Klärfall mit Angabe des abweichenden Feldes

### 1.3 Was sich für den Fachbereich ändert

| Heute | Künftig |
|---|---|
| Excel-Liste pflegen, Daten aus SAP hineinkopieren | entfällt — die Liste entsteht automatisch |
| Zolldokument und Rechnung visuell vergleichen | maschinell; der Mensch sieht nur Abweichungen |
| PDF am Laufwerk speichern, in SAP anhängen, in Excel verlinken | entfällt |
| Offene Nachweise manuell filtern, zweimal monatlich | Monitoring-Sicht jederzeit aktuell, Erinnerung automatisch |
| Alles händisch korrigierbar | weiterhin händisch korrigierbar — über eine eigene Nachbearbeitungstabelle |

**Was nicht wegfällt:** die fachliche Entscheidung in Zweifelsfällen, der Kontakt zum Broker bei Abweichungen, und die Beurteilung von Alternativnachweisen. Die Automatisierung liefert Vorarbeit und Transparenz, sie ersetzt keine Fachentscheidung.

### 1.4 Was der Fachbereich zur Freigabe entscheiden muss

Die wichtigsten fünf Punkte, ohne die nicht begonnen werden kann:

- **OP-01** Welche Felder müssen exakt übereinstimmen, damit ein Nachweis als geprüft gilt? (Drei Quellen nennen drei unterschiedliche Listen)
- **OP-04/05** Welche Toleranzen gelten bei Betrag und Gewicht?
- **OP-11** Ab welcher Frist gilt ein Nachweis als überfällig? (Drei Fristen im Umlauf: 6 Monate, 3 Monate, 12 Wochen)
- **OP-33** Welches Datum ist Bezugspunkt für die Fristenrechnung — Fakturadatum, Lieferdatum oder Gestellungsdatum?
- **OP-25** Sind DHL-Mustersendungen, Intercompany-Direktanlieferungen und Alternativnachweise in Phase 1 dabei?

---

## 2. Fachliche Grundlagen

### 2.1 Begriffe

| Begriff | Bedeutung | Quelle |
|---|---|---|
| **ABD** — Ausfuhrbegleitdokument (*Export Accompanying Document*) | Offizielles Zollpapier, das die Ware bis zur EU-Außengrenze begleitet und die zulässige Ausfuhr belegt. Pflicht bei Drittlandausfuhren über 1.000 € oder 1.000 kg | [SOP] 1.1 |
| **AGV** — Ausgangsvermerk / Austrittsnachweis (*Export Notification*) | Beweis, dass die Ware die EU tatsächlich verlassen hat. Voraussetzung der Umsatzsteuerfreiheit | [SOP] 1.1 |
| **MRN** — Movement Reference Number | Eindeutige Referenz der Ausfuhranmeldung, **18 Zeichen**. Steht auf ABD **und** AGV und verknüpft beide | [SOP] 1.1 |
| **LRN** | Weitere Referenz auf dem Dokument (Feld 12 08). Enthält im Beispiel eine Broker-Referenz plus **eine** Rechnungsnummer — für den vollständigen Rechnungssatz **nicht geeignet** | [SCR] |
| **Ergebniscode A1 / A2** | Prüfergebnis des Zolls auf dem AGV. Im Beispiel „A2 – Considered satisfactory". [SOP] verlangt, dass A1 **oder** A2 ausgewiesen ist | [SOP] 3.1, [SCR] |
| **Kern-Daten** | Die Felder, die zwischen Zolldokument und Rechnung übereinstimmen müssen | [PPT] Slide 6/7 |
| **Klärfall** | Vorgang, bei dem mindestens ein Kern-Datum abweicht. Führt zur Anfrage beim Broker | [PPT], [SOP] 2. und 3.1 |
| **Alternativnachweis** | Ersatznachweis, wenn kein AGV zu erlangen ist: Importverzollung Drittland, CMR, Air Waybill, Zahlungsnachweis mit Rechnungsbezug, CIM-Frachtbrief, Bill of Lading | [SOP] 4.1, Art. 312 UZK-DA |
| **Englmayer** | `G. ENGLMAYER ZOLL U.CONSULTING`, Wels; Declarant-ID `ATEOS1000000131`. Anmeldender Zolldienstleister, Absender der ABD und AGV | [SCR], [SOP] |
| **Sammelanmeldung** | Eine Ausfuhranmeldung, in der mehrere Rechnungen gemeinsam angemeldet sind. Im Realbeispiel **sieben** Handelsrechnungen in einer MRN | [SCR] Feld 12 03 |

### 2.2 Rechtlicher Rahmen und Fristen

**BELEGT** aus [SOP] 1.1 und 4:

- Der AGV muss vor oder mit der Umsatzsteuervoranmeldung vorliegen, **spätestens 6 Monate nach Lieferung**
- Aufbewahrungspflicht: **7 Jahre (AT)**, **10 Jahre (DE)**
- Der AGV ist „zeitnah, in der Regel innerhalb von **drei Monaten** nach dem Ausfuhrdatum" einzuholen
- Eskalationsmodell: **6 Wochen** Monitoring → **9 Wochen** Alternativnachweise einholen → **12 Wochen** Antrag auf nachträgliche Ausgangsbestätigung
- Alternativnachweise sichern die Umsatzsteuerfreiheit, **ersetzen aber nicht** den Ausgangsvermerk als zollrechtlichen Nachweis

→ **OP-11:** Diese drei Fristen (6 Monate / 3 Monate / 6-9-12 Wochen) sind nicht widersprüchlich, aber für eine Statusberechnung muss **eine** steuernde Frist benannt werden.

### 2.3 Segmente

Der Prozess unterscheidet drei Geschäftsfälle mit unterschiedlichen Regeln:

| Segment | Beschreibung | SAP-Selektion | Besonderheit |
|---|---|---|---|
| **Drittland** | Ausfuhren an Drittlandkunden | Query-Variante `DRITTL 20` | Standardregeln |
| **CH** | Ausfuhren an NovaTaste Switzerland AG | Query-Variante `92 CH` | **Kein Excel-/Sicht-Link seit 01.08.2025**, Anlage nur in SAP unter der Rechnungsnummer ([SOP] 1.2) |
| **Direktanlieferung CH** | NovaTaste Austria liefert direkt an Endkunden der NovaTaste Schweiz | Transaktion `ZIV2`, Variante `WIBERG Swiss Fakturen` | Angemeldet wird eine **„IV2 Faktura Zoll"** (z. B. `9922006057`), die auf dem Beleg als „Rechnung" bezeichnet ist — bekannte Verwechslungsquelle ([SOP] 1.2) |

---

## 3. Geschäftsziele und Erfolgskriterien

| Ziel | Messbares Erfolgskriterium | Heute |
|---|---|---|
| Z-1 Vollständigkeit sicherstellen | Für 100 % der ausfuhrpflichtigen Fakturen existiert eine Zeile mit Status | hängt daran, dass jemand den Excel-Refresh nicht vergisst |
| Z-2 Manuellen Aufwand senken | Vorgänge ohne Abweichung laufen ohne Benutzereingriff durch | 15–25 Klicks je Vorgang |
| Z-3 Fehler vermeiden | Keine Fehlzuordnung durch Spaltenversatz, keine Doppelablage | Spaltenversatz-Workaround belegt ([KD] 17), Doppelanlage belegt ([SCR] zu [KD] 32) |
| Z-4 Fristen einhalten | Kein Vorgang überschreitet die Eskalationsfrist unbemerkt | manuelles Filtern zweimal monatlich |
| Z-5 Prüfungssicherheit | Zu jeder Faktura Nachweis, Prüfprotokoll und Historie auffindbar | Nachweis vorhanden, Prüfprotokoll und Historie nicht |
| Z-6 Wartbarkeit | Änderungen an Regeln ohne Codeänderung; keine Bindung an GUI-Layouts | GUI-gebunden |

---

## 4. Prozesslandkarte

Neun Teilprozesse, zwei Auslöser-Klassen, drei Zeitpläne.

```
        ZEIT (täglich)                    EREIGNIS (Mail-Eingang)            ZEIT (2x monatlich)
              │                                     │                               │
              ▼                                     ▼                               ▼
   ┌─────────────────────┐              ┌────────────────────────┐        ┌──────────────────┐
   │ P1 Soll-Menge       │              │ P2 Eingang &           │        │ P7 Monitoring &  │
   │    aus SAP          │              │    Klassifikation      │        │    Follow-up     │
   └──────────┬──────────┘              └───────────┬────────────┘        └────────┬─────────┘
              │                                     │                              │
              │                   ┌─────────────────┼──────────────┬───────────┐   │
              │                   ▼                 ▼              ▼           ▼   │
              │            ┌────────────┐    ┌────────────┐  ┌──────────┐ ┌────────────┐
              │            │ P3 ABD     │    │ P4 AGV     │  │ P6 DHL   │ │ P8 Alter-  │
              │            │            │    │            │  │          │ │ nativnachw.│
              │            └─────┬──────┘    └─────┬──────┘  └────┬─────┘ └─────┬──────┘
              │                  │                 │              │             │
              └──────────────────┴─────────────────┴──────────────┴─────────────┘
                                          │
                                          ▼
                              ┌───────────────────────┐        ┌─────────────────────┐
                              │ P5 Kern-Daten-        │───────▶│ P9 Ausnahme-        │
                              │    Abgleich           │  Abw.  │    behandlung       │
                              └───────────┬───────────┘        └─────────────────────┘
                                          │ OK
                                          ▼
                              ┌───────────────────────┐
                              │ Ablage: Laufwerk,     │
                              │ SAP-Anlage, Status    │
                              └───────────────────────┘
```

| ID | Teilprozess | Auslöser | Ergebnis |
|---|---|---|---|
| **P1** | Soll-Menge aus SAP laden | Zeit, täglich (OP-12) | Vorgangszeilen mit Rechnungs- und Lieferdaten |
| **P2** | Eingang und Klassifikation | Eingehende E-Mail | Typisiertes Zolldokument, zugeordnet zu 1..n Fakturen |
| **P3** | ABD-Prozess | aus P2 | MRN am Vorgang vermerkt, Summenprüfung, Mail archiviert |
| **P4** | AGV-Prozess | aus P2 | PDF am Laufwerk, SAP-Anlage, Status „archiviert" |
| **P5** | Kern-Daten-Abgleich | aus P3/P4/P6 | Prüfprotokoll je Feld, Ergebnis OK oder Klärfall |
| **P6** | DHL-Mustersendungen | aus P2 | Proforma ermittelt, Bescheinigung angehängt, Seitenreferenz |
| **P7** | Monitoring und Follow-up | Zeit, 2× monatlich | Arbeitsvorrat, Anforderungsmails, Eskalationspaket |
| **P8** | Alternativnachweise | aus P2 bzw. manuell | Ablage, SAP-Anlage, Statusvermerk |
| **P9** | Ausnahmebehandlung | aus P5 | Klärfall, Brokeranfrage, Korrelation der Berichtigung |

**Nicht als eigener Teilprozess geführt:** „Stammdatenpflege" im engeren Sinn existiert nicht. Was in [KD] 1–21 und 88–113 wie Stammdatenpflege aussieht, ist tatsächlich P1 (Soll-Menge) — die Pflege einer Liste, nicht von Stammdaten. Segment- und Regelparameter (Varianten, Fristen, Toleranzen, Zielpfade) werden als **Konfiguration** geführt, siehe Abschnitt 8.5.

---

## 5. Datenmodell

Der Prozesszustand liegt in einem eigenen Speicher. **Grundregel:** Die Automatisierung schreibt ausschließlich in `T_VORGANG`, `T_ZOLLDOKUMENT`, `T_DOKUMENT_FAKTURA` und `T_PRUEFPROTOKOLL`. Der Fachbereich schreibt ausschließlich in `T_MANUELL`. Es gibt keine Zeile, um die beide konkurrieren.

### 5.1 T_VORGANG — eine Zeile je Faktura (Soll-Menge)

| Feld | Typ | Quelle | Bemerkung |
|---|---|---|---|
| `faktura_nr` | text, **PK** | SAP `VBRK-VBELN` | Natürlicher Schlüssel |
| `faktura_art` | text | SAP `VBRK-FKART` | Belegt: `F2`, `F8`; bei ZIV2 `IV2` |
| `faktura_datum` | date | SAP | |
| `ausfuhr_datum` | date | **OP-33** | Bezugsdatum für alle Fristen |
| `segment` | enum | Ableitung aus Selektionsvariante | `Drittland` \| `CH` \| `Direktanlieferung_CH` |
| `ausfuhrland` | text(2) | SAP, Spalte `ELnd` | |
| `nettowert` | decimal | SAP `VBRK-NETWR` | **OP-05:** netto oder brutto? |
| `waehrung` | text(3) | SAP `VBRK-WAERK` | `USD` kommt vor |
| `regulierer` | text | SAP | |
| `regulierer_name` | text | SAP `KNA1`, Spalte `Name 1` | |
| `warenempfaenger_id` | text | **OP-06** | Partnerrolle offen |
| `warenempfaenger_name` | text | **OP-06** | |
| `positionen_anzahl` | int | HYPOTHESE: Anzahl `VBRP`-Zeilen | **OP-01/03** |
| `bruttogewicht_kg` | decimal | HYPOTHESE: `VBRP-BRGEW` bzw. `LIKP-BTGEW` | **OP-03/04** |
| `packstuecke_anzahl` | int | HYPOTHESE: `LIKP-ANZPK` oder Handling Units | **OP-02/03** — steht auf keinem heute genutzten Dokument |
| `sap_beleg_deeplink` | text | generiert | Für die Sicht |
| `soll_geladen_am` | timestamp | P1 | |
| `letzte_aenderung` | timestamp | System | |

### 5.2 T_ZOLLDOKUMENT — eine Zeile je eingegangenem Dokument

| Feld | Typ | Bemerkung |
|---|---|---|
| `dokument_id` | uuid, **PK** | |
| `dokument_typ` | enum | `ABD` \| `AGV` \| `DHL_Bescheinigung` \| `Alternativnachweis` |
| `mrn` | text(18) | Prüfmuster siehe 6.4. Bei DHL/Alternativnachweis leer |
| `ergebnis_code` | text | `A1` \| `A2` \| weitere (**OP-28**) |
| `format` | enum | `XML` \| `PDF` |
| `eingang_am` | timestamp | Mail-Empfangszeit |
| `mail_message_id` | text | **Idempotenzschlüssel.** Verhindert Doppelverarbeitung |
| `absender` | text | |
| `pfad_pdf` | text | Ablageort, Dateiname = MRN |
| `pfad_xml` | text | Rohdatenablage, falls XML |
| `status` | enum | `eingegangen` → `in_pruefung` → `validiert` → `archiviert` \| `klaerfall` \| `verworfen` |
| `validiert_am` | timestamp | |
| `klaerfall_grund` | text | Kurztext, welches Feld abweicht |
| `korrelation_mrn` | text | Verweis auf Vorgängerdokument bei Berichtigung |

### 5.3 T_DOKUMENT_FAKTURA — n:m, der fachliche Kern

Ein Zolldokument kann mehrere Fakturen betreffen (Sammelanmeldung), eine Faktura kann mehrere Dokumente haben (ABD, AGV, ggf. Alternativnachweis). **Diese Beziehung ist der Grund, warum eine Excel-Zeile je Faktura fachlich zu eng ist.**

| Feld | Typ | Bemerkung |
|---|---|---|
| `dokument_id` | uuid, FK | |
| `faktura_nr` | text, FK | |
| `dokumenttyp_code` | text | aus Feld 12 03, z. B. `N380` = Handelsrechnung |
| `abgleich_ergebnis` | enum | `OK` \| `Abweichung` \| `nicht_pruefbar` |
| `abweichende_felder` | text[] | Liste der Feld-IDs |

### 5.4 T_PRUEFPROTOKOLL — Nachvollziehbarkeit des Abgleichs

| Feld | Typ |
|---|---|
| `dokument_id`, `faktura_nr` | FK |
| `feld_id` | text, z. B. `14 06` |
| `wert_dokument` | text |
| `wert_sap` | text |
| `ergebnis` | enum `OK` \| `Abweichung` \| `nicht_pruefbar` |
| `geprueft_am` | timestamp |

Dieses Protokoll ersetzt den heutigen visuellen Abgleich als Prüfungsnachweis. Es ist **neu** und im Ist-Prozess nicht vorhanden.

### 5.5 T_MANUELL — die einzige vom Fachbereich beschreibbare Tabelle

| Feld | Typ | Bemerkung |
|---|---|---|
| `faktura_nr` | text, **PK** | Muss in `T_VORGANG` existieren, sonst Hinweis |
| `manueller_status` | enum | Werteliste in 5.7. Übersteuert den berechneten Status |
| `kommentar` | text | Freitext |
| `bearbeiter` | text | |
| `geaendert_am` | timestamp | |

→ **OP-32:** Welche Statuswerte darf der Fachbereich setzen, und soll ein manueller Status Folgeaktionen auslösen (z. B. „Alternativnachweis erhalten" → automatisch in SAP anhängen)?

### 5.6 Berechneter Vorgangsstatus

Der Status wird **nicht gespeichert**, sondern bei jeder Abfrage berechnet. Vorteil: keine Statusleichen, keine nächtlichen Umbuchungsjobs.

Auswertungsreihenfolge — der erste zutreffende Fall gewinnt:

| Rang | Bedingung | Status | Ampel |
|---|---|---|---|
| 1 | `T_MANUELL.manueller_status` gesetzt | dieser Wert | je Wert |
| 2 | AGV mit Status `archiviert` vorhanden | `AGV archiviert` | grün |
| 3 | Offener Klärfall vorhanden | `Klärfall` | rot |
| 4 | Alter ≥ 12 Wochen | `Eskalation – nachträglicher AGV` | rot |
| 5 | Alter ≥ 9 Wochen | `Alternativnachweis anfordern` | orange |
| 6 | Alter ≥ 6 Wochen | `Monitoring` | gelb |
| 7 | ABD vorhanden, AGV fehlt | `ABD erfasst – AGV offen` | gelb |
| 8 | sonst | `offen` | gelb |

`Alter in Wochen = abgerundet((heute − ausfuhr_datum) / 7)`

### 5.7 Zulässige manuelle Statuswerte (Vorschlag, **OP-32**)

| Wert | Bedeutung | Ampel |
|---|---|---|
| `Kein Nachweis erforderlich` | z. B. Nullrechnung, Mustersendung ohne Warenwert | grün |
| `Alternativnachweis angefordert` | Anfrage an Kundenbetreuung raus | orange |
| `Alternativnachweis erhalten` | Ersatznachweis liegt vor und ist abgelegt | grün |
| `Klärfall an Broker` | Berichtigung angefordert | rot |
| `DHL_Seite<n>` | Nachweis über DHL-Sammelbescheinigung, Seite n | grün |
| `manuell geprüft` | Fachbereich hat trotz technischer Abweichung freigegeben | grün |

---

## 6. Kern-Daten-Regelwerk

Dies ist der fachliche Kern der Automatisierung: der Vergleich zwischen Zolldokument und SAP.

### 6.1 Grundsatz: XML verarbeiten, PDF archivieren

Mit Entscheidung **E-4** gilt:

- **Verarbeitung** ausschließlich aus dem **XML**. Keine PDF-Extraktion für den Abgleich
- **Ablage und Nachweiswirkung** weiterhin über das **PDF** — es ist das Dokument, das Prüfer und Zollbehörde erwarten, und es ist Gegenstand der 7- bzw. 10-jährigen Aufbewahrung
- Deshalb: **XML und PDF müssen gemeinsam eintreffen**, mit der MRN als Korrelationsschlüssel (**OP-17**)
- Fällt das XML aus oder ist es unvollständig → **kein** stiller Rückfall auf PDF-Extraktion, sondern Klärfall. Ein Rückfallpfad, der nur selten läuft, ist ein Pfad, der im Fehlerfall nicht funktioniert

**Wichtig:** Das SAP-generierte Rechnungs-PDF wird **nicht** ausgelesen. Die Rechnungsdaten kommen strukturiert aus SAP. Das im Ist-Prozess sichtbare Öffnen des Rechnungs-PDF im SAP Document Viewer ([KD] 33) entfällt vollständig — es existiert nur, weil ein Mensch etwas zum Hinschauen braucht.

### 6.2 Feldreferenz Zolldokument

**BELEGT** aus [SCR] (ABD und AGV im Vergleich) und [PPT] Slide 5–7. Beide Dokumenttypen verwenden **dasselbe Formularlayout**; unterschieden werden sie durch das Titelband.

| Feld-ID | Bezeichnung | Beispielwert | Verwendung |
|---|---|---|---|
| — | Titelband links | `EXPORT ACCOMPANYING DOCUMENT` / `EXPORT NOTIFICATION – A2 Considered satisfactory` | **Typunterscheidung ABD/AGV** |
| — | BCP MRN | `26AT520000AI0VRAA1` | Schlüssel, Dateiname, Korrelation |
| `13 01` | Exporter | `NOVATASTE AUSTRIA GMBH`, ID `ATEOS1000000594` | Kern-Datum |
| `13 02` | Consignor | dito | Kern-Datum |
| `13 03` | Consignee | `NOVATASTE SWITZERLAND AG, CH-9536 Schwarzenbach` | Kern-Datum |
| `13 05` | Declarant | `G. ENGLMAYER ZOLL U.CONSULTING`, ID `ATEOS1000000131` | Absenderprüfung |
| `12 03` | Supporting document | `1 N380 0092679257; 2 N380 0092679258; … 8 2VVMT` | **Quelle der Rechnungsnummern** |
| `12 08` | LRN | `83294737/92679259…` | nicht für den vollständigen Rechnungssatz geeignet |
| `14 05` | Inv. Cur. | `EUR` | Kern-Datum Währung |
| `14 06` | Total Amount invoiced | `59.154,71` | Kern-Datum Betrag bzw. Summe |
| `18 04` | Gross mass | `13.023,000` | Kern-Datum laut [PPT] |
| — | Total items | `29` | Kern-Datum laut [SOP]/[KD] — **Feld-ID unbekannt, OP-01** |
| — | Total packages | `36` | Kern-Datum laut [SOP]/[KD] als „Paletten" — **OP-02** |
| `16 07` | Country Export | `DE` | Kontext, siehe OP-18 |
| `17 01` | Customs office of exit | `DE004101` | Kontext |
| `15 08` | Presentation of goods date and time | `2026-03-06T08:13:00` | Kandidat für `ausfuhr_datum`, **OP-33** |

### 6.3 Abgleichregeln

**Der zentrale Widerspruch:** Drei Quellen nennen drei unterschiedliche Kern-Daten-Sets.

| Feld | [KD] Schritt 34 | [SOP] 3.1 | [PPT] Slide 6/7 |
|---|---|---|---|
| Consignee | ✔ | ✔ | ✔ |
| Consignor / Exporter | ✔ | ✔ | ✔ |
| Betrag | ✔ | ✔ | ✔ |
| Währung | ✔ | ✔ | — |
| Anzahl Positionen (Items) | ✔ | ✔ | — |
| Anzahl Paletten | ✔ | ✔ | — |
| **Gross mass** | — | — | ✔ |
| Supporting document | (als Quelle) | (als Quelle) | ✔ |

Auf dem Formular stehen `Total items` und `Total packages` unmittelbar neben den in [PPT] grün markierten Feldern und sind dort **bewusst nicht markiert**. Entweder wurde die Regel geändert oder [PPT] ist unvollständig. → **OP-01, blockierend.**

Bis zur Klärung gilt für die Spezifikation der **Vereinigungsmenge** als Prüfumfang, mit je Feld konfigurierbarer Härte:

| # | Feld-ID | Vergleich Zolldokument ↔ SAP | Härte | Toleranz | Status |
|---|---|---|---|---|---|
| R-1 | `12 03` | Rechnungsnummern → müssen in `T_VORGANG` existieren | **hart** | — | Regel klar |
| R-2 | `14 06` | Betrag bzw. **Summe** der Beträge aller in R-1 gefundenen Fakturen | **hart** | **OP-05** | Toleranz offen |
| R-3 | `14 05` | Währung | **hart** | — | **OP-05** bei Fremdwährung |
| R-4 | `13 03` | Consignee ↔ Warenempfänger | **hart** | Namens-/Adressnormalisierung nötig | **OP-06** |
| R-5 | `13 01/02` | Consignor ↔ Rechnungsersteller | **hart** | — | **OP-07** |
| R-6 | `18 04` | Gross mass ↔ Bruttogewicht | **weich** | **OP-04** | Feldherkunft offen |
| R-7 | Total items | Positionsanzahl | **weich** | 0 | **OP-01** |
| R-8 | Total packages | Packstücke/Paletten | **weich** | 0 | **OP-01/02/03** |

**Semantik von hart/weich:** Abweichung bei einer harten Regel → Klärfall, keine Ablage. Abweichung bei einer weichen Regel → Ablage erfolgt, Vorgang wird zusätzlich als „mit Hinweis" markiert und erscheint im Monitoring. Die Zuordnung hart/weich ist **Konfiguration**, nicht Code.

**Wichtige Präzisierung zu R-4** ([SOP] 3.1, wörtlich): „Normalerweise ist der Consignee identisch mit der Invoice Address, sofern Rechnungs- und Warenempfänger dieselbe Partei sind. Es kann jedoch vorkommen, dass beim Consignee andere Daten stehen – beispielsweise die Adresse des Warenempfängers. In diesem Fall ist es wichtig, dass Warenempfänger aus der Rechnung und Consignee exakt übereinstimmen." → Verglichen wird gegen den **Warenempfänger**, nicht gegen den Regulierer.

### 6.4 Parsing-Regeln

**Rechnungsnummern aus Feld 12 03** (BELEGT anhand des Realbeispiels):

```
Rohwert:  "1 N380 0092679257; 2 N380 0092679258; 3 N380 0092679259; 8 2VVMT
           4 N380 0092679260; 5 N380 0092679261; 6 N380 0092679262; 7 N380 0092679263"

Regel:    1. an ";" trennen
          2. je Eintrag: laufende Nummer, Dokumententyp-Code, Dokumentnummer
          3. nur Einträge mit Typcode N380 (Handelsrechnung) verwenden
          4. führende Nullen der Dokumentnummer entfernen
          5. Kardinalität 1..n, im Beispiel n=7

Ergebnis: 92679257, 92679258, 92679259, 92679260, 92679261, 92679262, 92679263
```

Im XML entfällt Schritt 1–2, weil es ein wiederholtes Element mit Typattribut ist (**OP-17**: Elementnamen aus dem XSD).

Nicht-`N380`-Einträge (im Beispiel `2VVMT`) werden protokolliert, aber nicht als Rechnung interpretiert.

**MRN-Validierung:** 18 Zeichen, Muster `^\d{2}[A-Z]{2}[A-Z0-9]{14}$` (HYPOTHESE, aus den Beispielen `26AT520000CI8CG9B3`, `26AT520000AI0VRAA1`, `26AT520000IKKBBNB5` abgeleitet). Der Dateiname der Ablage ist `<MRN>.pdf`. Ein in [PPT] Slide 10 genanntes Beispiel `26AT520000UCX3BB5.pdf` hat nur 17 Zeichen — vermutlich ein Tippfehler in der Präsentation, die Validierung würde ihn abweisen.

### 6.5 Klassifikation ABD ↔ AGV

**BELEGT:**

| Merkmal | ABD | AGV |
|---|---|---|
| Absender | namentliche Deklarant:in bei Englmayer | `donotreply@zoll-beratung.at` |
| Betreff | kein Muster dokumentiert (**OP-29**) | `Benachrichtigung ueber den Austritt zu MRN <MRN> / <Rechnungsnr> - Ergebnis A2 Als konform betrachtet` |
| Anhang-Dateiname | Zufallstoken, z. B. `3XLHLJAFT2SE0M.pdf` | Zufallstoken, z. B. `X5WD5PYT8ZFSKA.pdf` |
| Titelband im Dokument | `EXPORT ACCOMPANYING DOCUMENT` | `EXPORT NOTIFICATION` |

**Regel:** Die **Klassifikation erfolgt am Dokument** (XML-Typ bzw. Titelband), nicht an der Mail. Absender und Betreff dienen der Vorsortierung und Plausibilisierung. Der Dateiname ist als Merkmal **unbrauchbar** — er ist ein Zufallstoken.

Zusätzlich ist zu prüfen, dass ein Ergebniscode `A1` oder `A2` vorliegt ([SOP] 3.1). Der Ist-Prozess prüft das nur über den Mail-Betreff ([KD] 24) — die Spezifikation verlangt die Prüfung am Dokument. → **OP-28** für andere Codes.

---

## 7. Teilprozesse im Detail

Jeder Teilprozess folgt derselben Gliederung: Zweck · Auslöser · Ablauf · Geschäftsregeln · Fehlerfälle · Aufwand · Rückbezug auf [KD].

---

### P1 — Soll-Menge aus SAP laden

**Zweck.** Die Automatisierung muss wissen, welche Rechnungen überhaupt einen Nachweis brauchen. Das ist die Voraussetzung dafür, das **Fehlen** eines Dokuments überhaupt feststellen zu können. Ohne P1 gibt es kein Monitoring, nur Ablage.

**Auslöser.** Zeit, geplant. Vorschlag: arbeitstäglich früh. → **OP-12**

**Ablauf.**

| # | System | Aktion |
|---|---|---|
| 1.1 | Speicher | Wasserstand lesen: höchste bereits geladene Fakturanummer bzw. letztes Ladedatum je Segment |
| 1.2 | SAP | Fakturen ab Wasserstand lesen, je Segment (`Drittland`, `CH`, `Direktanlieferung_CH`) |
| 1.3 | Automat | Felder gemäß 5.1 abbilden; `segment` aus der Selektion setzen |
| 1.4 | Speicher | Neue Zeilen einfügen, bestehende bei geänderten Werten aktualisieren (Upsert auf `faktura_nr`) |
| 1.5 | Speicher | Wasserstand fortschreiben; Ladelauf protokollieren (Anzahl neu, Anzahl aktualisiert, Dauer) |

**Geschäftsregeln.**

- **P1-R1** Der Ladelauf ist **idempotent**. Ein zweimaliger Lauf erzeugt keine Duplikate. Schlüssel ist `faktura_nr`.
- **P1-R2** Der Lesezugriff muss **zusätzlich einzeln nach Fakturanummer** aufrufbar sein. Grund: ein Zolldokument kann eintreffen, bevor die zugehörige Soll-Zeile geladen ist. Der Dokumentenpfad ergänzt die Zeile dann selbst, statt zu scheitern. Dies ist eine **neue** Anforderung gegenüber [KD].
- **P1-R3** Segmente werden getrennt geladen und getrennt fortgeschrieben ([SOP] 1.2: „Rechnungen von NovaTaste Austria/Germany an NovaTaste Schweiz sowie an die weiteren Exportkunden müssen differenziert und separat ausgewertet werden").
- **P1-R4** Der Tagesüberlapp des Ist-Prozesses entfällt. Heute wird der letzte Aktualisierungstag erneut abgefragt, weil Fakturen desselben Tages nachträglich entstehen können, und die Duplikate werden manuell durch Markieren ab der Folgezeile weggeschnitten ([KD] 11–21, [SOP] 1.2). Mit Wasserstand plus Upsert ist beides unnötig.

**Fehlerfälle.**

| Fall | Verhalten |
|---|---|
| SAP nicht erreichbar | Lauf abbrechen, Wasserstand **nicht** fortschreiben, Alarm; nächster Lauf holt nach |
| Feld leer, das für den Abgleich gebraucht wird | Zeile anlegen, Feld als `nicht_pruefbar` markieren; betrifft weiche Regeln |
| Faktura wird in SAP nachträglich storniert | **OP-34** — im Ist-Prozess nicht behandelt |

**Aufwand: mittel.** Die Query existiert (`SQ00`, „Fakturan und Kundendaten VBRK und KNA1", Varianten `DRITTL 20` / `92 CH`), liefert aber nur Kopf- und Kundendaten. Zu ergänzen sind Warenempfänger, Positionsanzahl, Bruttogewicht und Packstücke sowie ein maschinenlesbarer Zugriff. Das ist Modellierung und Konfiguration, kein Neubau — aber ein bewusster Implementierungsschritt, der ohne das SAP-Team nicht geht. **Begründung des Mehraufwands:** ohne diese Felder ist der Kern-Daten-Abgleich (P5) nicht durchführbar, und damit hätte die Automatisierung keinen fachlichen Wert — sie würde nur Dateien verschieben.

**Rückbezug [KD]:** ersetzt Schritte 1–21 (Drittland) und 88–113 (CH) vollständig, einschließlich des Spaltenversatz-Workarounds in Schritt 17.

---

### P2 — Eingang und Klassifikation

**Zweck.** Eingehende Nachrichten typisieren, das Zolldokument extrahieren und den betroffenen Fakturen zuordnen.

**Auslöser.** Neue E-Mail im Postfach `ausfuhren@novataste.com`.

**Ablauf.**

| # | System | Aktion |
|---|---|---|
| 2.1 | Mailbox | Neue Nachricht erkennen; `mail_message_id` als Idempotenzschlüssel prüfen — bereits verarbeitet → verwerfen |
| 2.2 | Automat | Anhänge extrahieren: XML und PDF |
| 2.3 | Automat | Dokumenttyp aus dem XML bestimmen (bzw. Titelband, solange PDF); Ergebniscode lesen |
| 2.4 | Automat | MRN lesen und validieren (6.4) |
| 2.5 | Automat | Rechnungsnummern aus Feld `12 03` parsen (6.4) |
| 2.6 | Speicher | `T_ZOLLDOKUMENT` anlegen, Status `eingegangen`; `T_DOKUMENT_FAKTURA` je Rechnungsnummer anlegen |
| 2.7 | Speicher | Für jede Rechnungsnummer prüfen, ob eine Soll-Zeile existiert; wenn nicht → P1-R2 (Einzelabruf) |
| 2.8 | — | Weiterleitung an P3 (ABD), P4 (AGV), P6 (DHL) oder P8 (Alternativnachweis) |

**Geschäftsregeln.**

- **P2-R1** Idempotenz über `mail_message_id` **und** über die Kombination `mrn` + `dokument_typ`. Eine erneut zugestellte Mail darf keinen zweiten Vorgang erzeugen.
- **P2-R2** Ist zur MRN bereits ein Dokument desselben Typs vorhanden, gilt das neue als **Berichtigung**: `korrelation_mrn` setzen, den bestehenden Klärfall auflösen statt einen neuen anzulegen (siehe P9).
- **P2-R3** Fehlt das XML oder ist es gegen das Schema ungültig → Klärfall, **kein** Rückfall auf PDF-Extraktion (6.1).
- **P2-R4** Nicht klassifizierbare Nachricht → Klärfall-Warteschlange mit Grund `Klassifikation unmöglich`. Nie stilles Verwerfen.
- **P2-R5** Eine Nachricht kann mehrere Anhänge tragen. Ob „eine Mail = eine Anmeldung" garantiert ist, ist offen → **OP-29**.

**Aufwand: gering** unter Annahme E-4 (XML). Mit XML sind Klassifikation und Extraktion trivial, weil Typ und Felder codiert sind. **Ohne XML: mittel**, weil PDF-Extraktion, Layouterkennung und ein Confidence-Gate nötig wären. Das ist der konkrete Gegenwert der XML-Umstellung.

**Rückbezug [KD]:** ersetzt Schritte 22–27 (AGV) und 114–117 (ABD).

---

### P3 — ABD-Prozess

**Zweck.** Das Ausfuhrbegleitdokument gegen die angemeldeten Rechnungen prüfen und die MRN am Vorgang vermerken. Das ABD wird **nicht** archiviert.

**Auslöser.** Aus P2, `dokument_typ = ABD`.

**Ablauf.**

| # | System | Aktion |
|---|---|---|
| 3.1 | Speicher | Betroffene Fakturen aus `T_DOKUMENT_FAKTURA` lesen |
| 3.2 | SAP | Rechnungsdaten je Faktura strukturiert lesen (keine PDF-Anzeige) |
| 3.3 | P5 | Kern-Daten-Abgleich durchführen, inkl. **Summenprüfung** über alle Fakturen der MRN |
| 3.4 | Speicher | Bei Ergebnis OK: `mrn` am Vorgang als ABD-MRN vermerken, Dokumentstatus `validiert` |
| 3.5 | Mailbox | Mail in den definierten Ordner verschieben (**OP-14**) |
| 3.6 | Speicher | Bei Abweichung → P9 |

**Geschäftsregeln.**

- **P3-R1 Summenprüfung.** Bei mehr als einer Faktura je MRN muss die **Summe der Rechnungsbeträge** dem Feld `14 06` entsprechen ([KD] 116, 170–174; im aufgezeichneten Fall Summe 50.242,96 gegen drei Rechnungen). Diese Regel ist **nur in [KD] belegt** und fehlt in [SOP] und [PPT] — sie ist trotzdem fachlich zwingend.
- **P3-R2** Das ABD wird **weder am Laufwerk noch in SAP archiviert** ([SOP] 2., wörtlich). Es wird ausschließlich im Postfach abgelegt. Begründung: das ABD hat seinen Zweck mit bestätigtem Austritt erfüllt; nachweispflichtig ist der AGV.
- **P3-R3** Die ABD-MRN wird am Vorgang vermerkt. **BELEGT** durch [SCR]: die heutige Excel führt dafür die eigene Spalte **J „ABD"** (Werte wie `26AT520000YABLSLB2`), getrennt von Spalte **K „Austritt"** für den AGV-Link. Damit ist der scheinbare Widerspruch zwischen [PPT] („Excel: MRN aus dem ABD einpflegen") und [KD] (schreibt nur im AGV-Pfad) aufgelöst: **zwei Spalten, zwei Prozessschritte.**
- **P3-R4** Bei Diskrepanz: E-Mail an den Broker `kundenteam1@zoll-beratung.at` mit Bitte um Berichtigung ([SOP] 2.).

**Aufwand: gering.** Die Logik ist einfach; der Aufwand liegt in P5 und im SAP-Lesezugriff, die ohnehin gebraucht werden. Die Summenprüfung wird **billiger** als heute, weil sie aus SAP statt aus einer möglicherweise veralteten Excel-Zeile rechnet.

**Rückbezug [KD]:** ersetzt Schritte 114–180.

---

### P4 — AGV-Prozess

**Zweck.** Den Austrittsnachweis prüfen, revisionssicher ablegen und den Vorgang abschließen.

**Auslöser.** Aus P2, `dokument_typ = AGV`.

**Ablauf.**

| # | System | Aktion |
|---|---|---|
| 4.1 | Automat | Ergebniscode prüfen: `A1` oder `A2` erforderlich (**OP-28**) |
| 4.2 | Speicher | Betroffene Fakturen lesen |
| 4.3 | SAP | Rechnungsdaten strukturiert lesen |
| 4.4 | P5 | Kern-Daten-Abgleich, inkl. Summenprüfung bei Sammelanmeldung |
| 4.5 | Fileshare | PDF ablegen: `<Zielordner>/<MRN>.pdf` |
| 4.6 | SAP | PDF als Anlage am Fakturabeleg anlegen, **Titel = MRN** |
| 4.7 | SAP | Vorher prüfen, ob unter diesem Titel bereits eine Anlage existiert → Duplikatsschutz |
| 4.8 | Speicher | `pfad_pdf`, `pfad_xml`, Status `archiviert`, `validiert_am` setzen |
| 4.9 | Mailbox | AGV-Mail ablegen — **OP-13: im Ist-Prozess nicht dokumentiert** |
| 4.10 | Speicher | Bei Abweichung → P9, **keine** Ablage |

**Geschäftsregeln.**

- **P4-R1 Ablagereihenfolge.** Erst Laufwerk, dann SAP, dann Statusfortschreibung. Grund: die SAP-Anlage referenziert die Datei; ein Abbruch nach Schritt 4.5 ist harmlos und wiederholbar, ein Abbruch nach 4.6 ohne 4.8 wird durch 4.7 abgefangen.
- **P4-R2 Dateiname = MRN.** Zwingend, weil daran die Wiedererkennung hängt ([SOP] 3.1, [KD] 43–55).
- **P4-R3 Zielordner.** `\\wibergs20\Daten\02 Finanz\05 ZOLL\06_Exporte\Austritte gesamt gespeichert <Jahr>`. In [SOP] wird derselbe Ort einmal als UNC-Pfad und einmal als `T:\…` genannt; [KD] nutzt `T:\`. Für die Automatisierung ist **UNC verbindlich** — ein Laufwerksbuchstabe ist benutzerabhängig.
- **P4-R4 Anlagen-Titel.** **BELEGT** durch [SCR] der Anlagenliste: Spalten `Titel | Name des Erstellers | Erst.Datum`; der AGV erscheint als Zeile `26AT520000CI8CG9B3`, die Rechnung als Zeile `Faktura`. Der **Titel** ist damit das unterscheidende Merkmal. → **OP-20**: ist er über die Schnittstelle setzbar?
- **P4-R5 Segmentausnahme CH.** Für Segment `CH` wird **kein Link in der Sicht** erzeugt; die Anlage in SAP ist der einzige Ablageort ([SOP] 1.2, gültig seit 01.08.2025). → **OP-09**: gilt weiterhin, und gilt es auch für `Direktanlieferung_CH`?
- **P4-R6 Ersteller.** Heute steht in der Anlagenliste eine namentliche Bearbeiterin. Mit Automatisierung wird es ein technischer Benutzer. Die Zuordnung „wer hat geprüft" wandert damit in `T_PRUEFPROTOKOLL`. → **OP-22**, prüfungsrelevant.

**Fehlerfälle.**

| Fall | Verhalten |
|---|---|
| Fakturabeleg gesperrt | Retry mit Backoff; nach n Versuchen Klärfall `SAP-Sperre` |
| Ablage am Laufwerk schlägt fehl | Abbruch vor SAP-Zugriff, Wiederholung im nächsten Lauf |
| Anlage existiert bereits (4.7) | Kein zweiter Upload, Status auf `archiviert` setzen, protokollieren |
| MRN entspricht nicht dem Muster | Klärfall `MRN ungültig`, keine Ablage |

**Aufwand: mittel bis hoch, abhängig vom Schreibweg.** Die Ablage am Laufwerk ist gering. Der SAP-Upload ist der teuerste Einzelbaustein: über eine Schnittstelle **mittel**, über UI-Automation **hoch** — nicht in der Erstellung, sondern im Betrieb und in der Wartung. Details und Optionsvergleich in 8.2. **Begründung:** die SAP-Anlage ist für Segment `CH` der einzige Ablageort und damit nicht verzichtbar.

**Rückbezug [KD]:** ersetzt Schritte 22–87.

---

### P5 — Kern-Daten-Abgleich

**Zweck.** Die wertschöpfende Prüfung: stimmen Zolldokument und Rechnung überein?

**Auslöser.** Aufruf aus P3, P4, P6.

**Ablauf.**

| # | Aktion |
|---|---|
| 5.1 | Rechnungsnummern aus Feld `12 03` gegen `T_VORGANG` auflösen (R-1) |
| 5.2 | Fehlt eine Faktura → Einzelabruf aus SAP (P1-R2); bleibt sie unbekannt → Klärfall `Faktura unbekannt` |
| 5.3 | Regeln R-2 bis R-8 je Feld auswerten, Ergebnis je Feld in `T_PRUEFPROTOKOLL` schreiben |
| 5.4 | Gesamtergebnis bilden: harte Abweichung → `Abweichung`; nur weiche → `OK mit Hinweis`; sonst `OK` |
| 5.5 | Ergebnis in `T_DOKUMENT_FAKTURA.abgleich_ergebnis` und `abweichende_felder` festschreiben |

**Geschäftsregeln.**

- **P5-R1** Bei Sammelanmeldung wird `14 06` gegen die **Summe** geprüft, alle übrigen Felder gegen **jede** betroffene Faktura einzeln.
- **P5-R2** Ein Feld, das in SAP nicht verfügbar ist, ergibt `nicht_pruefbar` — nicht `OK`. „Nicht geprüft" darf nie wie „geprüft" aussehen.
- **P5-R3** Normalisierung vor dem Vergleich: Groß-/Kleinschreibung, Mehrfach-Leerzeichen, Rechtsformkürzel und Umlaute bei Namen; Dezimaltrennzeichen und Tausenderpunkte bei Zahlen. → **OP-35**: verbindliche Normalisierungsregel für Namensvergleiche.
- **P5-R4** Das Prüfprotokoll ist unveränderlich. Eine erneute Prüfung erzeugt einen neuen Satz Einträge, sie überschreibt nichts.

**Aufwand: mittel.** Der Code ist überschaubar — eine Regeltabelle und ein Auswerter. Der Aufwand liegt **nicht in der Implementierung, sondern in den Entscheidungen**: OP-01 bis OP-07 müssen beantwortet sein. **Begründung des Mehraufwands gegenüber [KD]:** im Ist-Prozess ist dieser Schritt rein visuell ([KD] 34–42, 151–169) und damit nicht spezifiziert. Ihn zu automatisieren erfordert erstmals eine explizite, vollständige Regeldefinition. Das ist der Kern des Vorhabens — ohne ihn bleibt nur Dateiverschieben.

**Rückbezug [KD]:** ersetzt Schritte 34–42 und 151–169.

---

### P6 — DHL-Mustersendungen

**Zweck.** Bei kostenlosen Mustersendungen kommt der Austrittsnachweis nicht von Englmayer, sondern als **DHL-Ausfuhrbescheinigung** — ein mehrseitiges Sammel-PDF. Der Nachweis muss der zugehörigen **Proformarechnung** zugeordnet werden.

**Auslöser.** Aus P2, Absender DHL, Dateiname im Format `DHL_Express_Ausfuhrbescheinigungen_<DDMMYYYY>`.

**Ablauf** ([SOP] 3.4):

| # | System | Aktion |
|---|---|---|
| 6.1 | Fileshare | Sammel-PDF ablegen im Ordner `Austritte gesamt gespeichert <Jahr>` |
| 6.2 | Automat | Je Bescheinigung die **Lieferscheinnummer** (Referenznummer im DHL-Dokument) lesen |
| 6.3 | SAP | Über den **Belegfluss** zur Lieferung die **Proformarechnung** ermitteln |
| 6.4 | SAP | Bescheinigung als Anlage an der Proformarechnung anlegen |
| 6.5 | Speicher | Statusvermerk `DHL_Seite<n>` — n = Seitennummer im Sammel-PDF |

**Geschäftsregeln.**

- **P6-R1 Vorrangregel.** „**Englmayer hat immer Vorrang (Priority) gegenüber DHL**" ([SOP] 3.4, wörtlich). Liegt bereits ein AGV von Englmayer vor, wird die DHL-Bescheinigung **nicht** zusätzlich abgelegt; es wird lediglich der Kommentar `DHL` gesetzt.
- **P6-R2** Findet sich keine Proformarechnung im Belegfluss: „normale" Rechnung prüfen. Ist es eine **Nullrechnung** → Customer Service kontaktieren. Ist ein Warenwert vorhanden → es ist keine kostenlose Mustersendung, **keine Proforma erforderlich**.
- **P6-R3** Fehlt eine Referenznummer im DHL-Dokument → Rückfrage beim Customer Service.
- **P6-R4** Der erweiterte Belegfluss über den kostenlosen Verkaufsbeleg zeigt die Proforma teilweise erst in zweiter Ebene ([SOP] 3.4).

**Aufwand: hoch.** Begründung: eigenes Dokumentformat ohne XML-Perspektive, Seitenzahl als fachliche Referenz, mehrstufige Belegflussnavigation, mehrere Ausnahmezweige mit menschlicher Rückfrage. Der Teilprozess ist in [KD] **überhaupt nicht enthalten** — es gibt also keine Ist-Aufzeichnung als Basis. **Empfehlung: Phase 2.** In Phase 1 lediglich Erkennung und Ablage der Sammelbescheinigung plus manueller Statusvermerk über `T_MANUELL`. → **OP-25**

---

### P7 — Monitoring und Follow-up

**Zweck.** Sicherstellen, dass zu jeder Ausfuhr ein Nachweis existiert, und die Beschaffung anstoßen, wenn nicht.

**Auslöser.** Zwei getrennte Dinge, die nicht verwechselt werden dürfen:

- **Anzeige** des Status: kein Auslöser nötig, wird bei jeder Abfrage berechnet (5.6)
- **Aktion** (Mails, Eskalationspaket): Zeit, zweimal monatlich zu Monatsbeginn und Monatsmitte ([SOP] 4.1)

**Ablauf der Aktionsstufen** ([SOP] 4.1, 6-9-12-Wochen-Modell):

| Stufe | Bedingung | Aktion | Empfänger |
|---|---|---|---|
| **6 Wochen** | AGV fehlt | nur Sichtbarkeit im Arbeitsvorrat, keine Aussenwirkung | — |
| **9 Wochen** | AGV fehlt | Alternativnachweis anfordern; gesammelte Mail mit Auszug aus der Liste | zuständige Kundenbetreuung, CC `ausfuhren@novataste.com` |
| **9 Wochen, Segment CH** | AGV fehlt | interne Anfrage nach Importverzollungen unter Angabe der Rechnungsnummern | `import@novataste.com` |
| **12 Wochen** | Alternativnachweise vollständig | Übersichtstabelle mit zugeordneten Alternativnachweisen übermitteln | `kundenteam1@zoll-beratung.at` (Englmayer beantragt den nachträglichen AGV) |

**Geschäftsregeln.**

- **P7-R1 Ermittlung der zuständigen Kundenbetreuung:** heute manuell über `VF02`/`VF03` → Kopfdaten → Nachname im oberen Bereich ([SOP] 4.1). Für die Automatisierung: dieses Feld muss in P1 mitgeladen werden. **HYPOTHESE**: entspricht dem Sachbearbeiter/Vertriebsbeauftragten am Fakturakopf — im Rechnungs-PDF erscheint ein Feld `Sales representative` ([SCR]). → **OP-36**
- **P7-R2** Zulässige Alternativnachweise (Art. 312 UZK-DA, [SOP] 4.1): Importverzollung Drittland, CMR-Frachtbrief, Air Waybill, Zahlungsnachweis mit eindeutigem Rechnungsbezug, CIM-Frachtbrief, Bill of Lading.
- **P7-R3** Alternativnachweise sichern die Umsatzsteuerfreiheit, ersetzen aber **nicht** den zollrechtlichen AGV. Der Vorgang bleibt für die Zollsicht offen, bis ein nachträglicher AGV eintrifft.
- **P7-R4** Trifft ein nachträglicher AGV ein, wird er nach P4 verarbeitet — kein Sonderweg ([SOP] 4.1, letzter Absatz).
- **P7-R5** Der heutige Arbeitsschritt „Screenshot aus der Excel-Liste an die Kundenbetreuung" braucht einen Ersatz: ein Export bzw. ein generierter Mailtext (Vorlage in [SOP] 4.1 vorhanden).

**Aufwand: mittel.** Statusberechnung und Arbeitsvorrat sind **gering** (reine Sicht). Die Mailgenerierung inkl. Empfängerermittlung ist **mittel**. **Begründung:** dieser Teilprozess ist in [KD] nicht enthalten und heute rein manuell — er ist aber der Grund, warum es die Liste überhaupt gibt (Ziel Z-4).

---

### P8 — Alternativnachweise

**Zweck.** Eingehende Ersatznachweise prüfen, ablegen und dokumentieren.

**Auslöser.** Aus P2 (Mail an `ausfuhren@novataste.com`) oder manuell.

**Ablauf** ([SOP] 4.2):

| # | System | Aktion |
|---|---|---|
| 8.1 | Mensch | Prüfung auf Vollständigkeit und Plausibilität — **bleibt manuell** |
| 8.2 | Fileshare | Ablage mit Namensschema `Alternativnachweis_<Rechnungsnummer>` |
| 8.3 | SAP | Anlage an der Faktura anlegen |
| 8.4 | Speicher | Statusvermerk `Alternativnachweis erhalten`, Verweis auf die Datei |

**Geschäftsregeln.**

- **P8-R1** Liegt für denselben Zeitraum bereits ein AGV (A2) vor, sind **keine weiteren Maßnahmen** erforderlich ([SOP] 4.2).
- **P8-R2** Die Bewertung, ob ein Dokument ein zulässiger Alternativnachweis ist, ist eine **Fachentscheidung** und wird nicht automatisiert. Die Formate sind heterogen (CMR, AWB, B/L, Kontoauszug) und es gibt keine strukturierte Quelle.
- **P8-R3** [PPT] nennt die Gleichbehandlung mit AGV als „optional" → **OP-30**: gleiche Ablagekette ja/nein?

**Aufwand: gering**, wenn auf Ablage plus Statusvermerk beschränkt. **Hoch**, falls eine inhaltliche Prüfung automatisiert werden soll — davon ist abzuraten: die Dokumentenvielfalt ist unbegrenzt und der fachliche Ermessensspielraum groß. Empfehlung: bewusst manuell lassen, Automatisierung nur für Ablage und Verlinkung.

---

### P9 — Ausnahmebehandlung und Klärfälle

**Zweck.** Der Teilprozess, den der Ist-Prozess **nicht kennt**. [KD] enthält keinen einzigen Abweichungspfad; [SOP] nennt nur die Brokeranfrage; [PPT] fordert „Klärfall inkl. klarer Hinweismeldung (welches Kern-Feld nicht passt)".

**Auslöser.** Abweichung aus P5, oder technischer Fehler aus P1/P2/P4/P6.

**Ablauf.**

| # | Aktion |
|---|---|
| 9.1 | Klärfall anlegen: Dokument, betroffene Faktura(en), Grund, abweichende Feld-IDs, Soll- und Ist-Wert |
| 9.2 | Kategorisieren (Tabelle unten) und dem zuständigen Adressaten zuordnen |
| 9.3 | Bei fachlicher Abweichung: Mailentwurf an `kundenteam1@zoll-beratung.at` mit Bitte um Berichtigung |
| 9.4 | Vorgang bleibt in der Klärfall-Warteschlange, Status `Klärfall`, Ampel rot |
| 9.5 | Trifft ein berichtigtes Dokument ein: Korrelation über die MRN (P2-R2), Klärfall auflösen, Prüfung erneut durchlaufen |

**Klärfallkategorien.**

| Kategorie | Ursache | Adressat | Automatische Auflösung möglich |
|---|---|---|---|
| `Kernfeld-Abweichung` | harte Regel verletzt | Broker | ja, bei Berichtigung |
| `Faktura unbekannt` | Nummer aus 12 03 nicht in SAP | Fachbereich | ja, nach P1-Lauf |
| `Summendifferenz` | 14 06 ≠ Summe | Broker | ja |
| `MRN ungültig` | Musterverletzung | Broker | ja |
| `Ergebniscode unerwartet` | nicht A1/A2 | Fachbereich | **OP-28** |
| `XML fehlt oder ungültig` | Formatproblem | Broker | ja |
| `Klassifikation unmöglich` | Typ nicht erkennbar | Fachbereich | manuell |
| `SAP-Sperre` | Beleg gesperrt | technisch | ja, per Retry |
| `Ablage fehlgeschlagen` | Laufwerk/SAP | technisch | ja, per Retry |

**Geschäftsregeln.**

- **P9-R1** Ein Klärfall blockiert **nur den betroffenen Vorgang**, nicht den Lauf. Andere Dokumente derselben Sammelanmeldung werden weiterverarbeitet, sofern sie selbst fehlerfrei sind. → **OP-37**: darf eine Sammelanmeldung teilweise abgeschlossen werden, oder gilt „alles oder nichts"?
- **P9-R2** Klärfälle werden nie automatisch geschlossen, ohne dass eine erneute erfolgreiche Prüfung dokumentiert ist.
- **P9-R3** Der Fachbereich kann einen Klärfall über `T_MANUELL` mit `manuell geprüft` übersteuern. Diese Übersteuerung wird protokolliert und ist in der Sicht sichtbar.

**Aufwand: mittel.** **Begründung des Mehraufwands:** dieser Teilprozess existiert im Ist-Prozess nicht und ist trotzdem zwingend. Ohne ihn müsste die Automatisierung bei jeder Abweichung stehenbleiben und ein Mensch müsste den Zustand rekonstruieren — womit der Nutzen der Automatisierung bei den ersten Problemfällen verloren geht. Die Erfahrung aus dem Ist-Prozess zeigt, dass Abweichungen regelmäßig auftreten: [SOP] beschreibt die Brokeranfrage an zwei Stellen als Regelfall, nicht als Ausnahme.

---

## 8. Technische Architektur

### 8.1 Komponenten

```
   Postfach ausfuhren@novataste.com
              │  (Mail mit XML + PDF)
              ▼
   ┌──────────────────────────────┐
   │  Eingangsverarbeitung        │   Klassifikation, XML-Parsing,
   │  (P2)                        │   Idempotenz über message-id
   └──────────┬───────────────────┘
              │
              ▼
   ┌──────────────────────────────┐        ┌────────────────────────┐
   │  Regelwerk / Abgleich (P5)   │◀──────▶│  SAP  (lesend, OData)  │
   └──────────┬───────────────────┘        └────────────────────────┘
              │
      ┌───────┴────────┐
      ▼                ▼
┌───────────┐   ┌──────────────┐          ┌────────────────────────┐
│ Fileshare │   │  SAP-Anlage  │─────────▶│  SAP  (schreibend)     │
│ <MRN>.pdf │   │  Titel = MRN │          │  Option A/B/C/D → 8.2  │
└───────────┘   └──────────────┘          └────────────────────────┘
      │                │
      └────────┬───────┘
               ▼
   ┌──────────────────────────────┐        ┌────────────────────────┐
   │  Prozessspeicher             │◀──────▶│  T_MANUELL             │
   │  T_VORGANG, T_ZOLLDOKUMENT,  │        │  (Fachbereich schreibt)│
   │  T_DOKUMENT_FAKTURA,         │        └────────────────────────┘
   │  T_PRUEFPROTOKOLL            │
   └──────────┬───────────────────┘
              │
      ┌───────┴────────┬─────────────────┐
      ▼                ▼                 ▼
┌───────────┐  ┌──────────────┐  ┌────────────────┐
│ Excel-    │  │ Klärfall-    │  │ Follow-up-     │
│ Sicht     │  │ Liste        │  │ Mails (P7)     │
└───────────┘  └──────────────┘  └────────────────┘
```

### 8.2 SAP-Integration: Optionen und Empfehlung

**Lesen — Empfehlung eindeutig: OData bzw. ein maschinenlesbarer Service.**

Begründung aus dem Ist-Zustand: die Liste umfasst rund 900 Zeilen und scrollt; die ALV-Spaltenreihenfolge ist variantenabhängig (genau die Ursache des `###`-Problems in [KD] 17); Zahlen und Datumsangaben kommen gebietsformatiert; der Ist-Weg läuft über „Text kopieren" in die Zwischenablage ([KD] 14, 100). Ein Service liefert Typen, Filter, Paging und bricht nur bei Vertragsänderung, nicht bei Layoutänderung.

**Regel: nicht mischen.** Gelesen wird ausschließlich über einen Kanal, sonst existieren zwei Wahrheiten über dieselbe Rechnung.

**Schreiben (Anlage am Fakturabeleg) — vier Optionen:**

| Option | Beschreibung | Aufwand Erstellung | Aufwand Betrieb | Bewertung |
|---|---|---|---|---|
| **A — Schnittstelle/API** | Anlage per freigegebener Schnittstelle anlegen, Titel = MRN | mittel | gering | **Ziel.** Setzt OP-20 voraus |
| **B — Interface-Ordner + ABAP-Job** | Automat legt PDF plus Metadaten in ein Verzeichnis, ein Report im System hängt an und protokolliert | mittel | gering | **Starke Alternative.** Kein technischer Dialoguser, SAP-Team behält Kontrolle über den schreibenden Zugriff — erfahrungsgemäß der schnellste Weg zur Zustimmung |
| **C — webGUI-Automation** | Browserautomatisierung der Transaktion | mittel | mittel | Rückfall. Vorteil gegenüber D: der Dateidialog wird zum HTML-`input type=file` und ist ohne OS-Automation bedienbar |
| **D — SAP GUI Scripting** | Scripting der Windows-GUI | mittel | **hoch** | Ungünstigste Option. Der Dateidialog von „Anlage anlegen" ist ein Windows-Dialog und **nicht** Teil des Scripting-Objektmodells — er erfordert Tastatursimulation. Zusätzlich: dauerhaft angemeldete Windows-Sitzung, Brüche bei GUI-Patches |

**Rangfolge: A > B > C > D.**

Drei Fakten entscheiden, welche Option möglich ist. Sie sind noch offen und sollten **vor** der Festlegung geklärt werden:

- **OP-19** Release und Verfügbarkeit von OData
- **OP-20** Freigegebene Anlagen-Schnittstelle vorhanden? Titel setzbar?
- **OP-21** Ist `sapgui/user_scripting` überhaupt aktiviert? Ist webGUI/ITS aktiv und rendert den Anlagendialog vollständig?

**Sperrverhalten.** Der Upload läuft im Ist-Prozess über `VF02` (Ändern) und sperrt den Fakturabeleg. Für alle **lesenden** Zugriffe ist `VF03` zu verwenden. In [KD] sind bis zu **sieben** parallele SAP-Fenster offen ([KD] 148) — bei automatisiertem Stoßbetrieb würde `VF02` serialisieren. Optionen A und B vermeiden oder verkürzen die Sperre.

### 8.3 Prozessspeicher

**Anforderungen:** transaktionale Schreibzugriffe, Historie, gleichzeitiger Lese- und Schreibzugriff durch Automat und Menschen, Berechtigungen, API für die Automatisierung, native Anbindung an Excel.

| Option | Bewertung |
|---|---|
| **SharePoint-Liste** | **Empfehlung**, sofern M365 durchgängig vorhanden ist ([PPT] belegt Teams-Nutzung). Versionierung, Berechtigungen und Änderungszeitstempel ohne eigenes Hosting; `T_MANUELL` ist in der Rasteransicht praktisch wie Excel bearbeitbar; erscheint in Excel und Power BI von selbst |
| **SQL-Datenbank** | Technisch am saubersten, erfordert aber Hosting, Backup und Betriebskonzept |
| **Excel als Master** | **Scheidet aus.** Automat und Mensch können nicht gleichzeitig in dieselbe Datei schreiben. [SCR] belegt, dass die heutige Datei bereits im Zustand „Schreibgeschützt" geöffnet wird — der Konflikt existiert schon vor jeder Automatisierung |

→ **OP-24**

### 8.4 Sichten für den Fachbereich

**Overlay-Muster.** Der Automat schreibt ausschließlich die Basisdaten, der Fachbereich ausschließlich `T_MANUELL`; die Sicht verbindet beide per LEFT JOIN über `faktura_nr`, wobei der manuelle Eintrag den berechneten Status übersteuert. Es gibt damit keine Zeile, um die zwei Parteien konkurrieren.

| Sicht | Inhalt | Technik |
|---|---|---|
| **Monitoring** | alle Vorgänge mit Status, Ampel, Alter in Wochen, Deeplinks | Excel + Power Query (Vorlage liegt vor: `PowerQuery_Monitoring_E2E.md`) |
| **Offene Vorgänge** | Ampel ≠ grün, sortiert nach Dringlichkeit | dito, gefiltert. Ersetzt den heutigen Excel-Screenshot in der Alternativnachweis-Anforderung |
| **Klärfälle** | Klärfall-Warteschlange mit Grund und abweichenden Feldern | Listenansicht im Speicher |
| **Sammelanmeldungen** | MRN, Anzahl Rechnungen, Summe, Soll aus 14 06 | Gruppierung, ersetzt [KD] 170–174 |
| **Jahresexport** | Snapshot als Archiv- und Notfallartefakt | geplanter Export |

**Bewusst festgehalten:** Die Excel-Sicht ist jederzeit wegwerfbar und in wenigen Minuten neu aufgebaut. Sie enthält keine Daten, die nicht aus dem Speicher kommen. Die Nachweiskraft liegt weiterhin bei den PDF am Laufwerk und den SAP-Anlagen, nicht bei dieser Liste.

### 8.5 Konfiguration statt Code

Folgendes muss ohne Codeänderung anpassbar sein:

| Parameter | Beispielwert | Quelle |
|---|---|---|
| Selektionsvarianten je Segment | `DRITTL 20`, `92 CH`, `WIBERG Swiss Fakturen` | [SOP] 1.2 |
| Zielordner Ablage | `\\wibergs20\Daten\02 Finanz\05 ZOLL\06_Exporte\Austritte gesamt gespeichert <Jahr>` | [SOP] 3. |
| Zielordner ABD-Mails | `2026 ABDs` (**OP-14**) | [KD] 180 |
| Fristenstufen | 6 / 9 / 12 Wochen | [SOP] 4.1 |
| Regeln R-1..R-8 inkl. hart/weich und Toleranz | siehe 6.3 | **OP-01..07** |
| Segmentausnahmen | CH: kein Link | [SOP] 1.2 |
| Adressaten | `kundenteam1@zoll-beratung.at`, `import@novataste.com`, `ausfuhren@novataste.com` | [SOP] |
| Zulässige manuelle Statuswerte | siehe 5.7 | **OP-32** |

### 8.6 Nachvollziehbarkeit und Protokollierung

- Jeder Lauf von P1 protokolliert Zeitraum, Anzahl neu/aktualisiert, Dauer
- Jede Dokumentverarbeitung protokolliert Eingang, Klassifikation, Prüfergebnis je Feld, Ablageorte
- Jede manuelle Übersteuerung protokolliert Benutzer und Zeitstempel
- Klärfälle protokollieren Entstehung und Auflösung inkl. auslösendem Dokument

Dies ist **neu** gegenüber dem Ist-Prozess, in dem die Excel-Liste überschrieben wird und keine Historie existiert.

---

## 9. Aufwandsbilanz gegenüber dem Ist-Prozess

### 9.1 Was der Vorschlag teurer macht — mit Begründung

| # | Neuerung | Aufwand | Begründung |
|---|---|---|---|
| M-1 | **Maschineller Kern-Daten-Abgleich** statt visueller Prüfung | **mittel** | Der Abgleich ist der wertschöpfende Schritt. Er ist heute nirgends spezifiziert, weil ein Mensch hinsieht. Ohne ihn wäre die Automatisierung reines Dateiverschieben. Der Aufwand liegt überwiegend in den **Entscheidungen** (OP-01..07), nicht im Code |
| M-2 | **Erweiterte SAP-Lesesicht** (Warenempfänger, Positionen, Bruttogewicht, Packstücke) | **mittel** | Die bestehende Query liefert nur Kopf- und Kundendaten. Ohne die Zusatzfelder ist M-1 nicht durchführbar. Packstücke stehen auf **keinem** heute genutzten Dokument — ohne diese Klärung ist eine der Kernregeln nicht umsetzbar |
| M-3 | **Persistenter Prozessspeicher** statt Excel-Spalten | **gering bis mittel** | Voraussetzung für Vollständigkeitsprüfung, Idempotenz, Historie und die n:m-Beziehung Dokument↔Faktura, die eine Excel-Zeile je Faktura nicht abbilden kann |
| M-4 | **Klärfallmodell (P9)** | **mittel** | Existiert im Ist-Prozess nicht. Ohne ihn bliebe die Automatisierung bei jeder Abweichung stehen. [SOP] beschreibt die Brokeranfrage an zwei Stellen als Regelfall |
| M-5 | **Idempotenz und Duplikatsprüfung** | **gering** | [SCR] belegt eine bereits vorhandene Anlage vor dem Upload-Schritt — Doppelablage ist real |
| M-6 | **Einzelabruf nach Fakturanummer** (P1-R2) | **gering** | Behandelt den Wettlauf zwischen Zeit- und Ereignistrigger. Ohne ihn scheitert die Verarbeitung, wenn ein Dokument vor dem Ladelauf eintrifft |
| M-7 | **Prüfprotokoll** (`T_PRUEFPROTOKOLL`) | **gering** | Ersetzt den visuellen Abgleich als Prüfungsnachweis. Heute existiert kein Nachweis, *dass* geprüft wurde |
| M-8 | **Explizite Regelparametrierung** (Segmentausnahmen, Fristen, Toleranzen) | **gering** | Heute implizites Erfahrungswissen — z. B. die CH-Ausnahme seit 01.08.2025, die in [KD] und [PPT] fehlt |
| M-9 | **DHL-Teilprozess (P6)** | **hoch** | Eigenes Format, Seitenreferenz, mehrstufiger Belegfluss, mehrere menschliche Rückfragezweige, keine Ist-Aufzeichnung. **Empfehlung: Phase 2** |
| M-10 | **ZIV2/IV2-Mapping** | **mittel bis hoch** | Die angemeldete Nummer ist keine Kundenrechnung. Mapping unklar (**OP-23**). **Empfehlung: Phase 2** |
| M-11 | **Aufbewahrungs-/Exportartefakt** | **gering** | Ersetzt die Excel-Datei als Prüfungsartefakt und dient als Notfallsicht |

### 9.2 Was der Vorschlag billiger macht

| Entfällt | Fundstelle im Ist-Prozess |
|---|---|
| Spaltenversatz-Workaround mit STRG+X/STRG+V und `###`-Erkennung | [KD] 17–21, [SOP] 1.2 |
| Manuelles Markieren ab der Folgezeile mit STRG+Y, Tagesüberlapp | [SOP] 1.2 |
| Öffnen und visuelles Auslesen des Rechnungs-PDF im SAP Document Viewer | [KD] 30–33, 37, 42, 120–125, 153 |
| Bis zu sieben parallele SAP-GUI-Fenster | [KD] 126–150 |
| Hyperlink-Choreographie in Excel (Suchen, Nach Datei suchen, Dateiname einfügen, OK, OK) | [KD] 71–87 |
| Manuelle Summenbildung in Excel bei Sammelanmeldungen | [KD] 170–174 |
| Manuelles Filtern des Arbeitsvorrats zweimal monatlich | [SOP] 4.1 |
| Manuelles Speichern der Excel-Datei nach jeder Änderung | [SOP] 3.3 |

**Netto:** Der Mehraufwand liegt fast vollständig in **Klärung und Regeldefinition**, nicht in Implementierung. Die eigentliche Codemenge sinkt gegenüber einer naiven 1:1-Nachbildung von [KD] erheblich, weil rund ein Drittel der 180 Ist-Schritte reine GUI-Navigation ist, die entfällt.

### 9.3 Phasenempfehlung

| Phase | Inhalt | Voraussetzung |
|---|---|---|
| **Phase 0** | Klärung der blockierenden offenen Punkte; SAP-Call zu Baustein A und B | OP-01..07, 11, 19..21, 33 |
| **Phase 1** | P1, P2, P3, P4, P5, P7, P9 für Segmente `Drittland` und `CH`; Speicher; Excel-Sicht | Phase 0 |
| **Phase 2** | P6 (DHL), `Direktanlieferung_CH` / ZIV2, P8 mit vollständiger Ablagekette | Phase 1 stabil |
| **Phase 3** | Ablösung der Excel-Sicht durch eine Anwendung, falls der Klärfall-Workflow es rechtfertigt | Erfahrungswerte aus Phase 1 |

---

## 10. Nicht-Ziele

Bewusst **nicht** Gegenstand dieser Automatisierung:

- Kein Eingriff in den Fakturierungsprozess, keine neuen Felder in SAP-Bestandstabellen
- Keine Änderung der Zollanmeldung oder der Zusammenarbeit mit dem Broker im Anmeldeprozess
- Keine automatisierte inhaltliche Bewertung von Alternativnachweisen (P8-R2)
- Keine Extraktion aus dem SAP-generierten Rechnungs-PDF (6.1)
- Keine Migration der Altjahre in den neuen Speicher; Altjahre werden eingefroren (**OP-26**)
- Keine Ersetzung fachlicher Entscheidungen in Zweifelsfällen

---

## 11. Offene Punkte

**Legende Blocker:** ● blockierend für Implementierungsstart · ○ nicht blockierend

| ID | Frage | Adressat | Blocker |
|---|---|---|---|
| **OP-01** | Welches Kern-Daten-Set ist verbindlich? [PPT] markiert Gross mass und *nicht* Items/Packages; [SOP]/[KD] nennen Items und Paletten, kein Gewicht. Bitte je Feld: prüfen ja/nein, hart/weich | Fachbereich | ● |
| **OP-02** | Ist `Total packages` = Paletten oder = Kartons/Packstücke? | Fachbereich | ● |
| **OP-03** | Woher kommt die Packstück-/Palettenzahl in SAP (`LIKP-ANZPK`, Handling Units, anderes)? Sie steht auf **keinem** heute genutzten Dokument | IT/SAP | ● |
| **OP-04** | Gewichtsvergleich: `18 04` Gross mass gegen welches SAP-Feld? Brutto↔Brutto oder Brutto↔Netto+Tara? Toleranz? | Fachbereich + IT/SAP | ● |
| **OP-05** | Betragsvergleich: `VBRK-NETWR` oder Bruttowert? Toleranz in Cent oder Prozent? Verhalten bei Fremdwährung (`USD` kommt vor) und bei `14 09` Exch. Rate? | Fachbereich | ● |
| **OP-06** | Welche SAP-Partnerrolle ist der zollrechtliche Warenempfänger (Vergleich gegen `13 03` Consignee)? | IT/SAP | ● |
| **OP-07** | Consignor/Exporter `13 01/02`: Vergleich gegen Buchungskreis oder gegen Partnerrolle? | IT/SAP | ○ |
| **OP-08** | Wird die MRN im ABD-Pfad in Spalte „ABD" geführt und im AGV-Pfad als Link in „Austritt"? Belegt durch [SCR], bitte bestätigen. Was gilt, wenn ein AGV **ohne** vorheriges ABD eintrifft? | Fachbereich | ○ |
| **OP-09** | CH-Ausnahme „kein Link seit 01.08.2025" — gilt weiterhin? Gilt sie auch für `Direktanlieferung_CH` (ZIV2)? | Fachbereich | ○ |
| **OP-10** | Ist die SAP-Anlage rechtlich zwingend oder Prüfungskomfort? Bleibt die Dreifachablage (Laufwerk, SAP, Sicht)? | Fachbereich | ○ |
| **OP-11** | Welche Frist ist steuernd für „überfällig" — 6 Monate (USt), 3 Monate ([SOP] 4.), oder 12 Wochen (Eskalation)? | Fachbereich | ● |
| **OP-12** | Takt für P1: arbeitstäglich? [SOP] definiert keinen Takt | Fachbereich | ○ |
| **OP-13** | Was passiert mit der **AGV-E-Mail** nach Verarbeitung? Im Ist-Prozess nicht dokumentiert | Fachbereich | ○ |
| **OP-14** | Exakter Zielordner der ABD-Mails: „2026 ABDs" ([KD]) / „Ordner 202(6)" ([SOP]) / „vordefiniert" ([PPT])? | Fachbereich | ○ |
| **OP-15** | Ist die Outlook-Kategorie „Ungeprüft" ([KD] 24) prozessrelevanter Status oder Beiwerk? | Fachbereich | ○ |
| **OP-16** | Weitere Broker außer Englmayer, insbesondere für DE-Ausfuhren? | Fachbereich | ○ |
| **OP-17** | XML: XSD, Versionierung mit Vorankündigung, Original des Zollsystems oder Broker-Derivat, XML **und** PDF in derselben Mail? | Broker/Englmayer | ● |
| **OP-18** | MRN beginnt mit `26AT…` (Anmeldung AT), `16 07 Country Export = DE`, Zollstellen `DE004101`/`DE007458`. Ein oder zwei Zollsysteme als Quelle? | Fachbereich + Broker | ○ |
| **OP-19** | SAP-Release (ECC oder S/4), Verfügbarkeit von OData bzw. eines Lesedienstes | IT/SAP | ● |
| **OP-20** | Freigegebene Schnittstelle für Anlagen (GOS/ArchiveLink)? Ist der **Titel** setzbar? | IT/SAP | ● |
| **OP-21** | Ist `sapgui/user_scripting` aktiviert? Ist webGUI/ITS aktiv und rendert den Anlagendialog? | IT/SAP | ● |
| **OP-22** | Technischer Benutzer als „Name des Erstellers" in der Anlagenliste — prüfungsrechtlich akzeptiert? | Fachbereich | ○ |
| **OP-23** | ZIV2 / „IV2 Faktura Zoll": wie mappt eine angemeldete Nummer wie `9922006057` auf einen Fakturabeleg? | IT/SAP | ○ |
| **OP-24** | Speichertechnologie: SharePoint-Liste, SQL, anderes? Governance und Berechtigungen | IT + Fachbereich | ● |
| **OP-25** | Scope Phase 1: sind DHL (P6), ZIV2 und Alternativnachweise (P8) dabei? | Fachbereich | ● |
| **OP-26** | Migration: Altjahre einfrieren und ab Stichtag neu starten — einverstanden? | Fachbereich | ○ |
| **OP-27** | Was bedeutet der Suchbegriff `EDTBX` ([KD] 73)? Kunden- oder Empfängerkürzel? | Fachbereich | ○ |
| **OP-28** | Kommen andere Ergebniscodes als A1/A2 vor? Wie sind sie zu behandeln? | Fachbereich + Broker | ○ |
| **OP-29** | Ist „eine Mail = eine Anmeldung" garantiert? Gibt es ein Betreffmuster für ABD-Mails? | Broker | ○ |
| **OP-30** | Alternativnachweise: gleiche Ablagekette wie AGV (Laufwerk, SAP, Link)? [PPT] nennt es „optional" | Fachbereich | ○ |
| **OP-31** | Aufbewahrungs- und Löschkonzept für 7 Jahre (AT) / 10 Jahre (DE) — wer verantwortet es? | Fachbereich | ○ |
| **OP-32** | Welche Statuswerte darf der Fachbereich setzen? Soll ein manueller Status Folgeaktionen auslösen? | Fachbereich | ○ |
| **OP-33** | Welches Datum ist Bezugspunkt der Fristenrechnung — Fakturadatum, Lieferdatum, oder `15 08` Presentation of goods date? Im Ist-Prozess **nirgends** definiert | Fachbereich | ● |
| **OP-34** | Verhalten bei nachträglicher Storno oder Änderung einer Faktura in SAP | Fachbereich | ○ |
| **OP-35** | Verbindliche Normalisierungsregel für Namens- und Adressvergleiche (R-4, R-5) | Fachbereich | ○ |
| **OP-36** | Zuständige Kundenbetreuung: entspricht sie dem `Sales representative` am Fakturakopf? | IT/SAP | ○ |
| **OP-37** | Darf eine Sammelanmeldung teilweise abgeschlossen werden, oder gilt „alles oder nichts"? | Fachbereich | ○ |

**Blockierend: 14 Punkte** — OP-01, 02, 03, 04, 05, 06, 11, 17, 19, 20, 21, 24, 25, 33. Diese vierzehn sind im Phase-0-Termin abzuarbeiten; ohne sie kann die Implementierung nicht sinnvoll beginnen.

---

## 12. Anhang

### 12.1 Rückverfolgbarkeit: Ist-Schritte aus [KD] → Soll-Prozess

| [KD] Schritte | Ist-Tätigkeit | Soll |
|---|---|---|
| 1–3 | Excel-Masterliste öffnen | entfällt |
| 4–16 | SQ00-Query, Variante, Zeitraum, Text kopieren, in Excel einfügen | **P1** |
| 17–21 | Spaltenversatz korrigieren, speichern | entfällt |
| 22–24 | Postfach, Posteingang, AGV-Mail auswählen | **P2** |
| 25–26 | Anhang öffnen | **P2** |
| 27 | Rechnungsnummer aus `Supporting documents` kopieren | **P2** (Parsing 6.4) |
| 28–33 | VF02, Anlagenliste, Faktura anzeigen | **P4/P5** (strukturierter Lesezugriff, kein PDF) |
| 34–42 | Datenabgleich visuell | **P5** |
| 43–55 | MRN kopieren, Speichern unter, Zielordner, Dateiname | **P4** Schritt 4.5 |
| 56–68 | SAP, Ändern, Anlage anlegen, Datei wählen, Sichern | **P4** Schritt 4.6 |
| 69–87 | Excel: Rechnung suchen, MRN eintragen, Hyperlink setzen | **P4** Schritt 4.8 (Statusfortschreibung) |
| 88–113 | CH: eigene Variante, Datum, kopieren, Blatt „CH", speichern | **P1** Segment `CH` |
| 114–116 | ABD-Mail, Anhang öffnen, Kontrolle | **P2/P3** |
| 117–150 | je Rechnung SAP öffnen, anzeigen, abgleichen (3 Rechnungen, 7 Fenster) | **P3/P5** |
| 151–169 | Datenabgleich ABD über alle Seiten | **P5** |
| 170–176 | Blatt „Drittland", Summenkontrolle gegen `14 06` | **P5** Regel R-2 / Sicht „Sammelanmeldungen" |
| 177–180 | ABD-Mail in Unterordner ablegen | **P3** Schritt 3.5 |

**Nicht in [KD] enthalten, aber fachlich erforderlich:** P6 (DHL), P7 (Follow-up), P8 (Alternativnachweise), P9 (Klärfälle), Ablage der AGV-Mail.

### 12.2 Belegte Strukturen aus den Screenshots

**SAP-Query** ([KD] 13): `SQ00`, Report „Fakturan und Kundendaten VBRK und KNA1".
Spalten: `Faktura | FkArt | Fakturadatum | Angel.am | BuKr | ELnd | Nettowert | Währg | Regulierer | Name 1`, mit Summenzeile.

**Excel-Spalten** ([KD] 16): `E ELnd | F Nettowert | G Währung | H Regulierer | I Name 1 | J ABD | K Austritt`.
Spalte **J „ABD"** enthält MRN-Werte im Klartext, Spalte **K „Austritt"** den Hyperlink auf das AGV-PDF. Zeilen mit gefülltem J und leerem K sind der Zustand „AGV fehlt".

**SAP-Anlagenliste** ([KD] 32): Dialog „Dienst: Anlagenliste", Kopf „Anlagen zu `0092681928`".
Spalten: `Ikone | Titel | Name des Erstellers | Erst.Datum`. Zeilen im Beispiel: `26AT520000CI8CG9B3` / Ines Dupanovic / 03.07.2026 sowie `Faktura` / ohne Ersteller / 08.04.2026.

**Rechnungs-PDF** ([KD] 33, SAP Document Viewer): enthält `Invoice no.`, `Invoice date`, `Delivery note no.`, `Invoice address`, `Net weight`, `Currency`, `Sales representative` sowie Positionszeilen mit `Article no.`, `Material`, `Quantity (PC)`, `Net weight`, `Price`, `Value`. **Keine Palettenzahl, kein Bruttogewicht.**

**Postfachstruktur** ([KD] 22): freigegebene Postfächer `Import`, `Exportcontrol`, `Trade Compliance Info`, `TariffCodes`, `Ausfuhren`.

### 12.3 Hinweis zur Verlässlichkeit von [KD]

Zwei Einschränkungen, die bei der Interpretation zu beachten sind:

1. **Kein sauberer Erstdurchlauf.** Im Screenshot zu Schritt 32 ist der AGV `26AT520000CI8CG9B3` bereits als Anlage vorhanden (Erstellerin, Datum 03.07.2026), obwohl der Upload erst in den Schritten 59–68 erfolgt. Der Vorgang war zum Aufzeichnungszeitpunkt schon einmal bearbeitet. Die Schrittfolge ist daher als Ablaufbeschreibung, nicht als lückenlose Transaktionsspur zu lesen.
2. **Stichprobe n=1.** Die Aufzeichnung zeigt einen AGV und ein ABD mit drei Rechnungen. Das ausgewertete Realbeispiel eines Zolldokuments enthält jedoch **sieben** Handelsrechnungen in Feld `12 03`. Die Kardinalität ist 1..n und darf nicht aus der Aufzeichnung abgeleitet werden.

### 12.4 Beispieldaten und Vorlagen

Ergänzend liegen vor:

- `Monitoring_Basisdaten.csv` — 22 Beispielvorgänge über alle Segmente und Statusstufen
- `Monitoring_Manuell.csv` — Beispiele der manuellen Nachbearbeitung
- `PowerQuery_Monitoring_E2E.md` — lauffähige Vorlage der Excel-Sicht inkl. M-Code für Join, Aging, Status, Ampel und Summenprüfung

Die Statuslogik aus 5.6 wurde gegen diese Beispieldaten nachgerechnet. Die Summenprüfung einer Sammelanmeldung ergibt **50.242,96** — derselbe Wert, der in [KD] Schritt 174 manuell in Excel ermittelt wurde.

---

*Ende der Spezifikation. Rückfragen und Korrekturen bitte unter Angabe der Abschnitts- oder OP-Nummer.*
