# revio 4 Setup – Installation und Wartung mit dem Setup-Assistenten

Diese Anleitung beschreibt das grafische **revio 4 Setup** (`RevioSetup.exe`) – den
geführten Assistenten für Installation, Update, Wartung und Deinstallation von revio 4.
Das Setup ersetzt die frühere manuelle Installation über PowerShell-Skripte,
Browser-Konfiguration und Client-Import.

> **Kurzfassung:** `RevioSetup.exe` als Administrator starten, auf der Startseite den
> gewünschten Modus wählen (Backend, Client oder Terminal-Server / RDS) und den
> Assistenten Seite für Seite durchlaufen: **Voraussetzungen → Konfiguration →
> Zusammenfassung → Installation → Abschluss**. Ist bereits ein Backend oder Client
> installiert, bietet die Startseite stattdessen Update, Zertifikatswechsel oder
> Deinstallation an.

---

## 1. Über das revio 4 Setup

Das Setup ist eine eigenständige Windows-Desktop-Anwendung (WPF, .NET 9). Sie wird als
**einzelne, self-contained `RevioSetup.exe`** ausgeliefert und benötigt auf dem Zielsystem
keine vorinstallierten Runtimes. Beim Start prüft das Setup automatisch, ob bereits ein
Backend (Windows-Dienst `revioServer`) oder ein Client (MSIX für den aktuellen Benutzer)
vorhanden ist, und passt die Auswahl entsprechend an.

