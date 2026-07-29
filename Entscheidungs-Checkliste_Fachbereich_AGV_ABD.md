# Entscheidungs-Checkliste Fachbereich
## Automatisierung Archivierung Austrittsnachweise (AGV / ABD)

**Für:** Trade Compliance · **Stand:** Juli 2026 · **Bezug:** `Automation_Implementation_Specification_AGV_ABD.md`

---

## Wie diese Liste zu verwenden ist

Die Analyse des Prozesses ist abgeschlossen. Was jetzt fehlt, sind **fachliche Entscheidungen**, die niemand außer dem Fachbereich treffen kann. Erst danach kann programmiert werden.

Diese Liste ist so gebaut, dass sie **ohne die Spezifikation lesbar** ist. Zu jeder Frage steht, warum sie gestellt wird, was die vorhandenen Unterlagen dazu sagen und was die Antwort bewirkt.

| Kennzeichen | Bedeutung |
|---|---|
| ⛔ | **Blockierend.** Ohne diese Antwort kann nicht begonnen werden |
| ⚪ | Kann während der Umsetzung nachgezogen werden |

**Aufwand:** In einem Workshop von 90 Minuten sind die Blöcke A, B und E realistisch zu klären. Die Blöcke C, D und F können schriftlich nachgereicht werden.

**Nicht in dieser Liste:** Zehn weitere offene Punkte richten sich an das SAP-Team bzw. an Englmayer. Sie stehen zur Information im Anhang, brauchen aber keine Antwort vom Fachbereich.

---

## Die acht Entscheidungen, die alles andere bestimmen

Wer nur wenig Zeit hat, sollte mit diesen beginnen:

| Nr. | Entscheidung |
|---|---|
| A1 ⛔ | Welche Felder müssen übereinstimmen, damit ein Nachweis als geprüft gilt? |
| A3/A4 ⛔ | Welche Toleranz gilt bei Gewicht und Betrag? |
| B1 ⛔ | Welche Frist ist steuernd für „überfällig"? |
| B2 ⛔ | Welches Datum ist der Startpunkt der Fristenrechnung? |
| C1 ⚪ | Bleibt die Dreifachablage (Laufwerk, SAP, Liste)? |
| D1 ⛔ | Womit wird das Monitoring künftig gemacht? |
| E1 ⛔ | Was ist in der ersten Ausbaustufe dabei? |
| A2 ⛔ | Bedeutet „Anzahl Paletten" Paletten oder Packstücke? |

---

# Block A — Wann gilt ein Nachweis als geprüft?

Dies ist der fachliche Kern. Heute vergleicht eine Kollegin das Zolldokument und die Rechnung mit dem Auge. Damit eine Maschine das übernimmt, muss die Regel vollständig aufgeschrieben sein.

### A1 ⛔ Welche Felder müssen verglichen werden?

**Warum wir fragen.** Die drei vorliegenden Unterlagen nennen drei unterschiedliche Listen.

| Feld | Klick-Doku | Arbeitsanweisung | Präsentation |
|---|---|---|---|
| Consignee (Warenempfänger) | ✔ | ✔ | ✔ |
| Consignor / Exporter (Versender) | ✔ | ✔ | ✔ |
| Rechnungsbetrag | ✔ | ✔ | ✔ |
| Währung | ✔ | ✔ | — |
| Anzahl Positionen | ✔ | ✔ | — |
| Anzahl Paletten | ✔ | ✔ | — |
| **Bruttogewicht** | — | — | ✔ |

Auf dem Zolldokument stehen „Total items" und „Total packages" direkt neben den in der Präsentation grün markierten Feldern — und sind dort **nicht** markiert. Entweder hat sich die Regel geändert oder die Präsentation ist unvollständig.

**Bitte je Feld entscheiden:**

| Feld | prüfen? | Bei Abweichung: Ablage stoppen oder nur Hinweis? |
|---|---|---|
| Consignee | ☐ ja ☐ nein | ☐ stoppen ☐ Hinweis |
| Consignor / Exporter | ☐ ja ☐ nein | ☐ stoppen ☐ Hinweis |
| Rechnungsbetrag | ☐ ja ☐ nein | ☐ stoppen ☐ Hinweis |
| Währung | ☐ ja ☐ nein | ☐ stoppen ☐ Hinweis |
| Anzahl Positionen | ☐ ja ☐ nein | ☐ stoppen ☐ Hinweis |
| Anzahl Paletten / Packstücke | ☐ ja ☐ nein | ☐ stoppen ☐ Hinweis |
| Bruttogewicht | ☐ ja ☐ nein | ☐ stoppen ☐ Hinweis |

