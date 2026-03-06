# KiDok – Projektplan und Checkliste

**Kinderdokumentation bei Hochkonflikt-Elternschaft**
Webapplikation zur strukturierten Dokumentation von Kindeswohl, Uebergaben, Medizin, Terminen und Vorfaellen

- **Technologie-Stack:** PHP 8.2 + Laravel 11 + MySQL + Alpine.js + Tailwind CSS
- **Basis:** Anforderungsdokument v2.2 FINAL (Stand Maerz 2026)
- **Gesamtlaufzeit:** 12 Monate (3 Phasen)

---

## Offene Punkte (vor Entwicklungsstart klaeren)

- [x] Hosting: **Netcup VPS** – Deutschland, DSGVO-konform ✓
- [x] Datenbankversion: **MySQL 8** ✓
- [x] PDF-Bibliothek: **DOMPDF** (rein PHP, kein Systembinary noetig) ✓
- [x] Monetarisierung: **Kostenlos, Spendenbasis** ✓
- [x] Integration hochkonflikteltern.de: **Eigenstaendige App** ✓
- [x] Warnschwellen: **Standardwerte beibehalten** (3/7 Tage gelb, 5/7 Tage rot, 50%-Muster orange) ✓
- [x] DSFA: **Selbst erstellen** mit BSI/BfDI-Vorlagen ✓

---

## Phase 1 – MVP (Monat 1–4)

### 1.1 Projektsetup und Infrastruktur