Auf der Startseite (**„Was möchten Sie installieren?"**) stehen drei Modi zur Auswahl:

| Modus (Startseite) | Zweck | Ergebnis |
|---|---|---|
| **Backend (Server)** | revio-Server als Windows-Dienst installieren | SSL-Zertifikat, `appsettings.json`, Dienst `revioServer`, Firewall-Regeln, MongoDB, Admin-Benutzer, Lizenz, Vorlagen und `revioClientConfig.zip` |
| **Client (dieser Computer)** | revio 4 Client auf einem Einzelplatz installieren | MSIX-Client inkl. Voraussetzungen, Server-Zertifikat, Serververbindung und Desktop-Verknüpfung |
| **Terminal-Server / RDS (Client-Verteilung)** | Client für **alle Benutzer** eines Sitzungshosts bereitstellen | Zentrale Ablage mit Freigabe, Logon-Skript und Anmeldungs-Task; MSIX wird pro Benutzer bei der Anmeldung registriert |

> **Hinweis:** Die gesamte Konfiguration läuft nativ im Setup – ein Browser oder WebView2
> wird **nicht** benötigt.

---

## 2. Systemvoraussetzungen

| Voraussetzung | Details |
|---|---|
| **Betriebssystem** | Backend: **Windows Server 2022 oder höher** empfohlen (Build ≥ 20348). Client: Windows Server 2019+, Windows 10/11. Ältere Systeme werden als Warnung gemeldet, blockieren aber nicht. |
| **Administratorrechte** | Das Setup verlangt eine UAC-Elevation (`requireAdministrator`) – nötig für Dienste, Zertifikate, Firewall und Fonts. Ohne Adminrechte bricht die Voraussetzungsprüfung ab. |
| **Freier Speicherplatz** | Backend ≥ 100 GB, Client ≥ 5 GB (Warnung bei Unterschreitung). |
| **Internetverbindung** | Für den Download der Installationspakete von `reviofiles.blob.core.windows.net`. Alternativ können Pakete lokal angegeben werden. |
| **Freie Ports (Backend)** | Standard **HTTP 27010**, **HTTPS 27011**, **MongoDB 27040**. Belegte Ports blockieren die Installation. |
| **Für RDS zusätzlich** | RD-Sitzungshost-Rolle und – bei GPO-Verteilung – Domänenmitgliedschaft (siehe eigene Anleitung, Abschnitt 6). |

Fehlende Client-Laufzeiten (ASP.NET Core 9, Windows Desktop Runtime 9, Windows App
Runtime 1.8, Segoe-Fluent-Icons) werden vom Setup **automatisch nachinstalliert** und
lediglich als Hinweis angezeigt.

---

## 3. Setup starten

1. `RevioSetup.exe` auf den Zielserver kopieren (nur diese eine Datei nötig).
2. Per Rechtsklick **„Als Administrator ausführen"** starten (bzw. UAC bestätigen).
3. Beim ersten Start entpackt .NET einmalig einige Komponenten nach `%TEMP%` – das kann
   ein paar Sekunden dauern.

**Bedienung des Assistenten:** Unten führen die Schaltflächen **„Weiter"** und
**„Zurück"** durch die Seiten; ein Fortschrittsbalken zeigt „Schritt n von m". Auf der
Zusammenfassungsseite wird die Schaltfläche kontextabhängig beschriftet
(z. B. **„Installieren"**, **„Update starten"**, **„Deinstallieren"**). Die letzte Seite
schliesst das Fenster mit **„Schliessen"**.

**Protokoll:** Jeder Lauf wird nach `%ProgramData%\revio\Setup\logs\setup_*.log`
protokolliert (mit Live-Anzeige im Wizard). Passwörter, Lizenzschlüssel und Zugangsdaten
werden dabei maskiert und nie im Klartext gespeichert.

### 3.1 Vorabversion installieren (`/preRelease`)

Standardmässig lädt das Setup Backend und Client aus dem Kanal **`latest`** (die freigegebene
Version). Mit dem optionalen Schalter `/preRelease` werden stattdessen die **Vorabversionen**
dieser beiden Pakete geladen:

```powershell
RevioSetup.exe /preRelease
```

- Betrifft ausschliesslich das **Backend-Server-Paket** (`revioServer_x64.zip`) und das
  **Client-Paket** (`revioClient.zip`). Datenbank-Engine, .NET-Laufzeiten und Schriftart
  bleiben unverändert.
- Statt `/preRelease` ist auch `--prerelease` zulässig (Gross-/Kleinschreibung egal).
- Der Assistent verhält sich ansonsten **exakt gleich** wie ohne den Schalter. Auf der
  Zusammenfassungsseite sind die verwendeten Download-Adressen ersichtlich (mit `/preRelease/`
  im Pfad), ebenso im Protokoll.
- Der Schalter wirkt **pro Aufruf** – für eine reguläre Installation einfach ohne ihn starten.

> **Hinweis:** Vorabversionen sind für Test- und Abnahmezwecke gedacht und sollten nicht
> ungeprüft in Produktivumgebungen eingespielt werden.

---

## 4. Backend installieren (Server)

Auf der Startseite **„Backend (Server)"** wählen und **„Weiter"**. Es folgt die Wahl des
Installationsprofils:

- **Standardinstallation (empfohlen)** – eine einzige, schlanke Konfigurationsseite mit
  sinnvollen Vorgaben.
- **Erweiterte Installation** – zwei Konfigurationsseiten mit voller Kontrolle über
  Ports, Dienstnamen, Zertifikats-DNS und MongoDB-Absicherung.

Danach durchläuft der Assistent:

**Voraussetzungen** → **Konfiguration** → **Zusammenfassung** → **Installation** →
**Abschluss**.

### 4.1 Voraussetzungsprüfung

Die Seite prüft automatisch Administratorrechte, Betriebssystem, freien Speicherplatz,
Internetverbindung, die Belegung von HTTP-/HTTPS-Port, den Backend-Dienstnamen und die
MongoDB. Rot markierte, blockierende Fehler (fehlende Adminrechte, belegter Port,
vergebener Dienstname) verhindern das Weiterklicken – nach dem Beheben **„Erneut prüfen"**.
Warnungen (gelb) blockieren nicht.

> **Hinweis – bestehende Datenbank:** Erkennt die Prüfung eine bereits vorhandene revio-
> Datenbank, die zum konfigurierten Dienstnamen/Port passt, schaltet das Setup automatisch
> in den **Weiterverwenden-Modus** (siehe 4.4).

### 4.2 Standardinstallation

Eine Seite mit den wichtigsten Angaben:

| Feld | Vorgabe / Hinweis |
|---|---|
| **Installationsverzeichnis** | `C:\revio4backend` (Durchsuchen-Schaltfläche vorhanden) |
| **Datenverzeichnis (MongoDB)** | wird aus dem Installationspfad abgeleitet: `…\data\revio4` |
| **Admin-Passwort** + **Bestätigung** | Passwort des ersten Administrators, mind. 6 Zeichen |
| **E-Mail des ersten Benutzers** | optional |
| **Lizenzschlüssel** | optional; muss eine gültige GUID sein (oder leer bleiben) |

Standardmässig werden Fluent-Font, Firewall-Regeln, Dienst-Autostart und die
Vorlagen-Installation aktiviert; das Zertifikat wird auf den **Computernamen** ausgestellt.

### 4.3 Erweiterte Installation

**Seite 1 – Backend-Konfiguration:**

| Feld | Vorgabe |
|---|---|
| **Installationsverzeichnis** | `C:\revio4backend` |
| **HTTP-Port** / **HTTPS-Port** | `27010` / `27011` (müssen unterschiedlich sein) |
| **Dienstname** | `revioServer` (ohne Leerzeichen) |
| **Zertifikats-DNS** | **„Computername verwenden (\<Rechnername\>)"** oder eigener DNS-Name/FQDN |
| **Optionen** | Fluent-Font installieren, Firewall-Regeln erstellen, Dienst nach Installation starten |

**Seite 2 – Datenbank & Konfiguration:**

| Feld | Vorgabe |
|---|---|
| **Datenverzeichnis** | aus Installationspfad abgeleitet |
| **Datenbank-Port** / **-Dienstname** | `27040` / `revio4Db` (Port und Name dürfen nicht mit dem Backend kollidieren) |
| **MongoDB absichern** | Admin-User + Keyfile + Authentifizierung aktivieren |
| **Admin-Passwort**, **E-Mail**, **Lizenzschlüssel** | wie Standardinstallation |
| **Vorlagen installieren** | Standard-Vorlagen nach der Konfiguration einspielen |

### 4.4 Bestehende Datenbank weiterverwenden

Wird eine vorhandene MongoDB erkannt, blendet die Konfigurationsseite den Hinweis
**„Bestehende Datenbank wird weiterverwendet …"** ein: das Datenverzeichnis (und in der
erweiterten Installation Port/Dienstname) ist gesperrt, das Passwortfeld wird zu
**„Admin-Passwort (bestehend)"** und verlangt das Kennwort der bestehenden Installation.

### 4.5 Installation und Abschluss

Nach der **Zusammenfassung** (alle geplanten Aktionen auf einen Blick) startet mit
**„Installieren"** die durchgehende Pipeline: Backend-Dateien/Dienst/Zertifikat/Firewall →
MongoDB → Admin-/Lizenz-/Vorlagen-Konfiguration → Erzeugung der `revioClientConfig.zip`.
Eine Schrittliste und ein Live-Protokoll zeigen den Fortschritt.

Schlägt die Installation fehl, macht das Setup Dienst, Firewall-Regeln und Zertifikat
automatisch rückgängig (**Rollback**); Dateien und Logs bleiben zur Analyse erhalten.

Die **Abschlussseite** listet die Ergebnisse (inkl. Pfad der `revioClientConfig.zip`),
offene Punkte und den Pfad zur Logdatei.

> **Wichtig:** Die `revioClientConfig.zip` (Zertifikat + Serververbindung) wird für die
> Client-Installation und die RDS-Verteilung benötigt. Sie liegt standardmässig im
> Installationsverzeichnis des Backends.

---

## 5. Client installieren (Einzelplatz)

Für einen einzelnen Arbeitsplatz auf der Startseite **„Client (dieser Computer)"** wählen.
Der Assistent führt durch **Voraussetzungen → Client-Konfiguration → Zusammenfassung →
Installation → Abschluss**.

**Client-Konfiguration:**

| Einstellung | Beschreibung |
|---|---|
| **Paketquelle** | `revioClient.zip` herunterladen (Standard) **oder** lokale ZIP-Datei angeben |
| **`revioClientConfig.zip`** | Zertifikat + Serververbindung; bei lokal installiertem Backend automatisch vorbelegt |
| **Serververbindung** | Titel, Server-URL (z. B. `https://SERVERNAME:27011`), zuletzt angemeldeter Benutzer |
| **Optionen** | .NET-Runtimes installieren, Windows App Runtime installieren, Server-Zertifikat importieren, `ServerList.json` schreiben, Desktop-Verknüpfung erstellen |

Die Installation importiert das Server-Zertifikat in *Vertrauenswürdige
Stammzertifizierungsstellen*, registriert das MSIX-Paket, schreibt die Serververbindung ins
Benutzerprofil und legt die Desktop-Verknüpfung **„revio 4"** an.

---

## 6. Terminal-Server / RDS (Client-Verteilung)

Für Mehrbenutzerumgebungen (Remotedesktop-Sitzungshosts) verteilt das Setup den Client
**pro Benutzer bei der Anmeldung**. Auf der Startseite
**„Terminal-Server / RDS (Client-Verteilung)"** wählen. Das Setup legt eine zentrale Ablage
(Standard `C:\Deploy\revioClient`) mit SMB-Freigabe, ein idempotentes Logon-Skript
(`revioClient-Logon.ps1`) und – wahlweise – einen Anmeldungs-Task an.

Die Verteilung erfolgt anschliessend entweder über einen **Anmeldungs-Task** (einfachster
Weg für einen einzelnen Server) oder über eine **Gruppenrichtlinie (GPO)** in der Domäne.

> **Ausführliche Anleitung:** Die vollständige Beschreibung der GPO-Verteilung, der
> nötigen Einstellungen im Setup, Server-2019-Besonderheiten und Fehlerbehebung finden Sie
> in [revio 4 Client – Verteilung auf Terminal-Servern (RDS) per Gruppenrichtlinie (GPO)](rds-client-verteilung-gpo.md).

---

## 7. Wartung eines bestehenden Backends

Ist bereits ein Backend installiert, zeigt die Startseite unter der Backend-Karte den
Balken **„Backend bereits installiert"** mit folgenden Optionen (bei mehreren Instanzen
zuerst die betroffene Instanz auswählen):