**Was die Antwort bewirkt.** „Stoppen" heißt: der Nachweis wird nicht abgelegt, es entsteht ein Klärfall und eine Anfrage an Englmayer. „Hinweis" heißt: die Ablage erfolgt, der Vorgang erscheint aber mit Warnung im Monitoring.

---

### A2 ⛔ Bedeutet „Anzahl Paletten" tatsächlich Paletten?

**Warum wir fragen.** Auf dem Zolldokument heißt das Feld `Total packages` — im Beispiel 36 Stück bei 13.023 kg. Die Arbeitsanweisung nennt es „Paletten-Anzahl". Packstücke können aber auch Kartons oder Gebinde sein. Wird gegen die falsche Größe verglichen, entstehen Dauer-Abweichungen.

☐ `Total packages` = Paletten
☐ `Total packages` = Packstücke allgemein (Kartons, Gebinde), nicht Paletten
☐ unklar — bitte gemeinsam an einem Realbeispiel klären

**Anmerkung:** Diese Zahl steht auf **keinem** der heute verwendeten Dokumente auf der SAP-Seite. Woher sie technisch kommt, klären wir parallel mit dem SAP-Team.

---

### A3 ⛔ Gewicht: welche Basis, welche Toleranz?

**Warum wir fragen.** Das Zolldokument führt ein **Bruttogewicht** (`Gross mass`, im Beispiel 13.023,000 kg). Das Rechnungs-PDF aus SAP zeigt dagegen ein **Nettogewicht** (im Beispiel 809,600 kg auf Seite 1 von 3). Beides direkt zu vergleichen ist fachlich falsch.

☐ Brutto gegen Brutto — SAP-Bruttogewicht wird herangezogen
☐ Brutto gegen Netto plus Verpackungsgewicht
☐ Gewicht wird **nicht** verglichen

Toleranz, falls verglichen: ☐ exakt ☐ ± ______ kg ☐ ± ______ %

---

### A4 ⛔ Betrag: netto oder brutto, welche Toleranz, was bei Fremdwährung?

**Warum wir fragen.** Das Zolldokument führt `Total Amount invoiced`. Die SAP-Auswertung liefert einen **Nettowert**. Bei Sammelanmeldungen ist der Betrag die **Summe** mehrerer Rechnungen — im aufgezeichneten Beispiel drei Rechnungen mit Summe 50.242,96 €. In der Rechnungsliste kommen außerdem Fremdwährungen vor (im Beispiel eine Rechnung in USD).

Vergleichsbasis: ☐ Nettowert ☐ Bruttowert ☐ anderes: ______

Toleranz: ☐ exakt auf den Cent ☐ ± ______ € ☐ ± ______ %

Fremdwährung: ☐ Vergleich in Originalwährung ☐ Umrechnung anhand des Kurses auf dem Zolldokument ☐ Fremdwährungsfälle grundsätzlich manuell prüfen

---

### A5 ⚪ Wie streng soll der Namensvergleich sein?

**Warum wir fragen.** Namen und Adressen sind auf Zolldokument und Rechnung selten zeichengleich — Rechtsformkürzel, Abkürzungen, Umlaute, Zeilenumbrüche. Eine zu strenge Regel erzeugt Dauer-Klärfälle, eine zu lockere übersieht echte Fehler.

☐ Nur Firmenname vergleichen, Groß-/Kleinschreibung und Leerzeichen ignorieren
☐ Firmenname und Land vergleichen
☐ Firmenname, Land und Ort vergleichen
☐ Vollständige Adresse vergleichen

**Wichtiger Hinweis aus der Arbeitsanweisung:** Verglichen wird der Consignee gegen den **Warenempfänger** der Rechnung — nicht gegen den Rechnungsempfänger. Beide können abweichen. Bitte bestätigen: ☐ richtig ☐ so nicht

---

### A6 ⚪ Sammelanmeldung: alles oder nichts?

**Warum wir fragen.** Ein Zolldokument kann mehrere Rechnungen betreffen — in einem ausgewerteten Realbeispiel **sieben**. Weicht eine davon ab, ist zu entscheiden, was mit den übrigen passiert.

