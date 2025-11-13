# Öffentliche Bereitstellung von revio

## Einleitung

Diese Dokumentation beschreibt, wie revio öffentlich im Internet verfügbar gemacht wird – entweder direkt auf einem Server oder über einen Reverse Proxy.

**Ziel:** Den revio-Server sicher über eine öffentliche Domain (z.B. `https://revio.firma.ch`) bereitstellen, unabhängig vom eingesetzten Betriebssystem oder der Hosting-Technologie.

---

## Voraussetzungen

### Server & System
- Öffentlicher Server oder virtuelle Maschine mit Internetzugang
- Unterstütztes Betriebssystem (Windows oder Linux)
- Installierte Webserver- oder Hosting-Umgebung (z.B. IIS, NGINX oder Apache mit aktivierten WebSocket- und HTTPS-Funktionen)

### Netzwerk & Domain
- DNS-Eintrag (A-Record) auf die öffentliche IP-Adresse
- Gültiges SSL-/TLS-Zertifikat (z.B. Let's Encrypt oder firmeneigene CA)

### Firewall-Freigaben
- **Port 443 (HTTPS)** → produktiver Zugriff
- **Port 80 (HTTP)** → Let's-Encrypt-HTTP-01-Challenge

### Sicherheit
- Sicheres Benutzer- und Netzwerk-Konzept

---

## Vorgehensweise (Übersicht)

### 1. Server bereitstellen
Virtuelle Maschine oder physischer Host mit aktueller Systemkonfiguration und Sicherheitsupdates.

### 2. DNS konfigurieren
A-Record für `revio.firma.ch` auf die öffentliche IP-Adresse setzen und DNS-Propagation abwarten.

### 3. Webserver installieren und konfigurieren
- IIS, NGINX oder Apache einrichten
- Standard-Website konfigurieren
- Port 80 aktivieren und Testseite bereitstellen

### 4. Funktionstest ohne HTTPS
Zugriff über `http://revio.firma.ch` prüfen, um grundlegende Erreichbarkeit zu bestätigen.

### 5. HTTPS einrichten
- Let's Encrypt-Zertifikat über Port 80 ausstellen
- Automatische Erneuerung konfigurieren
- Webserver auf HTTPS umstellen und Bindings aktualisieren

### 6. Funktionstest mit HTTPS
Zugriff über `https://revio.firma.ch` prüfen und sicherstellen, dass Umleitungen (HTTP → HTTPS) funktionieren.

### 7. revio-Server installieren
revio-Server nach Anleitung für die jeweilige Plattform installieren.

### 8. Webserver-Weiterleitung konfigurieren
- Webserver oder Proxy so einrichten, dass Anfragen an den revio-Server weitergeleitet werden (z.B. Reverse-Proxy-Regeln, `proxy_pass`, ASP.NET Core Module, URL Rewrite)
- WebSockets und Zeitlimits korrekt konfigurieren
- Abschlusstest der Gesamtverbindung unter HTTPS durchführen

> **Hinweis:** Port 80 sollte ausschließlich für die Zertifikatsprüfung (ACME-Challenge) offen bleiben. Danach sollte ein automatischer Redirect auf HTTPS erfolgen.

---

## Vorteile

- ✅ Öffentliche und sichere Erreichbarkeit über HTTPS
- ✅ Flexible Betriebsmodelle (direkt oder hinter Proxy)
- ✅ Plattformunabhängig (Windows / Linux)
- ✅ Kompatibel mit bestehenden Sicherheits- und Zertifikatssystemen

## Nachteile

- ⚠️ Höhere Sicherheitsanforderungen durch öffentliche Bereitstellung
- ⚠️ Aufwand abhängig von Infrastruktur und Hosting-Modell

## Empfohlen für

Organisationen, die revio öffentlich bereitstellen möchten – egal ob als direkt erreichbarer Server oder über einen Reverse Proxy in einer bestehenden Infrastruktur.

---

## Beispiel: Öffentliche Bereitstellung unter Windows (IIS Out-of-Process)

### Schritt 1: IIS installieren

Für die Bereitstellung von revio unter Windows wird der Internet Information Services (IIS)-Webserver benötigt.

#### Vorgehen

1. **PowerShell als Administrator öffnen**

2. **IIS installieren:**
   ```powershell
   Install-WindowsFeature Web-Server, Web-WebSockets, Web-Mgmt-Tools
   ```

3. **IIS-Dienst überprüfen:**
   ```powershell
   Get-Service W3SVC
   ```

4. **Installation testen:**
   Öffne im Browser die öffentliche IP-Adresse:
   ```
   http://<deine-public-ip>
   ```

#### Troubleshooting

Wenn die Seite nicht erreichbar ist, prüfe folgende Punkte:

**Windows-Firewall blockiert Port 80/443:**
- Überprüfe die lokalen Firewallregeln
- Erlaube eingehenden Datenverkehr auf Port 80 und 443

**IIS-Dienst nicht gestartet:**
```powershell
Get-Service W3SVC
Start-Service W3SVC
```

**Netzwerkkonfiguration:**
- Stelle sicher, dass die öffentliche IP-Adresse korrekt zugewiesen ist
- Bei Cloud-Anbietern (Azure, AWS, Google Cloud): Prüfe die Inbound Security Rules bzw. Network Security Groups (NSG) für Port 80 und 443

---

### Schritt 2: IIS für die Domain konfigurieren

Damit IIS Anfragen an `http://revio.firma.ch` korrekt entgegennimmt, muss die Website mit dem entsprechenden Hostname konfiguriert werden.

#### Vorgehen

1. **IIS-Manager öffnen:** `inetmgr`

2. **Site konfigurieren:**
   - Wähle im linken Bereich den Servernamen
   - Öffne den Knoten "Sites"
   - Markiere die Standardwebsite
   - Klicke rechts auf "Bindings…"

3. **Binding bearbeiten:**
   - **Typ:** http
   - **IP-Adresse:** All Unassigned (oder spezifische Adresse)
   - **Port:** 80
   - **Hostname:** `revio.firma.ch`

4. **Speichern und bestätigen**

> **Hinweis:** Nach der Einrichtung von HTTPS über Let's Encrypt wird automatisch eine zweite Bindung für Port 443 erstellt.

---

### Schritt 3: Let's Encrypt-Zertifikat einrichten

Um die Verbindung zum Server abzusichern, wird ein SSL-/TLS-Zertifikat benötigt. Die empfohlene Methode ist die automatische Ausstellung über Let's Encrypt mit dem Tool **win-acme**.

#### Vorgehen

1. **win-acme herunterladen:**
   👉 [https://www.win-acme.com/](https://www.win-acme.com/)

2. **Entpacken:**
   ```
   C:\Tools\win-acme
   ```

3. **Als Administrator ausführen:**
   ```powershell
   cd C:\Tools\win-acme
   .\wacs.exe
   ```

4. **Interaktives Setup durchführen:**
   - IIS-Site-Namen wählen (z.B. `revio.firma.ch`)
   - Option zur automatischen Zertifikatserstellung und Bindung an IIS wählen
   - Automatische Verlängerung des Zertifikats aktivieren

Nach erfolgreicher Einrichtung ist der Server unter `https://revio.firma.ch` mit einem gültigen Let's-Encrypt-Zertifikat erreichbar.

#### Wichtige Hinweise

- **Port 80** muss für die Zertifikatserstellung erreichbar sein (HTTP-01-Challenge)
- win-acme richtet einen geplanten Task ein, der das Zertifikat automatisch alle 60 Tage überprüft und erneuert
- Der Ablauf kann je nach Version variieren – folge den Anweisungen im interaktiven Setup

---

### Schritt 4: revio Backend installieren

Nach der erfolgreichen IIS- und Domainkonfiguration kann das revio Backend installiert werden.

#### Vorgehen

**1. Verzeichnis anlegen:**
```powershell
mkdir C:\revio4Backend
cd C:\revio4Backend
```

**2. Installationsskript herunterladen und ausführen:**
```powershell
iwr https://reviofiles.blob.core.windows.net/revio4-installation-files/4.0.270.0/InstallRevioBackend.ps1 -OutFile InstallRevioBackend.ps1

.\InstallRevioBackend.ps1
```

**3. Standardinstallation durchführen:**

Fahre mit der Installation gemäss der Dokumentation fort:
- Datenbankinstallation durchführen
- Administrator-Benutzer und Passwort festlegen
- Initialbenutzer erfassen
- Lizenzinformationen eingeben
- Vorlagendatenbank aktualisieren
- Installation abschliessen

Nach Abschluss ist das revio Backend vollständig installiert und betriebsbereit.

---

### Schritt 5: IIS-Weiterleitung an das revio Backend konfigurieren

Damit IIS eingehende Anfragen an das revio Backend weiterleitet, muss die IIS-Site angepasst und das Backend im Out-of-Process-Modus betrieben werden.

#### Vorgehen

**1. revioServer-Dienst deaktivieren und entfernen:**

Der durch die Standardinstallation angelegte Windows-Dienst wird nicht benötigt.

```powershell
Stop-Service revioServer
sc delete revioServer
```

**2. .NET Hosting Bundle installieren:**

Damit IIS ASP.NET Core-Anwendungen ausführen kann, muss das .NET Hosting Bundle installiert werden.

👉 [Download .NET Hosting Bundle](https://dotnet.microsoft.com/en-us/download/dotnet/thank-you/runtime-aspnetcore-9.0.10-windows-hosting-bundle-installer)

> **Hinweis:** Der Link kann je nach Version variieren.

**3. web.config anpassen:**

Öffne im Verzeichnis `C:\revio4Backend` die Datei `web.config` und ändere den Hostingmodus:

**Vorher:**
```xml
<aspNetCore processPath=".\revio.Backend.SingleServer.exe"
            stdoutLogEnabled="false"
            stdoutLogFile=".\logs\stdout"
            hostingModel="inprocess" />
```

**Nachher:**
```xml
<aspNetCore processPath=".\revio.Backend.SingleServer.exe"
            stdoutLogEnabled="false"
            stdoutLogFile=".\logs\stdout"
            hostingModel="outofprocess" />
```

**4. IIS-Verzeichniszuordnung anpassen:**

1. IIS-Manager öffnen (`inetmgr`)
2. Website auswählen (z.B. "Default Web Site")
3. Rechts auf "Basic Settings…" klicken
4. **Physical path** ändern auf:
   ```
   C:\revio4Backend
   ```
5. Mit OK bestätigen

**5. IIS neu starten:**

```powershell
iisreset
```

Nach dem Neustart verarbeitet IIS alle Anfragen an `https://revio.firma.ch` und leitet sie korrekt an das revio Backend weiter.

---

## Abschluss

Nach Durchführung aller Schritte ist revio erfolgreich öffentlich unter `https://revio.firma.ch` erreichbar und bereit für den produktiven Einsatz.

Bei Fragen oder Problemen konsultiere die offizielle revio-Dokumentation oder kontaktiere den Support.