| Option | Wirkung |
|---|---|
| **Update auf die aktuelle Version durchführen** | Aktualisiert die Backend-Dateien auf die neueste Version. `appsettings.json` (Ports, Zertifikat, JWT-Key, DB-Verbindung) sowie `mongod.cfg`/Keyfile bleiben erhalten. |
| **Diese Instanz deinstallieren** | Entfernt Dienst, Firewall-Regeln, Zertifikat und Dateien. **Die MongoDB-Datenbank bleibt vollständig erhalten.** Zusätzlich wählbar: *Installationsdateien löschen* und *SSL-Zertifikat aus dem Zertifikatsspeicher entfernen*. |
| **Weitere, unabhängige Backend-Instanz installieren** | Legt eine zusätzliche Instanz mit kollisionsfreien Ports, Namen und Pfaden an (bestehende Instanz bleibt unberührt). |
| **Zertifikat neu erstellen (DNS-Namen korrigieren)** | Stellt das SSL-Zertifikat neu aus – auf den Computernamen oder einen eigenen DNS-Namen/FQDN; optional *altes Zertifikat anschliessend aus dem Speicher entfernen*. |

> **Hinweis:** Beim Deinstallieren werden die MongoDB (Dienst `revio4Db`) und die
> db-/data-/log-Verzeichnisse **nie** angetastet. Zum Verwalten der Datenbank siehe 7.1.