- [ ] Netcup VPS bereitstellen (Deutschland, DSGVO-konform)
- [ ] Nginx + PHP-FPM + MySQL konfigurieren
- [ ] TLS 1.3 Zertifikat einrichten (Let's Encrypt oder kommerziell)
- [ ] HSTS-Header setzen
- [ ] Laravel 11 Projekt anlegen (`composer create-project`)
- [ ] Git-Repository und Deployment-Workflow einrichten
- [ ] `.env`-Konfiguration fuer Produktion und Entwicklung
- [ ] Laravel Queues + Scheduler konfigurieren
- [ ] Automatisiertes verschluesseltes Backup einrichten (taeglich, Retention 90 Tage)
- [ ] Monitoring-Tool einrichten (Uptime, Fehler-Alerts)

### 1.2 Authentifizierung und Benutzerverwaltung (F-01 bis F-07)

- [ ] **F-01** Registrierung mit E-Mail und sicherem Passwort (min. 12 Zeichen, Sonderzeichen, Staerke-Indikator)
- [ ] **F-02** Zwei-Faktor-Authentifizierung (2FA) via `pragmarx/google2fa` (TOTP)
- [ ] **F-03** Passwort-Reset per E-Mail-Link (Token max. 1 Stunde gueltig)
- [ ] **F-04** Automatischer Session-Timeout (Standard 30 Min., konfigurierbar)
- [ ] **F-05** Einladungssystem fuer Co-Elternteil und Fachkraft (Link mit Ablaufdatum, Rollenauswahl)
- [ ] **F-06** Audit-Log aller Anmeldevorgaenge via `owen-it/laravel-auditing` (IP, Zeitstempel, User-Agent, nicht loeschbar)
- [ ] **F-07** Kontosperrung nach 5 Fehlversuchen + CAPTCHA
- [ ] Rollen-Middleware implementieren (Administrator, Hauptelternteil, Mitlesender Elternteil, Fachkraft)
- [ ] CSRF-Schutz aktivieren (Laravel Standard)
- [ ] Passwort-Hashing mit `PASSWORD_BCRYPT` oder `PASSWORD_ARGON2ID`

### 1.3 Datenbankstruktur anlegen

- [ ] Migration `users` (id, email, password_hash, role, 2fa_secret, consent_flags)
- [ ] Migration `children` (id, user_id FK, name, birthdate, school_type, class, allergies JSON, insurance JSON, deleted_at)
- [ ] Migration `checklist_items` (id, child_id FK, label, category, sort_order, active, hint_text, season_start, season_end)
- [ ] Migration `handover_logs` (id, child_id FK, user_id FK, type ENUM pickup/dropoff, location, handover_at gesperrt, mood, physical_state, notes, hash SHA-256)
- [ ] Migration `handover_checklist_results` (id, handover_log_id FK, checklist_item_id FK, checked BOOL, note, reason ENUM)
- [ ] Migration `handover_photos` (id, handover_log_id FK, file_path, file_hash, taken_at)
- [ ] Migration `handover_notifications` (id, handover_log_id FK, notified BOOL, channel ENUM, notified_at, note)
- [ ] Migration `medical_contacts`
- [ ] Migration `medical_visits`
- [ ] Migration `vaccinations`
- [ ] Migration `medications`
- [ ] Migration `appointments`
- [ ] Migration `incidents` + `incident_media`
- [ ] Migration `messages` (nicht loeschbar)
- [ ] Migration `audit_log` (nicht loeschbar)
- [ ] Migration `documents`
- [ ] Foreign Keys, Soft-Delete und Indizes definieren

### 1.4 Kinderprofil-Verwaltung (F-10 bis F-14)

- [ ] **F-10** Kinderprofil anlegen (Name, Geburtsdatum, Foto optional, Schulform, Klasse)
- [ ] **F-11** Mehrere Kinder pro Account (unbegrenzte Anzahl, Wechsel per Reiter/Sidebar)
- [ ] **F-12** Notfallkontakte pro Kind (bis zu 5 Kontakte mit Name, Telefon, Beziehung)
- [ ] **F-13** Allergien und Dauermedikation (Liste mit Freitext, Dosierung, Warnhinweis-Flag)
- [ ] **F-14** Versicherungsdaten (Krankenkasse, Versicherungsnummer, Zusatzversicherungen)

### 1.5 Pflegemaske fuer Checklisten-Eintraege

- [ ] Bereich unter Einstellungen > Checklisten verwalten
- [ ] Felder: Bezeichnung (Pflicht, max. 80 Zeichen), Kategorie (Pflicht, nur in Pflegemaske sichtbar), Kind(er)-Zuordnung
- [ ] Reihenfolge per Drag & Drop (Alpine.js / JavaScript)
- [ ] Aktiv / Inaktiv-Schaltung (inaktiv = in Schnellerfassung ausgeblendet; historische Protokolle unveraendert)
- [ ] Saisonalitaet (optionaler Datumsbereich; automatisch ausblenden ausserhalb Saison)
- [ ] Hinweistext (optional, erscheint als Tooltip beim Abhaken)

### 1.6 Uebergabe-Schnellerfassung – Abholung UND Abgabe (3.4a)

- [ ] Typ-Auswahl: Abholung (gruen) oder Abgabe (blau)
- [ ] Schritt 1: Typ waehlen (Abholung / Abgabe) mit eigenem Zeitstempel
- [ ] Schritt 2: Zeitstempel automatisch gesetzt; editierbar +-30 Min.; danach gesperrt
- [ ] Schritt 3: Ort (Freitext oder gespeicherter Ort)
- [ ] Schritt 4: Zustand des Kindes (Stimmung 1-5, koerperlicher Zustand, Freitext-Notiz, Foto optional/Pflicht bei "Verletzt")
  - [ ] Stimmungsskala 1–5 mit Bezeichnungen (Sehr gut / Gut / Neutral / Angespannt / Stark belastet)
  - [ ] Koerperlicher Zustand: Gesund / Erkaeltet / Verletzt / Behandlungsbeduerftig / Unbekannt
  - [ ] Bei "Verletzt": Freitext-Pflichtfeld + Foto-Hinweis
- [ ] Schritt 5: Checkliste abhaken (flache Checkbox-Liste aus Pflegemaske)
- [ ] Schritt 6: Fehlende automatisch rot markieren + Eintrag in Abweichungsprotokoll
- [ ] Schritt 7: Co-ET informiert? (Ja / Nein / Nicht relevant + Kanal + Zeitpunkt)
- [ ] Schritt 8: Foto-Dokumentation (bis zu 5 Fotos; GPS-Daten automatisch entfernen; SHA-256-Hash speichern)
- [ ] Schritt 9: Speichern mit Zeitstempel + SHA-256-Hash (AES-256 Verschluesselung at rest)
- [ ] Foto-Komprimierung auf max. 2 MB (PHP GD oder Imagick)
- [ ] Gesamtworkflow in unter 3 Minuten abschliessbar (Usability-Ziel)

### 1.7 Benachrichtigungs-Tracking (F-86 bis F-89)

- [ ] **F-86** Erfassungsmaske: Pflichtauswahl Ja/Nein/Nicht relevant; bei "Ja" weitere Felder
- [ ] **F-87** Kanal-Auswahl mit Mehrfachauswahl (Telefon / App / SMS / E-Mail / Persoenlich / Sonstiges)
- [ ] Zeitpunkt der Meldung (vorausgefuellt, editierbar)
- [ ] Kurze Notiz (optional, max. 300 Zeichen)
- [ ] **F-88** Antwort des Co-ET nachtragen (Ja / Nein / Noch ausstehend) als Folge-Protokolleintrag
- [ ] **F-89** Benachrichtigungs-Verlauf im Protokoll (alle Meldungen pro Kind chronologisch)

### 1.8 Warnsystem: Schutz vor Kommunikations-Eskalation (F-91 bis F-93)

- [ ] **F-91** WarnService: `checkFrequency()` – ab 3 Meldungen / 7 Tage → Stufe 1 (gelber Hinweistext)
- [ ] **F-93** WarnService: `checkFrequency()` – ab 5 Meldungen / 7 Tage → Stufe 3 (rote Warnung + optionales Begruendungsfeld max. 200 Zeichen, kein Pflichtfeld)
- [ ] **F-92** WarnService: `checkPattern()` – Eintrag fehlt in >50% der Uebergaben letzte 30 Tage → Stufe 2 (oranger Hinweis + Handlungsempfehlung)
- [ ] `getWarnLevel()` – kombiniert beide Checks; gibt hoechste Stufe zurueck
- [ ] `logWarning()` – angezeigte Warnung in `audit_log` speichern (Stufe, Zeitstempel, child_id, user_id)
- [ ] Hinweistexte in klarer Sprache ohne Vorwurfston
- [ ] Warnsystem blockiert NICHT – Nutzer kann trotzdem fortfahren

### 1.9 PDF-Export Basisversion (F-61a, F-63a, F-64a, F-64b, F-65a, F-66a)

- [ ] DOMPDF einrichten (`barryvdh/laravel-dompdf`)
- [ ] Blade-Templates fuer PDF-Layouts erstellen
- [ ] Deckblatt: Kindname, Zeitraum, Ersteller, Erstellungsdatum, SHA-256-Hash
- [ ] Wasserzeichen: Nutzername + Datum (Graustufen 30%)
- [ ] Fusszeile: Seitenzahl, Zeitstempel, Hinweis "Dokument unveraenderlich"
- [ ] **F-64a** Uebergabe-Einzelprotokoll (Checkliste, Zustand Kind, Fotos, Benachrichtigungs-Tracking, Hash)
- [ ] **F-63a** Vorfallsbericht fuer Behoerden (Vorfaelle, Eskalationsstufen, Belege)
- [ ] **F-61a** Medizinischer Bericht (Arztbesuche, Medikamente, Impfungen)
- [ ] **F-62a** Terminchronik
- [ ] **F-64b** Uebergabe-Sammelbericht (Statistik, Stimmungsverlauf, Meldungs-Haeufigkeit)
- [ ] **F-65a** Gesamtbericht Kind (alle Bereiche in einem PDF)
- [ ] **F-66a** Wasserzeichen + SHA-256-Hash automatisch auf allen Exporten
- [ ] PdfExportService als Laravel Service-Klasse implementieren
- [ ] PDF-Generierung als asynchroner Queue-Job

### 1.10 Medizinische Basis-Dokumentation (F-20, F-21)

- [ ] **F-20** Arzt-/Therapeuten-Verwaltung (Fachrichtung, Praxis, Adresse, Telefon, aktiv seit/bis)
- [ ] **F-21** Arztbesuche protokollieren (Datum, Arzt, Grund, Diagnose ICD-10 optional, Anhaenge)

### 1.11 Vorfalls- und Beobachtungsdokumentation (F-40 bis F-45)

- [ ] **F-40** Vorfall erfassen (Datum, Uhrzeit, Ort, Beschreibung, Beteiligte, Zeugen)
- [ ] **F-41** Stimmungs- und Verhaltensprotokoll (taeglich: Skala 1-5 + Freitext, Dauer < 2 Min.)
- [ ] **F-42** Kommunikationsprotokoll (Nachrichten mit Co-ET; Kategorie)
- [ ] **F-43** Medienbeweise anfuegen (Fotos, Screenshots, Audio max. 50 MB, Hash-Pruefung)
- [ ] **F-44** Eskalationsstufen-Markierung (Keine / Beobachtung / Sorge / Dringlich – farbkodiert)
- [ ] **F-45** Timeline-Ansicht (chronologische Gesamtuebersicht aller Ereignisse pro Kind)

### 1.12 Dashboard Basisversion (F-70a, F-71a, F-72a)

- [ ] **F-70a** Startseite: Uebersicht (Termine, offene Protokolle, Medikamenten-Erinnerungen)
- [ ] **F-71a** Schnelleingabe Vorfall / Uebergabe (max. 1 Klick vom Dashboard)
- [ ] **F-72a** Kalenderuebersicht Monat (alle Kinder, farbkodiert)

### 1.13 Sicherheit und DSGVO Phase 1

- [ ] AES-256-Verschluesselung aller Dateien und Fotos at rest
- [ ] SHA-256-Hash fuer alle Protokolle und Fotos (Integritaetspruefung)
- [ ] GPS-Daten automatisch aus Fotos entfernen
- [ ] Laravel Storage konfigurieren (kein oeffentlicher URL fuer Kinderfotos)
- [ ] Prepared Statements fuer alle DB-Abfragen (Eloquent)
- [ ] OWASP Top 10 als Entwicklungs-Checkliste abarbeiten
- [ ] Datenschutzerklaerung nach Art. 13/14 DSGVO erstellen
- [ ] Einwilligungsmanagement implementieren
- [ ] Recht auf Auskunft (Art. 15), Loeschung (Art. 17), Berichtigung (Art. 16) implementieren
- [ ] AVV mit allen Sub-Dienstleistern abschliessen
- [ ] VVT (Verzeichnis der Verarbeitungstaetigkeiten) nach Art. 30 DSGVO erstellen

### 1.14 Testing Phase 1

- [ ] Unit-Tests fuer WarnService (alle Stufen, Grenzwerte)
- [ ] Unit-Tests fuer HashService und EncryptionService
- [ ] Feature-Tests fuer Uebergabe-Schnellerfassung (Abholung + Abgabe)
- [ ] Feature-Tests fuer Benachrichtigungs-Tracking
- [ ] Feature-Tests fuer PDF-Export (Hash-Integritaet)
- [ ] Usability-Test: Uebergabe-Schnellerfassung in unter 3 Minuten
- [ ] Backup-Restore-Test (Wiederherstellung innerhalb 4 Stunden)
- [ ] SHA-256-Hash-Integritaet aller Protokolle und Fotos verifizieren

---

## Phase 2 – Ausbau (Monat 5–8)

### 2.1 Terminkalender (F-30 bis F-35)

- [ ] **F-30** Terminkalender pro Kind (Monats-/Wochen-/Listenansicht, farbliche Kategorisierung)
- [ ] **F-31** Terminkategorien (Arzt, Schule, Gericht, Uebergabe, Freizeit – erweiterbar)
- [ ] **F-32** Terminprotokoll (Freitext-Nachbericht, Datei-Anhaenge, Stimmungs-Indikator)
- [ ] **F-33** Wiederkehrende Termine (taeglich/woechentlich/monatlich/benutzerdefiniert)
- [ ] **F-34** E-Mail- und Push-Benachrichtigungen (24h und 1h vor Termin, konfigurierbar)
- [ ] **F-35** Kalender-Export iCal (.ics-Export fuer externe Kalender)

### 2.2 Medizinische Erweiterungen (F-22 bis F-26)

- [ ] **F-22** Impf-Dokumentation (Impfstoff, Datum, Charge, Arzt, naechster Termin)
- [ ] **F-23** Medikamenten-Verwaltung (Praeparat, Dosierung, Einnahmezeitraum, verschreibender Arzt)
- [ ] **F-24** Befunde und Dokumente (Upload PDF/Bild, verschluesselt, suchbar)
- [ ] **F-25** Erinnerung Vorsorgeuntersuchungen (Push/E-Mail 4 Wochen vor U1-U11, J1)
- [ ] **F-26** Medizinischer Notfallpass (Einseiter: Diagnosen, Medi, Arzt, Allergie – druckbar)

### 2.3 Internes Nachrichtensystem (F-50 bis F-53)

- [ ] **F-50** Internes Nachrichtensystem (revisionssicher, nicht loeschbar)
- [ ] **F-51** BIFF-Vorlage-Assistent (kurze, informative, freundliche, feste Nachrichten-Vorlagen)
- [ ] **F-52** Lesebestaetigung (Zeitstempel wann gelesen – nicht loeschbar)
- [ ] **F-53** Nachrichtenarchiv 10 Jahre (alle Nachrichten aufbewahrt, exportierbar)

### 2.4 Dashboard-Erweiterungen (F-73a bis F-75a, F-90, F-96)

- [ ] **F-73a** Stimmungs-Trenddiagramm (Abholung vs. Abgabe ueber Zeit; PDF-exportierbar)
- [ ] **F-74a** Benachrichtigungs-Statistik (Meldungen letzte 7/30/90 Tage; Warnstufe sichtbar)
- [ ] **F-75a** Monatliche Warn-Zusammenfassung (haeufig fehlende Eintraege + Handlungsempfehlung)
- [ ] **F-90** Benachrichtigungs-Statistik im Dashboard (Anzahl Meldungen letzte 7/30/90 Tage pro Kind)
- [ ] **F-96** Monatliche Zusammenfassung: Uebersicht Meldungen + haeufigste fehlende Eintraege

### 2.5 PDF-Erweiterungen (F-67a)

- [ ] **F-67a** Leer-Uebergabeformular (druckbar; QR-Code fuer spaetere digitale Zuordnung)
- [ ] Uebergabe-Sammelbericht: Stimmungs-Trenddiagramm einbetten
- [ ] Uebergabe-Sammelbericht: Benachrichtigungs-Statistik + Warn-Ereignisse

### 2.6 Warnsystem-Erweiterungen (F-94, F-95)

- [ ] **F-94** Warnschwellen konfigurierbar durch Nutzer (Anpassung an dichten Umgang)
- [ ] **F-95** Warnungen werden im Protokoll gespeichert (jede angezeigte Warnung mit Zeitstempel)

### 2.7 PWA und Mehrsprachigkeit

- [ ] PWA Offline-Modus vollstaendig implementieren (Service Worker, Offline-Erfassung)
- [ ] Mehrsprachigkeit Englisch (Laravel Localization)

### 2.8 Testing Phase 2

- [ ] Feature-Tests fuer Terminkalender (inkl. wiederkehrende Termine)
- [ ] Feature-Tests fuer Nachrichtensystem (Unveraenderlichkeit, Lesebestaetigung)
- [ ] Tests fuer Benachrichtigungs-Statistiken und Dashboard-Diagramme
- [ ] Offline-PWA-Tests
- [ ] Penetrationstest (jaehrlich; keine kritischen Befunde als Go-Live-Kriterium)

---

## Phase 3 – Premium (Monat 9–12)

### 3.1 Erweiterte Features

- [ ] Digitale PDF-Signierung via eIDAS
- [ ] OCR-Import U-Heft (Vorsorgeuntersuchungen automatisch erkennen und eintragen)
- [ ] KI-gestuetzte Berichts-Zusammenfassung (Opt-in)
- [ ] Integration hochkonflikteltern.de (eigenstaendige App – Verlinkung / Co-Marketing)

### 3.2 Abnahme und Go-Live

- [ ] Alle F-MUSS-Anforderungen implementiert und getestet (Unit + Feature Tests)
- [ ] Abholung UND Abgabe als getrennte Erfassungen korrekt dokumentiert
- [ ] Warnsystem Stufe 1-3 bei Testdaten ausgeloest und protokolliert
- [ ] Benachrichtigungs-Tracking vollstaendig im PDF-Export sichtbar
- [ ] DSFA (Datenschutz-Folgenabschaetzung) durchgefuehrt und dokumentiert
- [ ] Penetrationstest ohne kritische Befunde abgeschlossen
- [ ] Usability-Test: Uebergabe-Schnellerfassung vollstaendig in unter 3 Minuten
- [ ] SHA-256-Hash-Integritaet aller Protokolle und Fotos verifiziert
- [ ] Backup-Restore-Test innerhalb 4 Stunden erfolgreich
- [ ] Verfuegbarkeit 99,5% sichergestellt (Monitoring aktiv)
- [ ] Seitenlade-Zeit < 2 Sekunden (3G) verifiziert

---

## Uebersicht: Alle Muss-Anforderungen (F-MUSS)

| ID | Bereich | Anforderung | Phase | Erledigt |
|----|---------|-------------|-------|----------|
| F-01 | Auth | Registrierung E-Mail + sicheres Passwort | 1 | [ ] |
| F-03 | Auth | Passwort-Reset per E-Mail-Link | 1 | [ ] |
| F-04 | Auth | Automatischer Session-Timeout | 1 | [ ] |
| F-05 | Auth | Einladungssystem Co-Elternteil / Fachkraft | 1 | [ ] |
| F-06 | Auth | Audit-Log aller Anmeldevorgaenge | 1 | [ ] |
| F-07 | Auth | Kontosperrung nach Fehlversuchen | 1 | [ ] |
| F-10 | Profil | Kinderprofil anlegen | 1 | [ ] |
| F-11 | Profil | Mehrere Kinder pro Account | 1 | [ ] |
| F-12 | Profil | Notfallkontakte pro Kind | 1 | [ ] |
| F-13 | Profil | Allergien und Dauermedikation | 1 | [ ] |
| F-20 | Medizin | Arzt-/Therapeuten-Verwaltung | 1 | [ ] |
| F-21 | Medizin | Arztbesuche protokollieren | 1 | [ ] |
| F-22 | Medizin | Impf-Dokumentation | 2 | [ ] |
| F-23 | Medizin | Medikamenten-Verwaltung | 2 | [ ] |
| F-24 | Medizin | Befunde und Dokumente | 2 | [ ] |
| F-30 | Termine | Terminkalender pro Kind | 2 | [ ] |
| F-31 | Termine | Terminkategorien | 2 | [ ] |
| F-32 | Termine | Terminprotokoll | 2 | [ ] |
| F-33 | Termine | Wiederkehrende Termine | 2 | [ ] |
| F-34 | Termine | E-Mail- und Push-Benachrichtigungen | 2 | [ ] |
| F-40 | Vorfall | Vorfall erfassen | 1 | [ ] |
| F-41 | Vorfall | Stimmungs- und Verhaltensprotokoll | 1 | [ ] |
| F-42 | Vorfall | Kommunikationsprotokoll | 1 | [ ] |
| F-43 | Vorfall | Medienbeweise anfuegen | 1 | [ ] |
| F-44 | Vorfall | Eskalationsstufen-Markierung | 1 | [ ] |
| F-45 | Vorfall | Timeline-Ansicht | 1 | [ ] |
| F-53 | Nachricht | Nachrichtenarchiv 10 Jahre | 2 | [ ] |
| F-61a | PDF | Medizinischer Bericht | 1 | [ ] |
| F-62a | PDF | Terminchronik | 1 | [ ] |
| F-63a | PDF | Vorfallsbericht fuer Behoerden | 1 | [ ] |
| F-64a | PDF | Uebergabe-Einzelprotokoll | 1 | [ ] |
| F-64b | PDF | Uebergabe-Sammelbericht | 1 | [ ] |
| F-65a | PDF | Gesamtbericht Kind | 1 | [ ] |
| F-66a | PDF | Wasserzeichen + SHA-256-Hash | 1 | [ ] |
| F-70a | Dashboard | Startseite Uebersicht | 1 | [ ] |
| F-71a | Dashboard | Schnelleingabe Vorfall / Uebergabe | 1 | [ ] |
| F-72a | Dashboard | Kalenderuebersicht Monat | 1 | [ ] |
| F-86 | Benachrichtigung | Benachrichtigungs-Tracking nach Uebergabe | 1 | [ ] |
| F-89 | Benachrichtigung | Benachrichtigungs-Verlauf im Protokoll | 1 | [ ] |
| F-91 | Warnsystem | Warnstufe 1 (gelb) ab 3 Meldungen / 7 Tage | 1 | [ ] |
| F-92 | Warnsystem | Warnstufe 2 (orange) bei Muster-Erkennung | 1 | [ ] |
| F-93 | Warnsystem | Warnstufe 3 (rot) ab 5 Meldungen / 7 Tage | 1 | [ ] |

---

## Technische Qualitaetssicherung (laufend)

- [ ] OWASP Top 10 Checkliste regelmaessig abarbeiten
- [ ] Jaehrlicher Penetrationstest (erstes Mal vor Go-Live)
- [ ] Code-Reviews vor jedem Merge
- [ ] Laravel-Pakete und Abhaengigkeiten regelmaessig aktualisieren
- [ ] Backup-Restore-Tests quartalsweise wiederholen
- [ ] Performance-Tests bei Release (Seitenlade-Zeit < 2s auf 3G)
- [ ] WCAG 2.1 Level AA Konformitaet sicherstellen

---

*Erstellt auf Basis: KiDok Anforderungsdokument v2.2 FINAL – Stand Maerz 2026*