☐ Die fehlerfreien Rechnungen werden abgeschlossen, nur die abweichende wird Klärfall
☐ Alles oder nichts — bei einer Abweichung bleibt die gesamte Anmeldung offen

---

# Block B — Ab wann ist ein Nachweis überfällig?

### B1 ⛔ Welche Frist ist steuernd?

**Warum wir fragen.** In den Unterlagen stehen drei Fristen, die nicht widersprüchlich sind, aber für eine automatische Statusberechnung muss **eine** maßgeblich sein:

- **6 Monate nach Lieferung** — Nachweis muss vor bzw. mit der Umsatzsteuervoranmeldung vorliegen
- **3 Monate nach Ausfuhrdatum** — „zeitnah einzuholen"
- **6 / 9 / 12 Wochen** — Monitoring, Alternativnachweis anfordern, Eskalation

☐ Das 6-9-12-Wochen-Modell ist maßgeblich, die anderen Fristen sind Kontext
☐ Andere Stufen: ______ / ______ / ______ Wochen
☐ Zusätzlich eine harte Warnung bei ______ Monaten (Umsatzsteuer-Sicht)

---

### B2 ⛔ Welches Datum ist der Startpunkt?

**Warum wir fragen.** Alle Fristen zählen „ab Ausfuhrdatum". In keiner der Unterlagen ist definiert, welches Feld das ist. Die Kandidaten liegen zeitlich auseinander, teilweise um Wochen.

☐ Fakturadatum
☐ Lieferdatum aus SAP
☐ Datum der Gestellung beim Zoll (auf dem Zolldokument)
☐ Datum aus der Ausfuhranmeldung: ______
☐ anderes: ______

**Was die Antwort bewirkt.** Dieses Datum bestimmt, wann ein Vorgang gelb, orange und rot wird — und damit, wie viel Nacharbeit entsteht.

---

# Block C — Wo wird was abgelegt?

### C1 ⚪ Bleibt die Dreifachablage?

**Warum wir fragen.** Heute wird jeder AGV an drei Stellen geführt: als PDF am Laufwerk unter der MRN, als Anlage am SAP-Fakturabeleg und als Link in der Excel-Liste. Ob die SAP-Anlage rechtlich zwingend ist oder ein Prüfungskomfort, ist in den Unterlagen nicht begründet.

☐ Alle drei Ablagen bleiben unverändert
☐ Laufwerk und Verweis genügen, SAP-Anlage entfällt
☐ SAP-Anlage genügt, Laufwerksablage entfällt
☐ Entscheidung braucht Rücksprache mit Steuer/Recht

**Zu beachten:** Für die Schweiz ist die SAP-Anlage seit 01.08.2025 der **einzige** Ablageort (siehe C2). Ein Verzicht darauf hätte dort besondere Wirkung.

---

### C2 ⚪ Gilt die Schweiz-Ausnahme weiterhin?

**Warum wir fragen.** Die Arbeitsanweisung hält fest: „Seit dem 01.08.2025 wird kein Link beim AGV mehr für die Schweiz in der Tabelle angelegt. Anhang bleibt nur in der SAP unter der RE Nr." Weder Klick-Doku noch Präsentation kennen diese Ausnahme — eine Automatisierung würde sie ohne Bestätigung übergehen.

☐ Ausnahme gilt weiterhin
☐ Ausnahme ist überholt, Schweiz wird wie Drittland behandelt

Gilt die Ausnahme auch für **Direktanlieferungen** an Endkunden der Schweiz (ZIV2)?
☐ ja ☐ nein ☐ unklar

---

### C3 ⚪ Was passiert mit der AGV-E-Mail nach der Verarbeitung?

**Warum wir fragen.** Für die ABD-Mail ist die Ablage dokumentiert. Für die AGV-Mail findet sich in keiner Unterlage ein Ablageschritt — der Ablauf endet mit dem Eintrag in der Liste.

☐ In einen Unterordner verschieben, Name: ______________________
☐ Im Posteingang belassen und nur als gelesen markieren
☐ Kategorie setzen: ______________________
☐ bisher tatsächlich nicht geregelt — bitte jetzt festlegen

---

### C4 ⚪ Wie heißt der Zielordner der ABD-Mails genau?

**Warum wir fragen.** Drei Unterlagen, drei Bezeichnungen: „2026 ABDs", „Ordner 202(6)", „vordefinierter Ordner". Die Automatisierung braucht den exakten Pfad.

Vollständiger Pfad im Postfach `ausfuhren@novataste.com`: _______________________________________