### 7.1 MongoDB-Datenbank verwalten

Existieren Datenbank-Instanzen, erscheint die Karte **„MongoDB-Datenbank verwalten"**. Wählen
Sie zuerst die betroffene Instanz – die Karte zeigt deren **Engine-Version** und den **belegten
Speicher**. Alle Aktionen wirken **unabhängig vom Backend**:

| Aktion | Wirkung |
|---|---|
| **Datenbank aktualisieren** | Erscheint nur, wenn eine neuere, mit diesem Windows kompatible MongoDB-Engine vorliegt. Aktualisiert die Engine an Ort und Stelle; **Daten, `mongod.cfg` und Keyfile bleiben unverändert**. Vor dem Update wird automatisch eine Kaltkopie des Datenverzeichnisses angelegt (Rollback bei Fehler). Nur online. |
| **Daten kopieren …** | Exportiert **alle Anwendungsdaten** der Instanz in eine einzelne, portable Datei (Migration, siehe 7.1.1). |
| **Daten ersetzen …** | Spielt eine zuvor erstellte Kopie in diese Instanz ein und ersetzt deren Anwendungsdaten (Migration, siehe 7.1.1). |
| **Ausgewählte Datenbank löschen …** | Entfernt die Instanz vollständig und setzt den Server in den Ur-Zustand zurück (siehe 7.1.2). |

#### 7.1.1 Daten migrieren (Umgebung A → B)

Mit **Daten kopieren** und **Daten ersetzen** verschieben Sie Prüfdaten zwischen zwei
Umgebungen – etwa von einem Test- auf einen Produktivserver:

1. **Auf Umgebung A – Kopie erstellen:** Schaltfläche **„Daten kopieren …"**, Zieldatei wählen
   (Vorschlag `revio-daten-<Dienst>-<Zeitstempel>.archive.gz`). Das Setup lädt bei Bedarf die
   MongoDB Database Tools, prüft sie per Prüfsumme und exportiert alle Anwendungsdaten in **eine**
   komprimierte Datei (dazu eine kleine `.json`-Begleitdatei mit Versions- und Inhaltsangaben).
