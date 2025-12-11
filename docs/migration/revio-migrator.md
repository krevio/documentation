# Revio Migration Tool - Benutzerhandbuch

Das **Revio Migration Tool** (`revio.migrator.exe`) extrahiert Revisionsdaten aus einer Legacy-Revio MongoDB-Datenbank und erstellt eine ZIP-Exportdatei zur Migration nach Revio 4.

## Voraussetzungen

- Das Tool muss auf dem **gleichen Server** ausgeführt werden, auf dem die MongoDB-Datenbank und der Revio-Service laufen
- Der Dienst `revioOnlineDb` muss laufen (MongoDB, Port 27024)
- Der Dienst `revioOnlineServer` muss laufen (REST-API, Port 27029)
- Ausreichend Festplattenspeicher für temporäre Dateien und die Export-ZIP-Datei

## Kommandozeilenparameter

| Parameter | Beschreibung | Standardwert |
|-----------|--------------|--------------|
| `--db` | MongoDB Connection String | `mongodb://localhost:27024` |
| `--year` | Revisionsjahr für die Extraktion | `2025` |
| `--parallelism` | Maximale Anzahl paralleler Tasks | Anzahl CPU-Kerne |
| `--apiurl` | URL des Revio REST-API Services | `http://localhost:27029` |
| `--output` | Ausgabeverzeichnis für die ZIP-Datei | `.` (aktuelles Verzeichnis) |
| `--temp` | Verzeichnis für temporäre Dateien | System-Temp-Verzeichnis |
| `--tlsMin` | Minimale TLS-Version (10/11/12) | `12` |

## Verwendung

### Grundlegende Verwendung

```cmd
revio.migrator.exe --year 2024
```

### Beispiel mit Connection String aus Konfiguration

```cmd
revio.migrator.exe --db "<CONNECTION_STRING>" --year 2024
```

> **Hinweis:** Den Connection String finden Sie in der Datei `revio.Server.exe.Config` im Revio-Server-Verzeichnis (standardmässig `C:\Program Files (x86)\revio\server`).

### Beispiel mit allen Parametern

```cmd
revio.migrator.exe ^
  --db "<CONNECTION_STRING>" ^
  --year 2024 ^
  --parallelism 4 ^
  --apiurl "http://localhost:27029" ^
  --output "C:\Export" ^
  --temp "D:\Temp"
```

## Detaillierte Parameterbeschreibung

### --db (Datenbank-Verbindung)

Der MongoDB Connection String für die Verbindung zur Legacy-Revio-Datenbank.

#### Connection String finden

Den korrekten Connection String finden Sie in der Revio-Server-Konfiguration:

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
revio.migrator.exe --db "<CONNECTION_STRING_AUS_CONFIG>" --year 2024

# Ohne Authentifizierung (falls keine Auth konfiguriert)
revio.migrator.exe --db "mongodb://localhost:27024" --year 2024
```

### --year (Revisionsjahr)

Das Jahr, für das die Revisionsdaten extrahiert werden sollen.

**Beispiele:**
```cmd
# Daten für 2024 extrahieren
revio.migrator.exe --year 2024

# Daten für 2023 extrahieren
revio.migrator.exe --year 2023
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
revio.migrator.exe --year 2024 --parallelism 2

# Schneller bei ausreichend Ressourcen
revio.migrator.exe --year 2024 --parallelism 8
```

### --apiurl (REST-API URL)

URL des Revio-Service für zusätzliche Datenabfragen (Anhang-Details, etc.).

**Standard:**
```
http://localhost:27029
```

**Beispiel:**
```cmd
# Anderer Port
revio.migrator.exe --year 2024 --apiurl "http://localhost:8080"
```

### --output (Ausgabeverzeichnis)

Verzeichnis, in dem die finale ZIP-Exportdatei erstellt wird.

**Beispiele:**
```cmd
# Export ins aktuelle Verzeichnis
revio.migrator.exe --year 2024

# Export in ein bestimmtes Verzeichnis
revio.migrator.exe --year 2024 --output "C:\Revio\Exports"

# Export auf ein Netzlaufwerk
revio.migrator.exe --year 2024 --output "\\fileserver\exports"
```

### --temp (Temporäres Verzeichnis)

Verzeichnis für temporäre Dateien während der Extraktion.

> **Wichtig:** Die Extraktion erstellt temporäre JSON-Dateien und lädt Anhänge herunter. Bei grossen Datenmengen kann dies mehrere Gigabyte Speicherplatz benötigen!

**Beispiele:**
```cmd
# Alternatives Temp-Verzeichnis bei wenig Speicherplatz auf C:
revio.migrator.exe --year 2024 --temp "D:\Temp"

# Temp auf schneller SSD
revio.migrator.exe --year 2024 --temp "E:\FastTemp"
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
revio.migrator.exe --year 2024 --tlsMin 10
```

## Typische Szenarien

### Szenario 1: Standard-Migration

```cmd
revio.migrator.exe ^
  --db "<CONNECTION_STRING>" ^
  --year 2024
```

### Szenario 2: Wenig Speicherplatz auf C:

Wenn das System-Temp-Verzeichnis (normalerweise auf C:) wenig freien Speicherplatz hat:

```cmd
revio.migrator.exe ^
  --db "<CONNECTION_STRING>" ^
  --year 2024 ^
  --temp "D:\MigrationTemp" ^
  --output "D:\MigrationExport"
```

### Szenario 3: Langsamer Server

Bei einem Server mit wenig RAM oder CPU-Leistung:

```cmd
revio.migrator.exe ^
  --db "<CONNECTION_STRING>" ^
  --year 2024 ^
  --parallelism 2
```

### Szenario 4: Mehrere Jahre migrieren

Führen Sie das Tool mehrfach aus - einmal pro Jahr:

```cmd
revio.migrator.exe --db "<CONNECTION_STRING>" --year 2022
revio.migrator.exe --db "<CONNECTION_STRING>" --year 2023
revio.migrator.exe --db "<CONNECTION_STRING>" --year 2024
```

## Ausgabe

Das Tool erstellt eine ZIP-Datei im Format:

```
revio_export_YYYY_YYYYMMDD_HHMMSS.zip
```

**Beispiel:** `revio_export_2024_20241211_143022.zip`

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
2. Kontrollieren Sie den Port in der Revio-Konfiguration

### Speicherplatz-Fehler

```
Error: Not enough disk space
```

**Lösung:** Verwenden Sie ein anderes Temp-Verzeichnis auf einem Laufwerk mit ausreichend Speicherplatz:

```cmd
revio.migrator.exe --year 2024 --temp "D:\Temp" --output "D:\Export"
```

### Abbruch mit Ctrl+C

Sie können die Migration jederzeit mit `Ctrl+C` abbrechen. Temporäre Dateien werden automatisch aufgeräumt.

## Systemanforderungen

- Windows 10/11 oder Windows Server 2016+
- Mindestens 4 GB RAM (8 GB empfohlen)
- Freier Festplattenspeicher: ca. 2-3x die Grösse der Anhänge in der Datenbank

> **Hinweis:** Das Tool wird als eigenständige Einzeldatei ausgeliefert. Eine separate .NET-Installation ist nicht erforderlich.