Wie wird der Ordner beim Jahreswechsel benannt? ☐ automatisch nach Jahr ☐ wird manuell angelegt

---

### C5 ⚪ Aufbewahrung und Löschung

**Warum wir fragen.** Die Arbeitsanweisung nennt 7 Jahre (Österreich) und 10 Jahre (Deutschland). Ein Verfahren, was nach Ablauf passiert, ist nicht beschrieben.

☐ Es gibt bereits ein Löschkonzept, verantwortlich: ______________________
☐ Es gibt keines — bleibt außerhalb dieses Vorhabens
☐ Soll im Rahmen dieses Vorhabens mitgeregelt werden

Zusätzlich: soll jährlich ein **archivierbarer Export** der Nachweisliste erzeugt werden (als Prüfungsartefakt und Notfallsicht)? ☐ ja ☐ nein

---

### C6 ⚪ Alternativnachweise: gleiche Ablagekette wie ein AGV?

**Warum wir fragen.** Die Arbeitsanweisung beschreibt Ablage am Laufwerk als `Alternativnachweis_<Rechnungsnummer>`, Anlage in SAP und Verweis in der Liste. Die Präsentation nennt die Gleichbehandlung dagegen „optional".

☐ Ja, identische Kette wie beim AGV
☐ Nur Ablage am Laufwerk und Statusvermerk, keine SAP-Anlage
☐ Nur Statusvermerk

**Nicht automatisiert wird** die inhaltliche Beurteilung, ob ein Dokument als Alternativnachweis taugt. Die Formate sind zu heterogen (CMR, Air Waybill, Bill of Lading, Zahlungsnachweis) und die Entscheidung ist fachlich. Bitte bestätigen: ☐ einverstanden ☐ anders gewünscht

---

# Block D — Wie soll das Monitoring aussehen?

### D1 ⛔ Womit wird künftig überwacht?

**Warum wir fragen.** Die Excel-Liste kann als führendes System entfallen — die Vollständigkeitsprüfung und das Monitoring müssen aber erhalten bleiben. Wichtig: eine Maschine und ein Mensch können nicht gleichzeitig in dieselbe Excel-Datei schreiben. Die heutige Datei wird bereits im Zustand „Schreibgeschützt" geöffnet, wenn sie jemand offen hat.

Vorgeschlagenes Modell: die Daten liegen in einer Liste, die die Automatisierung pflegt. Der Fachbereich pflegt eine **eigene, kleine Nachbearbeitungstabelle**. Beide werden in einer Excel-Sicht zusammengeführt, die sich beim Öffnen aktualisiert. Damit bleibt Filtern, Pivotieren und der Screenshot für die Kundenbetreuung genau wie heute.

☐ Einverstanden mit diesem Modell
☐ Wir wollen eine eigene Anwendung mit Oberfläche statt einer Excel-Sicht
☐ Wir wollen bei einer Excel-Datei als führendem System bleiben (Konsequenzen bitte besprechen)

---

### D2 ⛔ Welche Vermerke soll der Fachbereich setzen können?

**Warum wir fragen.** Für Rand- und Sonderfälle braucht der Fachbereich weiterhin die Möglichkeit, den automatisch berechneten Status zu übersteuern. Eine kurze, gepflegte Werteliste ist dabei besser als Freitext — sonst lässt sich nicht mehr filtern.

Vorschlag, bitte streichen oder ergänzen:

☐ `Kein Nachweis erforderlich` — z. B. Nullrechnung, Mustersendung ohne Warenwert
☐ `Alternativnachweis angefordert`
☐ `Alternativnachweis erhalten`
☐ `Klärfall an Broker`
☐ `DHL_Seite<n>`
☐ `manuell geprüft` — Freigabe trotz technischer Abweichung
☐ weitere: ______________________

Soll ein solcher Vermerk eine Folgeaktion auslösen? Beispiel: `Alternativnachweis erhalten` → Dokument automatisch in SAP anhängen.
☐ ja ☐ nein, reiner Vermerk

Zusätzlich ein Freitextfeld für Kommentare? ☐ ja ☐ nein

---

### D3 ⚪ Wie oft soll die Soll-Liste aus SAP nachgeladen werden?

**Warum wir fragen.** Heute passiert das, wenn jemand daran denkt — die Arbeitsanweisung sagt nur „vom letzten Aktualisierungstag bis zum heutigen Datum". Für einen automatischen Lauf braucht es einen Takt.