2. **Datei übertragen:** Kopieren Sie die `.archive.gz`-Datei auf Umgebung B (Netzlaufwerk,
   USB-Stick o. Ä.).
3. **Auf Umgebung B – Kopie einspielen:** Schaltfläche **„Daten ersetzen …"**, Kopie-Datei wählen
   und bestätigen. Vor dem Ersetzen erstellt das Setup **automatisch eine Sicherung** der aktuellen
   Daten (`revio-daten-<Dienst>-pre-restore-<Zeitstempel>.archive.gz`, neben der gewählten Datei);
   schlägt das Einspielen fehl, wird daraus automatisch zurückgerollt.

Es werden ausschliesslich **Anwendungsdaten** übertragen. Die MongoDB-internen Datenbanken
(`admin`, `config`, `local`) bleiben ausgenommen – **Zugangsdaten und Keyfile der Zielumgebung
bleiben unangetastet**. Beim Einspielen werden nur die in der Kopie enthaltenen Collections
ersetzt; sonstige, nicht in der Kopie enthaltene Daten der Zielumgebung bleiben erhalten.

> **Voraussetzungen und Hinweise:**
> - **Online-Verbindung erforderlich** – die Database Tools (mongodump/mongorestore) werden bei
>   Bedarf von MongoDB geladen und geprüft; im Offline-Modus sind die beiden Schaltflächen
>   ausgeblendet.
> - **Freier Speicherplatz** wird vorab je Laufwerk geprüft (Vorab-Sicherung und Einspielen);
>   reicht er nicht aus, bricht das Setup **vor jeder Änderung** ab.
> - **„Daten ersetzen" ist destruktiv:** Zur Sicherheit müssen Sie im Bestätigungsdialog das Wort
>   **`ERSETZEN`** eingeben, bevor die Aktion frei wird. Die automatische Vorab-Sicherung dient als
>   Rückfalloption.

#### 7.1.2 MongoDB-Datenbank vollständig löschen (Ur-Zustand)

Die Aktion **„Ausgewählte Datenbank löschen …"** entfernt eine Datenbank-Instanz vollständig
(Dienst, Datenverzeichnis, Keyfile) und setzt den Server in den Zustand vor der Installation
zurück. Optional werden auch die MongoDB-Programmdateien mitentfernt.

> **Achtung – unwiderruflich:** Das Löschen der Datenbank ist **nicht umkehrbar**. Zur
> Sicherheit verlangt das Setup eine getippte Bestätigung: Sie müssen im Bestätigungsdialog
> das Wort **`LÖSCHEN`** eingeben, bevor die rote Schaltfläche **„Endgültig löschen"** frei
> wird.

---

## 8. Client aktualisieren oder deinstallieren

Ist bereits ein Client installiert, zeigt die Startseite den Balken **„Client bereits
installiert"**:

| Option | Wirkung |
|---|---|
| **Client aktualisieren / neu installieren** | Installiert die aktuelle Client-Version über die bestehende. |
| **Client deinstallieren** | Entfernt Desktop-Verknüpfung, Anheftungen und die MSIX-App. Optional zusätzlich: *RDS-Verteilung entfernen* (Anmeldungs-Task, Freigabe, zentrale Ablage) und *Serververbindungen (ServerList.json) des aktuellen Benutzers entfernen*. |

---

## 9. Protokolle und Fehlerbehebung

