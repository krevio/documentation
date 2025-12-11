# revio Migration Tool - Datenextraktion

Das **revio Migration Tool** (`revio.migrator.exe`) extrahiert Revisionsdaten aus einer Legacy-revio MongoDB-Datenbank und erstellt eine ZIP-Exportdatei zur Migration nach revio 4.

## Migrationsablauf

Die Migration von revio Legacy nach revio 4 erfolgt in zwei Schritten:

1. **Datenextraktion (dieses Tool):** Das Migration Tool extrahiert alle relevanten Daten für das gewählte Umstellungsjahr aus der Legacy-Datenbank und erstellt eine ZIP-Exportdatei.

2. **Import in revio 4:** Die erstellte ZIP-Datei wird anschliessend in revio 4 eingelesen. Siehe dazu die separate Dokumentation zum Datenimport.

> **Wichtig:** Es kann nur **ein Jahr** nach revio 4 migriert werden. Wählen Sie das Jahr, ab dem Sie mit revio 4 arbeiten möchten (z.B. 2025 oder 2026).

### Was wird migriert?

Da revio 4 viele Bereiche neu und besser umsetzt, können nicht alle Daten 1:1 übernommen werden.

#### Mandanten und Akten

- **Alle Mandanten** werden vollständig übernommen
- **Alle Akten** für das gewählte Extraktionsjahr werden übernommen

#### Jahresrechnung

- **Konten** werden übernommen
- **Spiegelgruppen** und **Umbuchungen** werden erstellt
- **Stille Reserven** werden übernommen (müssen im revio 4 den Bilanz- und Erfolgsgruppen zugeordnet werden)
- **Bilanz-, ER- und Geldflussstruktur** werden *nicht* übernommen

> **Hinweis zur Jahresrechnung:** Im revio 4 gibt es nur noch eine Jahresrechnung pro Mandant (nicht mehr pro Akte). Falls im revio 3 mehrere Aktentypen mit unterschiedlichen Jahresrechnungen pro Mandant existieren, muss beim Import festgelegt werden, welcher Aktentyp als Jahresrechnungsbasis dient.

#### Prüfungsdokumentation

Folgende Bereiche werden mit allen zugehörigen Anhängen übernommen:

- **Allgemeine Prüfungshandlungen**
- **Notizen**
- **Dauerakten**

#### Sonstige Anhänge

Alle anderen Anhänge, die nicht in die oben genannten Bereiche fallen (z.B. Checklisten, Spezial-Prüfgebiete), werden in der Akte im Bereich **Belege** zur Wiederverwendung bereitgestellt.

#### Archiv

Alle **archivierten PDF-Akten** werden ins revio 4 Archiv übertragen.

### Was wird NICHT migriert?

| Bereich | Grund | Aktion erforderlich |
|---------|-------|---------------------|
| **Mitarbeiter** | Login in revio 4 ist neu E-Mail-basiert statt Visum-basiert | Mitarbeiter manuell in revio 4 anlegen. **Wichtig:** Dasselbe Visum wie in revio 3 verwenden! |
| **Bilanz-/ER-/Geldflussstruktur** | Neue auf Kontenklassen basierte Strukturlogik in revio 4 | Vordefinierte Standard-Strukturen vom revio 4 verwenden. Punktuell anpassen |
| **Checklisten** | Umstellung auf Formulareingabe statt Excel-Style | Neue Formulare in revio 4 verwenden |
| **Prüfungsplanung** | Komplett überarbeitet in revio 4 | Neue Prüfungsplanung in revio 4 verwenden |
| **Funktionsprüfungen** | Komplett überarbeitet in revio 4 | Neues Kontrollsystem in revio 4 verwenden |
| **Spezialprüfungen (SA-CH 2022)** | Komplett überarbeitet in revio 4 | Besondere Prüfgebiete in revio 4 verwenden |

## Voraussetzungen

- Das Tool muss auf dem **gleichen Server** ausgeführt werden, auf dem die MongoDB-Datenbank und der revio-Service laufen
- Der Dienst `revioOnlineDb` muss laufen (MongoDB, Port 27024)
- Der Dienst `revioOnlineServer` muss laufen (REST-API, Port 27029)
- Ausreichend Festplattenspeicher für temporäre Dateien und die Export-ZIP-Datei

## Kommandozeilenparameter

| Parameter | Beschreibung | Standardwert |
|-----------|--------------|--------------|
| `--db` | MongoDB Connection String | `mongodb://localhost:27024` |
| `--year` | Revisionsjahr für die Extraktion | `2025` |
| `--parallelism` | Maximale Anzahl paralleler Tasks | Anzahl CPU-Kerne |
| `--apiurl` | URL des revio REST-API Services | `http://localhost:27029` |
| `--output` | Ausgabeverzeichnis für die ZIP-Datei | `.` (aktuelles Verzeichnis) |
| `--temp` | Verzeichnis für temporäre Dateien | System-Temp-Verzeichnis |
| `--tlsMin` | Minimale TLS-Version (10/11/12) | `12` |