☐ arbeitstäglich früh ☐ zweimal täglich ☐ wöchentlich ☐ anderes: ______

---

### D4 ⚪ Was passiert mit den Altjahren?

**Warum wir fragen.** Die Liste 2026 hat rund 900 Zeilen mit Verweisen. Eine Migration ist aufwendig und fehleranfällig.

☐ Altjahre einfrieren, unverändert am Laufwerk belassen, ab Stichtag neu starten
☐ Laufendes Jahr migrieren, Vorjahre einfrieren
☐ Alles migrieren

Stichtag für den Umstieg: ______________

---

### D5 ⚪ Bestätigung der heutigen Listenstruktur

**Warum wir fragen.** Aus den Screenshots gelesen: **Spalte J heißt „ABD"** und enthält die MRN im Klartext, **Spalte K heißt „Austritt"** und enthält den Verweis auf das AGV-PDF. Zeilen mit gefülltem J und leerem K sind der Zustand „Austrittsnachweis fehlt".

☐ Richtig gelesen
☐ Nicht richtig — tatsächlich: _______________________________________

Und: wird die MRN aus dem **ABD** in Spalte J eingetragen, oder erst beim Eintreffen des AGV?
☐ beim ABD ☐ erst beim AGV ☐ unterschiedlich gehandhabt

Was gilt, wenn ein AGV eintrifft, **ohne** dass zuvor ein ABD verarbeitet wurde?
☐ kommt nicht vor ☐ kommt vor, Vorgehen: ______________________

---

# Block E — Was ist in der ersten Ausbaustufe dabei?

### E1 ⛔ Umfang Phase 1

**Warum wir fragen.** Drei Teilbereiche sind aufwendig und teilweise in der Klick-Doku überhaupt nicht abgebildet. Sie später zu ergänzen ist billiger, als die erste Stufe damit zu belasten.

| Teilbereich | Aufwand | in Phase 1? |
|---|---|---|
| AGV- und ABD-Prozess, Drittland | Basis | ☐ ja (empfohlen) |
| Segment Schweiz | gering | ☐ ja (empfohlen) ☐ nein |
| Monitoring und Follow-up 6-9-12 | mittel | ☐ ja (empfohlen) ☐ nein |
| Klärfallbehandlung | mittel | ☐ ja (empfohlen) ☐ nein |
| **DHL-Mustersendungen** | **hoch** | ☐ ja ☐ nein (empfohlen: Phase 2) |
| **Direktanlieferungen Schweiz (ZIV2)** | **mittel bis hoch** | ☐ ja ☐ nein (empfohlen: Phase 2) |
| Alternativnachweise, vollständige Ablagekette | gering bis mittel | ☐ ja ☐ nein |

**Zu DHL:** eigenes Dokumentformat ohne strukturierte Daten, die Seitenzahl im Sammel-PDF ist fachliche Referenz, mehrstufige Belegflussnavigation in SAP, mehrere Zweige mit Rückfrage beim Customer Service. In Phase 1 würde lediglich die Sammelbescheinigung abgelegt und ein manueller Vermerk gesetzt.

---

# Block F — Sonderfälle und Randbedingungen

### F1 ⚪ Kommen andere Prüfergebnisse als A1 und A2 vor?

Die Arbeitsanweisung verlangt, dass auf dem Dokument A1 oder A2 ausgewiesen ist. Was passiert bei einem anderen Code?

☐ Andere Codes kommen nicht vor
☐ Kommen vor: ______________________ · Behandlung: ______________________
☐ unbekannt — bei Englmayer erfragen

---

### F2 ⚪ Ist Englmayer der einzige Zolldienstleister?

Aus den Unterlagen sind drei Quellen für Nachweise bekannt: Englmayer, DHL und die Kunden selbst (Alternativnachweise). Die Arbeitsanweisung hält fest: „Englmayer hat immer Vorrang gegenüber DHL."

☐ Englmayer ist der einzige anmeldende Dienstleister
☐ Weitere: ______________________
☐ Für Ausfuhren aus Deutschland gilt: ______________________

Vorrangregel Englmayer vor DHL: ☐ bestätigt ☐ anders

---

### F3 ⚪ Österreich oder Deutschland — welches Zollsystem?