| Symptom | Ursache / Lösung |
|---|---|
| Setup startet nicht / Fenster verschwindet | Bekannter DirectWrite-/Font-Cache-Effekt. Das Setup startet die Oberfläche automatisch bis zu 3× neu; siehe `%ProgramData%\revio\Setup\logs\launcher.log`. |
| „Administratorrechte" schlägt fehl | Setup als Administrator starten (Rechtsklick → „Als Administrator ausführen"). |
| „Port ist bereits belegt" | Anderen HTTP-/HTTPS-/DB-Port wählen oder den belegenden Dienst beenden, dann **„Erneut prüfen"**. |
| „Windows-Dienst '\<name\>' … vergeben" | Anderen Dienstnamen wählen (Backend- und DB-Dienstname müssen sich unterscheiden). |
| Installation fehlgeschlagen | Live-Protokoll und Logdatei prüfen (Schaltfläche **„Logdatei öffnen"** auf der Fortschritts-/Abschlussseite). Bei der Backend-Installation wurden Dienst/Firewall/Zertifikat automatisch zurückgerollt. |

**Logdateien:**

```
%ProgramData%\revio\Setup\logs\setup_*.log       # Ablauf des jeweiligen Setup-Laufs
%ProgramData%\revio\Setup\logs\launcher.log      # Start-/Watchdog-Protokoll
```

---

## 10. Offline-Installation (Paket vorab erstellen)

Für Zielserver ohne direkten Zugang zu `reviofiles.blob.core.windows.net` lässt sich das
Setup **offline** betreiben: Sämtliche Installationspakete werden vorab auf einem Rechner
mit Internetzugang in **ein einziges Artefakt-Paket** (`revioSetupArtifacts.zip`) gebündelt
und auf dem Zielserver von dort statt aus dem Internet installiert.

> **Wichtig:** Offline bezieht sich nur auf die *Installationspakete*. Die **Lizenzprüfung**
> und der Paketabruf laufen weiterhin über `https://intern.revioservices.ch` – hierfür
> bleibt auf dem Zielserver ein Internetzugriff erforderlich.

### 10.1 Artefakt-Paket erstellen

Auf einem Rechner **mit Internetzugang** `RevioSetup.exe` einmalig im Konsolenmodus
`/SingleArtifact` aufrufen:

```powershell
RevioSetup.exe /SingleArtifact [Ausgabeverzeichnis]
```

- Ohne Angabe wird das Paket im **aktuellen Verzeichnis** abgelegt; ein optionales erstes
  Argument gibt das Ausgabeverzeichnis vor.
- Statt `/SingleArtifact` sind auch `--singleartifact` zulässig (Gross-/Kleinschreibung egal).
- Der Aufruf läuft **ohne Oberfläche** in der Konsole, zeigt den Fortschritt an und liefert
  einen Exit-Code zurück (`0` = Erfolg, `1` = Fehler).

Der Befehl lädt alle Artefakte (Backend-, Datenbank- und Client-ZIP, die drei Laufzeiten
und den Segoe-Fluent-Font) herunter und schreibt sie zusammen mit einer
`manifest.json` (Grösse und SHA-256 je Datei) in die Datei **`revioSetupArtifacts.zip`**.

Der Schalter `/preRelease` (siehe [Abschnitt 3.1](#31-vorabversion-installieren-prerelease))
lässt sich kombinieren – dann enthält das Paket die **Vorabversionen** von Backend und Client:

```powershell
RevioSetup.exe /SingleArtifact /preRelease [Ausgabeverzeichnis]
```

> **Wichtig:** Wird ein solches Vorabversions-Paket verwendet, muss die anschliessende
> Offline-Installation auf dem Zielserver **ebenfalls mit `/preRelease`** gestartet werden
> (`RevioSetup.exe /preRelease`). Andernfalls werden die Artefakte im Paket nicht gefunden.

### 10.2 Auf dem Zielserver installieren

1. `RevioSetup.exe` **und** `revioSetupArtifacts.zip` auf den Zielserver kopieren.
2. Setup als Administrator starten.
3. Auf der Startseite **„Offline-Installation"** anhaken und über **„Durchsuchen…"** das
   `revioSetupArtifacts.zip` auswählen.
4. Den Assistenten wie gewohnt durchlaufen. Beim Weiterschalten wird das Paket geprüft
   (Grösse und Prüfsumme je Datei); anschliessend werden alle Artefakte aus dem Paket statt
   aus dem Internet installiert.

---

## Standardwerte auf einen Blick

| Element | Standard |
|---|---|
| Installationsverzeichnis Backend | `C:\revio4backend` |
| Datenverzeichnis MongoDB | `C:\revio4backend\data\revio4` |
| HTTP-Port / HTTPS-Port | `27010` / `27011` |
| MongoDB-Port | `27040` |
| Backend-Dienst | `revioServer` |
| Datenbank-Dienst | `revio4Db` |
| SSL-Zertifikat (Subject) | `revioBackendServer.<Rechnername>` (Trusted Root, 15 Jahre) |
| Zentrale Ablage (RDS) | `C:\Deploy\revioClient` (Freigabe `Deploy`) |
| Client-Konfiguration | `revioClientConfig.zip` (im Backend-Installationsverzeichnis) |

---

*Version 1.0 · Stand: Juli 2026 · Sprache: Deutsch*