## Verwendung

### Grundlegende Verwendung

```cmd
revio.migrator.exe --year 2025
```

### Beispiel mit Connection String aus Konfiguration

```cmd
revio.migrator.exe --db "<CONNECTION_STRING>" --year 2025
```

> **Hinweis:** Den Connection String finden Sie in der Datei `revio.Server.exe.Config` im revio-Server-Verzeichnis (standardmässig `C:\Program Files (x86)\revio\server`).

### Beispiel mit allen Parametern

```cmd
revio.migrator.exe ^
  --db "<CONNECTION_STRING>" ^
  --year 2025 ^
  --parallelism 4 ^
  --apiurl "http://localhost:27029" ^
  --output "C:\Export" ^
  --temp "D:\Temp"
```

## Detaillierte Parameterbeschreibung

### --db (Datenbank-Verbindung)

Der MongoDB Connection String für die Verbindung zur Legacy-revio-Datenbank.

#### Connection String finden

Den korrekten Connection String finden Sie in der revio-Server-Konfiguration:

**Datei:** `revio.Server.exe.Config`
**Verzeichnis:** `C:\Program Files (x86)\revio\server` (Standardpfad)

Öffnen Sie die Datei und suchen Sie nach dem Abschnitt `<connectionStrings>`. Der Connection String befindet sich im Eintrag `revioDBConnection`:

```xml
<connectionStrings>
  <add name="revioDBConnection" connectionString="mongodb://...@localhost:27024/revioDB"/>
</connectionStrings>
```

Kopieren Sie den Wert aus dem Attribut `connectionString`.

#### Fallback ohne Authentifizierung

Falls in Ihrer Umgebung keine MongoDB-Authentifizierung konfiguriert ist:

```
mongodb://localhost:27024
```

**Beispiele:**
```cmd
# Mit Connection String aus der Konfigurationsdatei
revio.migrator.exe --db "<CONNECTION_STRING_AUS_CONFIG>" --year 2025

# Ohne Authentifizierung (falls keine Auth konfiguriert)
revio.migrator.exe --db "mongodb://localhost:27024" --year 2025
```

### --year (Revisionsjahr)

Das Jahr, ab dem Sie mit revio 4 arbeiten möchten.

**Beispiele:**
```cmd
# Umstellung ab 2025
revio.migrator.exe --year 2025

# Umstellung ab 2026
revio.migrator.exe --year 2026
```

### --parallelism (Parallelität)

Steuert die maximale Anzahl gleichzeitig verarbeiteter Mandanten. Ein höherer Wert beschleunigt die Verarbeitung, benötigt aber mehr Arbeitsspeicher.

**Empfehlungen:**
- **Standard:** Anzahl der CPU-Kerne (automatisch)
- **Wenig RAM:** `2` oder `4`
- **Viel RAM & schnelle Festplatte:** `8` oder höher

**Beispiele:**
```cmd
# Langsamer, aber speicherschonend
revio.migrator.exe --year 2025 --parallelism 2

# Schneller bei ausreichend Ressourcen
revio.migrator.exe --year 2025 --parallelism 8
```

### --apiurl (REST-API URL)

URL des revio-Service für zusätzliche Datenabfragen (Anhang-Details, etc.).

**Standard:**
```
http://localhost:27029
```

> **Wichtig:** Der Standardwert `http://localhost:27029` funktioniert in den meisten Installationen **nicht**, da der revio-Server typischerweise auf die öffentliche IP-Adresse des Servers hört, nicht auf localhost.

#### API-URL finden

Die korrekte API-URL finden Sie in der revio-Server-Konfiguration:

**Datei:** `revio.Server.exe.Config`
**Verzeichnis:** `C:\Program Files (x86)\revio\server` (Standardpfad)

Öffnen Sie die Datei und suchen Sie nach dem Eintrag `revioServer` im Abschnitt `<connectionStrings>`:

```xml
<connectionStrings>
  <add name="revioDBConnection" connectionString="mongodb://...@localhost:27024/revioDB"/>
  <add name="revioServer" connectionString="http://192.168.0.205:27029"/>
</connectionStrings>
```

Verwenden Sie den Wert aus dem `revioServer`-Eintrag als `--apiurl` Parameter.

