# reimagined-computing-machine
Consistency Audit
# Consistency Audit

**Version 0.3.0 · Read-only static analysis for software architecture, consistency and reliability · Windows / Python**

[English](#english) · [Deutsch](#deutsch)

---

<a id="english"></a>

# English

## Overview

**Consistency Audit** is a local, read-only source-code inspection tool. It helps developers identify potentially problematic patterns involving software architecture, input/output boundaries, transactions, event processing, concurrency, error handling and security. The scanner produces structured findings, review recommendations and **ready-to-copy remediation prompts** for subsequent manual work.

Its purpose is to help uncover possible design inconsistencies **without running the project being inspected**.

**Language note:** This README is bilingual; the current launcher messages and generated reports are in German.

**Typical workflow:** Extract the release → launch `SCAN_STARTEN.bat` → select a project folder → review the reports → verify findings → approve any corrections separately.

> **Important:** The scanner does not run unit tests, build scripts or the inspected application. It does not install dependencies, modify the source code or automatically fix findings. A scan without findings does **not** establish that the project is correct or secure.

### Features at a glance

| Feature | Current implementation (v0.3.0) |
| --- | --- |
| Launch | Double-click `SCAN_STARTEN.bat` on Windows, or use CMD/PowerShell |
| Runtime | Python 3.9 or newer; no third-party Python packages required |
| Operation | Local, static analysis; reads supported project files |
| Language coverage | Text-based rules for multiple formats; deeper AST checks mainly for Python; limited local dependency analysis for Python and JavaScript/TypeScript |
| Rule set | 34 implemented rule identifiers |
| Reference coverage | 26 topics from the consistency reference material, with explicit coverage limitations |
| Optional profiles | `auto`, `seven-voices-stage1`, `seven-voices-stage2`, `seven-voices-midi` |
| Outputs | `report.html`, `report.md`, `report.json`, `behebungsauftraege.md` |
| Project changes | None during normal use with the default external report location |
| Remote services | No network calls, remote scanners, API keys or cloud uploads |

## 1. Requirements

- **Windows 10 or Windows 11** for the provided batch launcher. The Python scanner can also be used on other operating systems.
- **Python 3.9+**, accessible through `py -3` or `python`.
- Read access to the project being inspected and write access to the selected report destination.
- A web browser to view the HTML report.

Check the Python version in Command Prompt:

```bat
py -3 --version
```

If the Python launcher is unavailable, try:

```bat
python --version
```

Download Python from [python.org](https://www.python.org/downloads/) if necessary. The tool itself does **not** require `pip install` or any additional Python library.

## 2. Quick start on Windows

1. Download and **fully extract** the release archive.
2. Open the extracted folder containing `SCAN_STARTEN.bat`, `audit.py` and `deep_rules.py`.
3. Double-click `SCAN_STARTEN.bat`.
4. Enter the full path of the project you want to inspect, then press **Enter**.
5. Wait for the scan summary and report locations to appear.
6. The launcher attempts to open `report.html` in your default browser. If this does not work, open the file using the path printed in the terminal.

**Alternative:** Drag a project folder onto `SCAN_STARTEN.bat` to supply its path directly.

The batch file is only the launcher. The inspection logic runs in `audit.py` and `deep_rules.py`.

See [Windows quick-start guide](docs/01_SCHNELLSTART.md) for additional instructions (supporting document currently in German).

## 3. Command-line usage

Open CMD or PowerShell in the extracted tool folder, which contains `audit.py`. Replace the uppercase placeholders below with your own paths; **no internal project names are required**.

**Scan a project:**

```bat
py -3 audit.py scan "PATH_TO_PROJECT"
```

**Scan and open the HTML report:**

```bat
py -3 audit.py scan "PATH_TO_PROJECT" --open-report
```

**Choose a separate output location:**

```bat
py -3 audit.py scan "PATH_TO_PROJECT" --output "PATH_TO_REPORTS"
```

**Select an optional analysis profile:**

```bat
py -3 audit.py scan "PATH_TO_PROJECT" --profile auto
```

**Show implemented command-line options:**

```bat
py -3 audit.py --help
```

If `py -3` is not available, use `python` instead. You can also call `audit.py` by its full path from a different working directory; the `scan` argument then accepts `.` to select that working directory.

**Implemented command:** `scan`. Commands such as `fix`, `test`, `watch` and a globally installed `consistency-audit` executable are **not** part of version 0.3.0.

See [Command-line options and profiles](docs/02_BEDIENUNG_UND_PROFILE.md) (German).

## 4. Generated reports

By default, each scan creates a **new report folder outside the inspected project**, under the user profile. On Windows the base location is normally:

```text
%LOCALAPPDATA%\ConsistencyAudit\reports\
```

The scanner prints the exact report path after completing a scan.

| Output file | Purpose |
| --- | --- |
| `report.html` | Browser-friendly summary, findings, recommendations and open review questions |
| `report.md` | Full Markdown report, including the reference-topic coverage matrix |
| `report.json` | Machine-readable results for further processing |
| `behebungsauftraege.md` | Copyable prompts for reviewing, validating and addressing individual findings |

**The prompts are not executable commands.** They instruct a developer or a separate coding assistant to review a suspected issue, establish context and suggest an appropriate correction. Changes require separate approval.

**Privacy note:** Reports may include absolute project paths, source-file locations, symbols and descriptions of potential vulnerabilities. Review them before publishing or sharing them.

See [Understanding the reports](docs/04_BERICHTE_UND_BEHEBUNG.md) (German).

## 5. Inspection scope

The tool combines:

1. **Text-pattern rules** for potentially risky constructs.
2. **Python AST inspection** for selected syntax, calls, exception handling and simple local data flows. Source code is parsed, not executed.
3. **Limited local import-graph analysis** for Python and certain relative JavaScript/TypeScript imports, including detection of import cycles.
4. **Project inventory** and a **26-topic reference coverage matrix**, which makes unverified topics visible rather than silently marking them as passed.

### The 34 implemented rules

| Area | Rule identifiers | Examples of indicators |
| --- | --- | --- |
| Syntax and project inventory | `PY-001`, `CFG-001`, `DOC-001`, `TST-001` | Python parse errors, invalid `package.json`, missing recognized README or test files |
| Security | `SEC-001`–`SEC-006` | Potential embedded secrets, disabled TLS checks, dynamic execution, sensitive logs, risky deserialization |
| Input/output and trust boundaries | `IO-001`–`IO-006`, `IO-101`, `IO-102` | Mixing I/O with business logic, HTML sinks, possibly unvalidated input reaching SQL, files or process execution |
| Events and consistency | `EVT-001`–`EVT-003`, `EVT-101`, `CON-002` | Possible dual writes, outbox review, acknowledgements, time-based conflict resolution |
| Concurrency and background work | `CON-001`, `ASYNC-101`–`ASYNC-103` | Potentially concurrent calls, tasks without an evident lifecycle or completion check |
| Error handling and reliability | `ERR-001`, `ERR-101`, `NET-001`, `NET-002`, `NET-101`, `REL-001` | Silenced exceptions, missing visible timeouts, retry/backoff review points |
| Architecture | `ARCH-101` | Detected cycles in the statically resolved local import graph |

> A reported pattern is a **review indicator**, not necessarily a confirmed defect. For example, a background task may be intentional, a timeout may be configured centrally, and an outbox implementation may use a different name.

**Supported language depth:** Python receives the most extensive analysis. JavaScript and TypeScript receive text-pattern checks and limited relative-import analysis, **not** full syntax or type checking. Other supported text-based source/configuration formats receive only applicable text/inventory checks; binary files are not analyzed.

A full rule-by-rule description is available in the [rule catalogue](docs/03_PRUEFUMFANG.md) (German).

### The 26 reference topics

The coverage matrix includes local ACID transactions, transaction boundaries, serializability, linearizability, CAP/PACELC, replication and quorums, consensus, eventual consistency, session guarantees, Event Sourcing/CQRS, Outbox/Inbox, idempotency, Sagas, causal ordering, CRDTs, domain invariants, consistency budgets, retries, dead-letter queues, reconciliation, observability, formal models, input/output separation, validation, trust zones, privacy, architecture documentation and test coverage.

**The matrix is not a certification or a set of 26 completed tests.** Each topic is explicitly described as supported by limited indicators or **not verified** where a static scan cannot establish the guarantee.

## 6. Optional execution profiles

The default `auto` profile makes no assumption about how agents or tasks are scheduled. Three additional built-in profiles describe specialized orchestration patterns:

| Profile | Intended execution model |
| --- | --- |
| `auto` | General-purpose scan; no fixed agent execution assumptions |
| `seven-voices-stage1` | Manually authorized, sequential task execution |
| `seven-voices-stage2` | Sequential processing through specialized result layers |
| `seven-voices-midi` | MIDI/MCP-based work, where separate connections do not necessarily imply resource isolation |

**Current limitation:** These profiles primarily supply contextual assumptions and review questions. Version 0.3.0 does **not** formally prove scheduling guarantees or enable a fully separate rule engine for each profile.

## 7. Interpreting findings

Every finding identifies the rule, file and line number, category, severity, indicator confidence, evidence type and proposed next step.

- **Severity** (`HIGH`, `MEDIUM`, `INFO`) prioritizes review; it is not a verified measurement of impact.
- **Confidence** (`high`, `medium`, `low`) expresses how clearly a rule recognized a pattern; it is not a statistical probability.
- **Evidence type** distinguishes parse-confirmed structures from static indicators, function-level hypotheses, limited data-flow hypotheses and dependency-graph observations.
- **Not verified** in the reference matrix means the relevant property was **not established**, not that the project passed.

Recommended review procedure: **Locate → understand context → confirm or dismiss → assess impact → propose a minimal correction → obtain approval → validate separately.**

This release does not execute existing test suites or measure runtime test coverage.

## 8. Safety, privacy and analysis limits

- Scans project files locally; it does not run the inspected code or make network requests.
- Does not upload source code, call language models or invoke third-party services.
- Does not automatically edit, format, migrate or delete project files.
- Report files are written outside the project **by default**. Setting `--output` to a directory **inside** the source tree will create report files there.
- Source-code lines are not copied verbatim into reports, but paths, function/module names and findings may still be sensitive. The scanner may also read `.env` files; keep project files and reports confidential.
- Symbolic links are not followed; junction handling depends on the Python runtime.
- Common dependency/build folders, including `.git`, `node_modules`, `.venv`, `build` and `dist`, are skipped.
- Supported text files must be UTF-8 decodable (with or without a BOM).

**Per-scan limits:**

| Limit | Value |
| --- | --- |
| Maximum files read | 8,000 |
| Maximum individual file size | 768 KiB |
| Maximum total content read | 128 MiB |

Some files can be skipped because of format, size, permissions or excluded directories. Review the skipped-file statistics before interpreting the result as comprehensive.

**No remote integrations are included in this version:** CodeQL, Semgrep, Trivy, Shodan and Notion are not connected.

See [Limits and privacy](docs/05_GRENZEN_DATENSCHUTZ.md) (German).

## 9. Troubleshooting

| Problem | What to check |
| --- | --- |
| Python command is not found | Install Python 3.9+ and check `py -3 --version` or `python --version` |
| The batch file closes or fails | Start it from CMD to see the error; verify the three program files are together |
| Project folder not found | Provide the full, existing project directory path; drag-and-drop is also supported |
| No findings reported | Check read-file and skip counts; no findings does not guarantee correctness |
| HTML report does not open | Open `report.html` manually using the reported path |
| Output cannot be written | Choose a writable report destination outside the scanned project |
| A finding looks incorrect | Examine the source location, call context and configuration before making changes |

See [Troubleshooting](docs/06_FEHLERBEHEBUNG.md) (German).

## 10. Package contents

The release includes these files and supporting guides:

```text
SCAN_STARTEN.bat
START_HIER.txt
audit.py
deep_rules.py
README.md
CHANGELOG.md
docs/
```

The README should be placed **next to the program files** in your GitHub repository so the relative documentation links remain valid. Do not copy internal source-project paths into the public README.

### Additional documentation

The following companion documents are currently provided in German:

- [Quick start](docs/01_SCHNELLSTART.md)
- [CLI usage and profiles](docs/02_BEDIENUNG_UND_PROFILE.md)
- [Rule catalogue and coverage](docs/03_PRUEFUMFANG.md)
- [Reports and remediation workflow](docs/04_BERICHTE_UND_BEHEBUNG.md)
- [Limits and privacy](docs/05_GRENZEN_DATENSCHUTZ.md)
- [Troubleshooting](docs/06_FEHLERBEHEBUNG.md)
- [Technical architecture](docs/07_TECHNISCHE_ARCHITEKTUR.md)
- [Roadmap and release checklist](docs/08_WEITERENTWICKLUNG_UND_FREIGABE.md)
- [Changelog](CHANGELOG.md)

## 11. Project status and licensing

**Status:** v0.3.0 is a working static-analysis prototype. Basic checks were exercised on an artificial fixture project. This is **not** a complete Windows acceptance test, a formal verification or an assurance that all findings and risks will be detected.

**License:** No license has been chosen or included in this release. Publishing code on GitHub does not, by itself, grant others permission to reuse or redistribute it. The repository owner should add an appropriate `LICENSE` file and define contribution terms before inviting external use or contributions.

---

<a id="deutsch"></a>

# Deutsch

## Überblick

**Consistency Audit** ist ein lokal arbeitendes Werkzeug zur **statischen Quellcodeanalyse im Nur-Lese-Modus**. Es hilft dabei, mögliche Probleme bei Softwarearchitektur, Ein-/Ausgabegrenzen, Transaktionen, Ereignisverarbeitung, Nebenläufigkeit, Fehlerbehandlung und Sicherheit aufzudecken. Anschließend erstellt es strukturierte Befunde, Empfehlungen und **direkt kopierbare Behebungsaufträge** zur späteren manuellen Bearbeitung.

Das Ziel ist, mögliche Struktur- und Konsistenzprobleme zu erkennen, **ohne das untersuchte Programm auszuführen**.

**Sprachhinweis:** Diese README ist zweisprachig; die aktuellen Meldungen des Startprogramms und die erzeugten Berichte sind weiterhin auf Deutsch.

**Typischer Ablauf:** Archiv entpacken → `SCAN_STARTEN.bat` starten → Projektordner auswählen → Berichte lesen → Befunde überprüfen → Korrekturen gesondert freigeben.

> **Wichtig:** Das Werkzeug führt keine Unit-Tests, Builds oder Programme des untersuchten Projekts aus. Es installiert keine Abhängigkeiten, verändert keinen Quellcode und behebt keine Fehler automatisch. Ein Scan ohne Befunde ist **kein Nachweis für Fehlerfreiheit oder Sicherheit**.

### Funktionen im Überblick

| Merkmal | Implementierter Stand (v0.3.0) |
| --- | --- |
| Start | Doppelklick auf `SCAN_STARTEN.bat` unter Windows oder über CMD/PowerShell |
| Laufzeit | Python 3.9 oder neuer; keine zusätzlichen Python-Pakete erforderlich |
| Arbeitsweise | Lokale, statische Analyse unterstützter Projektdateien im Nur-Lese-Modus |
| Sprachen | Textbasierte Prüfungen für verschiedene Formate; tiefere AST-Prüfungen vor allem für Python; begrenzte lokale Abhängigkeitsanalyse für Python und JavaScript/TypeScript |
| Regelumfang | 34 implementierte Regelkennungen |
| Referenzwerk | 26 Themen mit ausdrücklich gekennzeichneten Abdeckungsgrenzen |
| Optionale Profile | `auto`, `seven-voices-stage1`, `seven-voices-stage2`, `seven-voices-midi` |
| Ergebnisse | `report.html`, `report.md`, `report.json`, `behebungsauftraege.md` |
| Projektänderungen | Keine bei normaler Verwendung mit standardmäßig externer Berichtsablage |
| Externe Dienste | Keine Netzwerkabfragen, Remote-Scanner, API-Schlüssel oder Cloud-Uploads |

## 1. Voraussetzungen

- **Windows 10 oder Windows 11** für die mitgelieferte Batch-Datei. Der Python-Kern lässt sich grundsätzlich auch unter anderen Betriebssystemen verwenden.
- **Python 3.9 oder neuer**, aufrufbar über `py -3` oder `python`.
- Leserechte für das untersuchte Projekt sowie Schreibrechte für die Berichtsablage.
- Ein Webbrowser zum Öffnen des HTML-Berichts.

Python-Version in der Eingabeaufforderung prüfen:

```bat
py -3 --version
```

Falls der Python-Launcher nicht verfügbar ist:

```bat
python --version
```

Python kann bei Bedarf über [python.org](https://www.python.org/downloads/) installiert werden. Das Programm benötigt **keine `pip`-Installation** und keine zusätzlichen Python-Bibliotheken.

## 2. Schnellstart unter Windows

1. Das Programmarchiv herunterladen und **vollständig entpacken**.
2. Den entpackten Ordner mit `SCAN_STARTEN.bat`, `audit.py` und `deep_rules.py` öffnen.
3. `SCAN_STARTEN.bat` doppelklicken.
4. Den vollständigen Pfad zum gewünschten Projektordner eingeben und **Enter** drücken.
5. Auf die Abschlussmeldung und die angezeigten Berichtsdateien warten.
6. Das Programm versucht, `report.html` im Standardbrowser zu öffnen. Falls das nicht gelingt, die im Terminal angezeigte Datei manuell öffnen.

**Alternative:** Einen Projektordner per Drag-and-drop auf `SCAN_STARTEN.bat` ziehen. Der Ordnerpfad wird so direkt übergeben.

Die Batch-Datei dient nur als Starter. Die eigentliche Prüflogik befindet sich in `audit.py` und `deep_rules.py`.

Weitere Hinweise: [Schnellstartanleitung](docs/01_SCHNELLSTART.md).

## 3. Nutzung über CMD oder PowerShell

Die Eingabeaufforderung oder PowerShell im entpackten Werkzeugordner mit `audit.py` öffnen. Die großgeschriebenen Platzhalter durch die eigenen Pfade ersetzen. **Interne Projekt- oder Ordnernamen werden in den Beispielen nicht verwendet.**

**Projekt untersuchen:**

```bat
py -3 audit.py scan "PFAD_ZUM_PROJEKT"
```

**Projekt untersuchen und HTML-Bericht öffnen:**

```bat
py -3 audit.py scan "PFAD_ZUM_PROJEKT" --open-report
```

**Speicherort der Berichte selbst festlegen:**

```bat
py -3 audit.py scan "PFAD_ZUM_PROJEKT" --output "PFAD_ZU_BERICHTEN"
```

**Optionales Prüfprofil wählen:**

```bat
py -3 audit.py scan "PFAD_ZUM_PROJEKT" --profile auto
```

**Alle verfügbaren Parameter anzeigen:**

```bat
py -3 audit.py --help
```

Wenn `py -3` nicht verfügbar ist, stattdessen `python` verwenden. `audit.py` kann auch über seinen vollständigen Pfad von einem anderen Arbeitsverzeichnis aus gestartet werden; dann bezeichnet `scan .` das aktuelle Arbeitsverzeichnis.

**Implementierter Befehl:** `scan`. Die Unterbefehle `fix`, `test` oder `watch` sowie ein global installierbarer Befehl `consistency-audit` gehören **nicht** zu Version 0.3.0.

Weitere Informationen: [Bedienung und Profile](docs/02_BEDIENUNG_UND_PROFILE.md).

## 4. Erstellte Berichte

Standardmäßig erzeugt jeder Scan einen **neuen Berichtsordner außerhalb des untersuchten Projekts** im Benutzerprofil. Unter Windows lautet der übliche Basisordner:

```text
%LOCALAPPDATA%\ConsistencyAudit\reports\
```

Den genauen Speicherort zeigt das Programm nach dem Scan im Terminal an.

| Datei | Inhalt |
| --- | --- |
| `report.html` | Übersicht im Browser mit Befunden, Empfehlungen und offenen Prüffragen |
| `report.md` | Vollständiger Markdown-Bericht einschließlich Abdeckungsmatrix des Referenzwerks |
| `report.json` | Maschinenlesbare Ergebnisse für weitere Auswertungen |
| `behebungsauftraege.md` | Einzeln kopierbare Aufträge zur Prüfung und Behebung gefundener Auffälligkeiten |

**Die Behebungsaufträge sind keine ausführbaren Terminalbefehle.** Sie dienen als Arbeitsanweisungen für Entwickler oder einen gesonderten KI-Coding-Assistenten. Zuerst muss der Befund im Kontext überprüft werden. Änderungen benötigen eine gesonderte Freigabe.

**Datenschutzhinweis:** Berichte können absolute lokale Projektpfade, Dateinamen, Programmbezeichner und Beschreibungen möglicher Schwachstellen enthalten. Vor einer Veröffentlichung oder Weitergabe müssen sie deshalb geprüft werden.

Siehe [Berichte verstehen und Befunde bearbeiten](docs/04_BERICHTE_UND_BEHEBUNG.md).

## 5. Umfang der Untersuchung

Das Programm kombiniert:

1. **Textmuster-Prüfungen** für möglicherweise problematische Konstruktionen.
2. **Python-AST-Analysen** für ausgewählte Syntaxelemente, Funktionsaufrufe, Ausnahmebehandlung und einfache lokale Datenflüsse. Der Quellcode wird geparst, nicht ausgeführt.
3. **Begrenzte lokale Importgraph-Analysen** für Python und bestimmte relative JavaScript-/TypeScript-Imports einschließlich Erkennung von Importzyklen.
4. **Projektinventur** und eine **Abdeckungsmatrix mit 26 Referenzthemen**, die nicht überprüfte Bereiche offen ausweist.

### Die 34 implementierten Prüfregeln

| Bereich | Regelkennungen | Beispiele für erkennbare Hinweise |
| --- | --- | --- |
| Syntax und Projektinventur | `PY-001`, `CFG-001`, `DOC-001`, `TST-001` | Python-Syntaxfehler, fehlerhafte `package.json`, keine erkannte README oder Testdateien |
| Sicherheit | `SEC-001`–`SEC-006` | Mögliche Geheimnisse im Code, deaktivierte TLS-Prüfung, dynamische Ausführung, sensible Logs, riskante Deserialisierung |
| Ein-/Ausgabe und Vertrauensgrenzen | `IO-001`–`IO-006`, `IO-101`, `IO-102` | Vermischung von I/O und Fachlogik, HTML-Ausgabesenken, möglicherweise ungeprüfte Eingaben in SQL, Dateien oder Prozessaufrufen |
| Events und Konsistenz | `EVT-001`–`EVT-003`, `EVT-101`, `CON-002` | Mögliche Dual Writes, Outbox-Prüfung, Bestätigungen, zeitbasierte Konfliktauflösung |
| Nebenläufigkeit und Hintergrundaufgaben | `CON-001`, `ASYNC-101`–`ASYNC-103` | Nebenläufigkeitsindikatoren, Aufgaben ohne erkennbaren Lebenszyklus oder Abschlusskontrolle |
| Fehlerbehandlung und Zuverlässigkeit | `ERR-001`, `ERR-101`, `NET-001`, `NET-002`, `NET-101`, `REL-001` | Unterdrückte Ausnahmen, nicht erkennbare Timeouts, Fragen zu Wiederholung und Backoff |
| Architektur | `ARCH-101` | Zyklen in lokal statisch aufgelösten Importabhängigkeiten |

> Ein gemeldetes Muster ist ein **Prüfhinweis**, nicht automatisch ein bestätigter Fehler. Beispielsweise kann eine Hintergrundaufgabe beabsichtigt sein, ein Timeout zentral gesetzt werden oder eine Outbox einen anderen Namen tragen.

**Tiefe der Sprachunterstützung:** Python wird am umfassendsten analysiert. Für JavaScript/TypeScript stehen Textmuster und begrenzte Prüfungen relativer Imports zur Verfügung, **keine vollständige Syntax- oder Typprüfung**. Andere unterstützte textbasierte Quell- und Konfigurationsformate werden nur mit den jeweils passenden Text- und Inventurregeln untersucht. Binärdateien werden nicht analysiert.

Alle einzelnen Regeln sind im [Regelkatalog](docs/03_PRUEFUMFANG.md) beschrieben.

### Die 26 Referenzthemen

Die Abdeckungsmatrix berücksichtigt unter anderem lokale ACID-Transaktionen, Transaktionsgrenzen, Serialisierbarkeit, Linearizierbarkeit, CAP/PACELC, Replikation und Quoren, Konsens, Eventual Consistency, Sitzungsgarantien, Event Sourcing/CQRS, Outbox/Inbox, Idempotenz, Sagas, kausale Reihenfolgen, CRDTs, fachliche Invarianten, Konsistenzbudgets, Wiederholungen, Dead-Letter-Queues, Reconciliation, Observability, formale Modelle, Trennung von Eingabe und Ausgabe, Validierung, Vertrauenszonen, Datenschutz, Architekturdokumentation und Testabdeckung.

**Die Matrix ist weder ein Zertifikat noch eine Sammlung von 26 erfolgreich abgeschlossenen Tests.** Jedes Thema wird als teilweise durch Indikatoren gestützt oder als **nicht verifiziert** ausgewiesen, sofern ein statischer Scan die jeweilige Garantie nicht belegen kann.

## 6. Optionale Ausführungsprofile

Das Standardprofil `auto` unterstellt kein bestimmtes Ausführungsmodell für Agenten oder Aufgaben. Drei weitere eingebaute Profile beschreiben spezielle Orchestrierungsmuster:

| Profil | Beschriebenes Ausführungsmodell |
| --- | --- |
| `auto` | Allgemeine Prüfung ohne feste Annahmen zur Agentenausführung |
| `seven-voices-stage1` | Manuell freigegebene, sequenzielle Aufgabenbearbeitung |
| `seven-voices-stage2` | Sequenzielle Bearbeitung in spezialisierten Ergebnisschichten |
| `seven-voices-midi` | MIDI-/MCP-Arbeit, bei der getrennte Verbindungen keine getrennten Ressourcen garantieren |

**Derzeitige Einschränkung:** Die Profile ergänzen hauptsächlich den architektonischen Kontext und die offenen Prüffragen. Version 0.3.0 beweist damit **nicht** formal, dass die jeweilige Ausführungsreihenfolge eingehalten wird. Auch gibt es noch keinen vollständig eigenständigen Regelsatz pro Profil.

## 7. Befunde richtig einordnen

Jeder Befund nennt Regel, Datei und Zeilennummer, Kategorie, Schweregrad, Indikatorsicherheit, Nachweistyp und einen Vorschlag für das weitere Vorgehen.

- **Schweregrad** (`HIGH`, `MEDIUM`, `INFO`) dient zur Priorisierung und ist keine bestätigte Schadensbewertung.
- **Indikatorsicherheit** (`high`, `medium`, `low`) beschreibt die Eindeutigkeit des erkannten Musters, nicht eine statistische Wahrscheinlichkeit.
- **Nachweistyp** unterscheidet bestätigte Parserstrukturen, statische Muster, funktionsbezogene Hypothesen, begrenzte Datenfluss-Hypothesen und Beobachtungen aus dem Abhängigkeitsgraphen.
- **Nicht verifiziert** bedeutet in der Referenzmatrix, dass die Eigenschaft **nicht nachgewiesen** wurde – nicht, dass das Projekt bestanden hat.

Empfohlener Ablauf: **Fundstelle finden → Kontext verstehen → bestätigen oder verwerfen → Risiko bewerten → minimale Korrektur vorschlagen → freigeben → separat überprüfen.**

Diese Version führt weder vorhandene Testprogramme aus noch ermittelt sie die tatsächliche Laufzeit-Testabdeckung.

## 8. Sicherheit, Datenschutz und Prüfgrenzen

- Projektdateien werden lokal gelesen; untersuchter Programmcode wird nicht ausgeführt und es erfolgen keine Netzwerkabfragen.
- Keine Übertragung von Quellcode, keine KI-Modellaufrufe und keine externen Prüfdienste.
- Keine automatischen Codeänderungen, Formatierungen, Migrationen oder Löschungen.
- Berichte werden **standardmäßig außerhalb des Projekts** abgelegt. Wird `--output` absichtlich auf ein Verzeichnis **innerhalb** des Projektbaums gesetzt, entstehen dort Berichtsdateien.
- Quelltextzeilen werden nicht wörtlich in die Berichte übernommen. Pfade, Funktions-/Modulnamen und Befundtexte können dennoch vertraulich sein. Der Prüfer kann auch `.env`-Dateien einlesen; Projektdateien und Berichte deshalb vertraulich behandeln.
- Symbolische Links werden nicht verfolgt; die Behandlung von Junctions hängt von der Python-Version ab.
- Übliche Abhängigkeits- und Buildordner wie `.git`, `node_modules`, `.venv`, `build` und `dist` werden ausgelassen.
- Unterstützte Textdateien müssen als UTF-8 (gegebenenfalls mit BOM) lesbar sein.

**Grenzen pro Scan:**

| Grenze | Wert |
| --- | --- |
| Maximal eingelesene Dateien | 8.000 |
| Maximale Größe je Datei | 768 KiB |
| Maximal eingelesene Datenmenge | 128 MiB |

Dateien können wegen Dateityp, Größe, fehlender Leserechte oder ausgeschlossener Verzeichnisse übersprungen werden. Die entsprechende Statistik ist vor einer umfassenden Bewertung zu berücksichtigen.

**Keine Remote-Integrationen in dieser Version:** CodeQL, Semgrep, Trivy, Shodan und Notion sind nicht angebunden.

Siehe [Grenzen und Datenschutz](docs/05_GRENZEN_DATENSCHUTZ.md).

## 9. Fehlerbehebung

| Problem | Prüfung beziehungsweise Lösung |
| --- | --- |
| Python-Befehl wird nicht gefunden | Python 3.9+ installieren und `py -3 --version` oder `python --version` prüfen |
| Batch-Datei schließt sich oder meldet Fehler | Über CMD starten, um die Meldung zu sehen; prüfen, ob die drei Programmdateien zusammenliegen |
| Projektordner wird nicht gefunden | Vollständigen, vorhandenen Verzeichnispfad angeben; alternativ Drag-and-drop nutzen |
| Keine Auffälligkeiten gefunden | Dateizahl und Übersprungstatistik prüfen; keine Befunde sind kein Sicherheitsnachweis |
| HTML-Bericht öffnet sich nicht | `report.html` über den angezeigten Pfad manuell öffnen |
| Berichte lassen sich nicht speichern | Schreibbaren Ausgabeordner außerhalb des Projektbaums wählen |
| Ein Fund erscheint unzutreffend | Fundstelle, Aufrufkontext und Konfiguration vor Änderungen sorgfältig prüfen |

Weitere Hilfen: [Fehlerbehebung](docs/06_FEHLERBEHEBUNG.md).

## 10. Paketinhalt

Das Programmpaket enthält folgende Dateien und ergänzende Anleitungen:

```text
SCAN_STARTEN.bat
START_HIER.txt
audit.py
deep_rules.py
README.md
CHANGELOG.md
docs/
```

Die README sollte im GitHub-Repository **neben den Programmdateien** liegen, damit die relativen Dokumentationslinks funktionieren. Interne Pfade aus untersuchten Projekten gehören nicht in die öffentliche README.

### Ergänzende Dokumentation

- [Schnellstart](docs/01_SCHNELLSTART.md)
- [Bedienung und Profile](docs/02_BEDIENUNG_UND_PROFILE.md)
- [Regelkatalog und Abdeckung](docs/03_PRUEFUMFANG.md)
- [Berichte und Behebungsaufträge](docs/04_BERICHTE_UND_BEHEBUNG.md)
- [Grenzen und Datenschutz](docs/05_GRENZEN_DATENSCHUTZ.md)
- [Fehlerbehebung](docs/06_FEHLERBEHEBUNG.md)
- [Technische Architektur](docs/07_TECHNISCHE_ARCHITEKTUR.md)
- [Weiterentwicklung und Freigabe-Checkliste](docs/08_WEITERENTWICKLUNG_UND_FREIGABE.md)
- [Änderungsverlauf](CHANGELOG.md)

## 11. Projektstatus und Lizenz

**Status:** Version 0.3.0 ist ein funktionsfähiger Prototyp für statische Analysen. Grundlegende Prüfungen wurden an einem künstlichen Beispielprojekt ausgeführt. Das ist **kein vollständiger Windows-Abnahmetest**, keine formale Verifikation und keine Garantie, dass sämtliche Fehler entdeckt werden.

**Lizenz:** Für diese Version wurde noch keine Softwarelizenz festgelegt oder beigelegt. Die Veröffentlichung auf GitHub allein erlaubt Dritten nicht automatisch, das Programm weiterzuverwenden oder weiterzuverbreiten. Der Repository-Verantwortliche sollte vor einer Freigabe zur Nutzung oder Mitarbeit eine passende `LICENSE`-Datei ergänzen und Mitwirkungsbedingungen festlegen.