**Warum wir fragen.** Im ausgewerteten Dokument beginnt die MRN mit `26AT…` (Anmeldung Österreich), gleichzeitig ist als Ausfuhrland `DE` eingetragen und die Zollstellen sind deutsch. Die Arbeitsanweisung erwähnt „NovaTaste Austria/**Germany**". Für eine künftige elektronische Datenlieferung ist entscheidend, aus welchem System die Daten stammen.

☐ Alle Anmeldungen laufen über Österreich
☐ Es gibt beides — Anteil Deutschland etwa: ______ %
☐ unklar, bitte mit Englmayer klären

---

### F4 ⚪ Was passiert bei Storno oder Änderung einer Rechnung?

Im heutigen Prozess nicht behandelt. Wird eine Faktura nachträglich storniert, hätte die Automatisierung eine Zeile, die niemals einen Nachweis bekommt.

☐ Kommt praktisch nicht vor
☐ Kommt vor — Vorgang soll dann: ☐ aus der Überwachung fallen ☐ als „storniert" gekennzeichnet bleiben

---

### F5 ⚪ Die Outlook-Kategorie „Ungeprüft"

In der Klick-Doku erscheint bei der Mailauswahl der Vermerk „Ungeprüft". Wir vermuten eine Kategorie, die das Team als Arbeitsstatus nutzt.

☐ Ist eine Kategorie und Teil des Arbeitsablaufs — Werte: ______________________
☐ Ist keine Kategorie / nicht prozessrelevant
☐ Kategorien werden künftig durch das Monitoring ersetzt

---

### F6 ⚪ Was bedeutet der Suchbegriff `EDTBX`?

In der Klick-Doku wird in Excel nach `EDTBX` gesucht (Schritt 73). In keiner Unterlage ist erklärt, was das ist.

☐ Kunden- oder Empfängerkürzel: ______________________
☐ Etwas anderes: ______________________
☐ nicht mehr relevant

---

### F7 ⚪ Technischer Benutzer als Ersteller der SAP-Anlage

**Warum wir fragen.** In der SAP-Anlagenliste steht heute der Name der Bearbeiterin, die den Nachweis angehängt hat. Mit Automatisierung steht dort ein technischer Benutzer. Die Zuordnung „wer hat geprüft" wandert damit in das Prüfprotokoll des neuen Systems.

☐ Akzeptiert
☐ Nicht akzeptiert — der Name der freigebenden Person muss in SAP sichtbar bleiben
☐ Rücksprache mit Steuer/Revision erforderlich

---

# Anhang: Punkte, die nicht der Fachbereich beantwortet

Zur Information — diese Fragen laufen parallel an andere Stellen:

| An das SAP-Team | An Englmayer |
|---|---|
| Woher kommt die Packstückzahl technisch? | Schema der künftigen XML-Lieferung, Versionierung |
| Welche Partnerrolle ist der Warenempfänger? | Werden XML und PDF gemeinsam geliefert? |
| Vergleichsbasis für Versender/Exporteur | Ist das XML das Original des Zollsystems oder eine Ableitung? |
| SAP-Release und Möglichkeit eines Lesedienstes | Gibt es ein Betreffmuster für ABD-Mails? |
| Schnittstelle für Anlagen, Titel setzbar? | Ist „eine Mail = eine Anmeldung" garantiert? |
| Ist GUI-Scripting bzw. webGUI verfügbar? | Andere Prüfergebnisse als A1/A2? |
| Mapping der IV2-Nummern auf Fakturabelege | |
| Entspricht die Kundenbetreuung dem Sachbearbeiter am Fakturakopf? | |
| Speichertechnologie und Berechtigungen | |

---

# Abschluss

**Wie es weitergeht.** Sobald Block A, B und E beantwortet sind, kann mit dem Aufbau begonnen werden — zunächst als lauffähiges Gerüst mit Testdaten, ohne Zugriff auf SAP oder das Postfach. Die Antworten aus den Blöcken C, D und F fließen anschließend als Einstellungen ein, nicht als Programmänderungen.

**Was ausdrücklich nicht Gegenstand ist:** Eingriffe in die Fakturierung, Änderungen an der Zollanmeldung, automatische Beurteilung von Alternativnachweisen, Ersatz fachlicher Entscheidungen in Zweifelsfällen.

---

| | |
|---|---|
| Beantwortet am | ______________________ |
| Teilnehmende | ______________________ |
| Freigegeben von | ______________________ |
| Offen geblieben | ______________________ |

*Rückfragen bitte unter Angabe der Punktnummer (z. B. „A3") stellen.*