**Beispiele:**
```cmd
# Mit IP-Adresse aus der Konfiguration
revio.migrator.exe --year 2025 --apiurl "http://192.168.0.205:27029"

# Anderer Port
revio.migrator.exe --year 2025 --apiurl "http://192.168.0.205:8080"
```

### --output (Ausgabeverzeichnis)

Verzeichnis, in dem die finale ZIP-Exportdatei erstellt wird.

**Beispiele:**
```cmd
# Export ins aktuelle Verzeichnis
revio.migrator.exe --year 2025

# Export in ein bestimmtes Verzeichnis
revio.migrator.exe --year 2025 --output "C:\revio\Exports"

# Export auf ein Netzlaufwerk
revio.migrator.exe --year 2025 --output "\\fileserver\exports"
```

### --temp (Temporäres Verzeichnis)

Verzeichnis für temporäre Dateien während der Extraktion.

> **Wichtig:** Die Extraktion erstellt temporäre JSON-Dateien und lädt Anhänge herunter. Bei grossen Datenmengen kann dies mehrere Gigabyte Speicherplatz benötigen!

**Beispiele:**
```cmd
# Alternatives Temp-Verzeichnis bei wenig Speicherplatz auf C:
revio.migrator.exe --year 2025 --temp "D:\Temp"

# Temp auf schneller SSD
revio.migrator.exe --year 2025 --temp "E:\FastTemp"
```

### --tlsMin (TLS-Version)

Minimale TLS-Version für verschlüsselte Verbindungen. Nur relevant bei SSL-Verbindungen.

| Wert | Erlaubte Protokolle |
|------|---------------------|
| `12` | TLS 1.2 (Standard, sicher) |
| `11` | TLS 1.1 + TLS 1.2 |
| `10` | TLS 1.0 + TLS 1.1 + TLS 1.2 |

**Beispiel:**
```cmd
# Ältere TLS-Versionen erlauben (nur bei Kompatibilitätsproblemen)
revio.migrator.exe --year 2025 --tlsMin 10
```

## Typische Szenarien

### Szenario 1: Standard-Migration

```cmd
revio.migrator.exe ^
  --db "<CONNECTION_STRING>" ^
  --year 2025
```

### Szenario 2: Wenig Speicherplatz auf C:

Wenn das System-Temp-Verzeichnis (normalerweise auf C:) wenig freien Speicherplatz hat:

```cmd
revio.migrator.exe ^
  --db "<CONNECTION_STRING>" ^
  --year 2025 ^
  --temp "D:\MigrationTemp" ^
  --output "D:\MigrationExport"
```

### Szenario 3: Langsamer Server

Bei einem Server mit wenig RAM oder CPU-Leistung:

```cmd
revio.migrator.exe ^
  --db "<CONNECTION_STRING>" ^
  --year 2025 ^
  --parallelism 2
```

## Ausgabe

Das Tool erstellt eine ZIP-Datei im Format:

```
revio_export_YYYY_YYYYMMDD_HHMMSS.zip
```

**Beispiel:** `revio_export_2025_20251211_143022.zip`

Die ZIP-Datei enthält:
- JSON-Dateien mit Mandanten-, Dossier- und Finanzdaten
- Alle zugehörigen Anhänge aus GridFS

## Fehlerbehebung

### Verbindungsfehler zur Datenbank

```
Error: Unable to connect to server
```

**Lösungen:**
1. Prüfen Sie, ob der Datenbank-Dienst läuft: `sc query revioOnlineDb`
2. Testen Sie die Verbindung ohne Authentifizierung: `--db "mongodb://localhost:27024"`
3. Prüfen Sie Firewall-Einstellungen

### API-Verbindungsfehler

```
Error: Connection refused to http://localhost:27029
```

**Lösungen:**
1. Prüfen Sie, ob der API-Dienst läuft: `sc query revioOnlineServer`
2. Kontrollieren Sie den Port in der revio-Konfiguration

### Speicherplatz-Fehler

```
Error: Not enough disk space
```

**Lösung:** Verwenden Sie ein anderes Temp-Verzeichnis auf einem Laufwerk mit ausreichend Speicherplatz:

```cmd
revio.migrator.exe --year 2025 --temp "D:\Temp" --output "D:\Export"
```

### Abbruch mit Ctrl+C

Sie können die Migration jederzeit mit `Ctrl+C` abbrechen. Temporäre Dateien werden automatisch aufgeräumt.

## Systemanforderungen

- Windows 10/11 oder Windows Server 2016+
- Mindestens 4 GB RAM (8 GB empfohlen)
- Freier Festplattenspeicher: ca. 2-3x die Grösse der Anhänge in der Datenbank

> **Hinweis:** Das Tool wird als eigenständige Einzeldatei ausgeliefert. Eine separate .NET-Installation ist nicht erforderlich.
