# Installation und Konfiguration - revio 4 Client auf Windows Server 2019+

## Inhaltsverzeichnis

1. [Über MSIX](#über-msix)
2. [Systemvoraussetzungen](#systemvoraussetzungen)
3. [Vorbereitende Schritte](#vorbereitende-schritte)
4. [Installation](#installation)
5. [Konfiguration](#konfiguration)
6. [Updates](#updates)

---

## Über MSIX

### Was ist eine MSIX-Datei?

MSIX ist das moderne Installationsformat von Microsoft für Windows-Anwendungen. Es wurde entwickelt, um Software sicher, zuverlässig und sauber zu installieren und zu aktualisieren. Es ist der Nachfolger von MSI und AppX.

MSIX-Pakete enthalten alle benötigten Dateien, Abhängigkeiten und Konfigurationen in einem einzigen, digital signierten Container. Dadurch wird sichergestellt, dass:

- die Installation sicher ist (Windows prüft die Signatur)
- keine Systemdateien verändert werden
- Deinstallation rückstandsfrei erfolgt
- Updates automatisch und inkrementell verteilt werden können

**Weitere Informationen:** [Microsoft MSIX Overview](https://learn.microsoft.com/en-us/windows/msix/overview)

### Warum unsere App als MSIX verteilt wird

Unsere Anwendung ist eine moderne WinUI 3 App – also eine Anwendung, die auf dem neuesten Windows-App-Framework aufbaut. Dieses Framework nutzt das Windows App SDK, das von Microsoft für MSIX-Verteilung optimiert ist.

Wir verwenden MSIX, weil es:

1. die Installation vereinfacht – kein manuelles Setup, alles ist in einem Paket
2. höhere Sicherheit bietet – das Paket ist digital signiert und überprüfbar
3. automatische Updates ermöglicht – zukünftige Versionen können direkt installiert werden, ohne vorherige manuelle Deinstallation
4. kompatibel mit IT-Verteilungssystemen wie Intune oder Gruppenrichtlinien ist

---

## Systemvoraussetzungen

### Unterstützte Betriebssysteme

- **Windows Server 2022 oder höher** (empfohlen)
- **Windows Server 2019** (mit zusätzlichen Vorbereitungsschritten möglich)
- Ältere Versionen werden nicht unterstützt

---

## Vorbereitende Schritte

Folgende Komponenten müssen vor der Installation einmalig eingerichtet werden:

### 1. Developer Mode aktivieren

#### Warum der Entwicklermodus aktiviert werden muss

Unsere Anwendung wird als MSIX-Paket ausgeliefert. Dieses Format ist offiziell von Microsoft vorgesehen, erfordert aber eine Vertrauensquelle für die Installation.

Da die App nicht aus dem Microsoft Store stammt, muss Windows wissen, dass der Administrator die Installation ausdrücklich erlaubt. Der Developer Mode ist dafür der einfachste Weg:

- Er erlaubt die Installation signierter, aber nicht aus dem Store stammender MSIX-Pakete
- Ohne diesen Modus blockiert Windows die Installation standardmäßig aus Sicherheitsgründen

#### Was der Developer Mode nicht tut

- Er schaltet keinen Debug-Modus oder unsicheren Zustand frei
- Er verändert keine System- oder Netzwerksicherheitsrichtlinien
- Er ermöglicht nur die Installation von vertrauenswürdigen MSIX-Paketen, die digital signiert sind

**Das heißt:** Aktivieren des Developer Mode ist auf Servern eine rein technische Voraussetzung, keine Gefährdung der Systemintegrität.

#### Aktivierung

1. Öffnen Sie die **Einstellungen** (Windows + I)
2. Navigieren Sie zu **Update & Sicherheit** > **Für Entwickler**
3. Aktivieren Sie den **Entwicklermodus**

---

### 2. Windows App Runtime installieren

Damit unsere WinUI 3-Anwendung korrekt funktioniert, muss die Windows App Runtime installiert sein.

#### Für Windows Server 2019

Installieren Sie die **WindowsAppRuntimeInstall-x64.exe**, da hier die Paket- und Installationsinfrastruktur für MSIX nicht standardmäßig vorhanden ist.

**Download:**
- [Windows App SDK Downloads](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/downloads)
- [Direkter Download (v1.8)](https://aka.ms/windowsappsdk/1.8/latest/windowsappruntimeinstall-x64.exe)

#### Für Windows Server 2022 und neuer

Das MSIX-Paket **Microsoft.WindowsAppRuntime.1.8.msix** ist ausreichend, da dort die Voraussetzungen bereits gegeben sind. Diese Runtime-Version wird mit unserem Client-Setup mitgeliefert (Dependencies-Folder).

---

### 3. Fluent Icons Font installieren

Unsere Anwendung nutzt die Fluent UI Icons – das ist das offizielle Symbol-Designsystem von Microsoft für moderne Windows-Apps.

Diese Symbole stammen aus der **Segoe Fluent Icons**-Schriftart, die normalerweise ab Windows 10 21H2 oder Windows 11 automatisch im System enthalten ist.

#### Warum diese Schriftart erforderlich ist

Da Windows Server 2019 auf dem älteren Windows 10 1809-Kernel basiert, fehlt diese Schriftart dort standardmäßig. WinUI 3 verwendet sie aber für viele Bedienelemente (Buttons, Menüs, Navigation, Symbolleisten).

**Fehlt sie, erscheinen entweder leere Rechtecke (□) oder falsche Zeichen anstelle der Icons.**

**Download:** [Segoe Fluent Icons](https://aka.ms/SegoeFluentIcons)

---

### 4. .NET 9 Runtime installieren

Unsere Anwendung basiert auf .NET 9. Damit sie ausgeführt werden kann, müssen die passenden Microsoft-Laufzeitumgebungen (ASP.NET Core und Windows Desktop Runtime) installiert werden.

Diese stellen sicher, dass die Anwendung sowohl ihre Oberfläche (WinUI 3) als auch interne Web-Komponenten korrekt ausführen kann.

**Windows Server-Systeme bringen diese Laufzeiten nicht von Haus aus mit – sie müssen daher einmalig manuell installiert werden.**

#### Benötigte Komponenten

1. **ASP.NET Core Runtime 9.0.9**
   - [Direkter Download](https://builds.dotnet.microsoft.com/dotnet/aspnetcore/Runtime/9.0.9/aspnetcore-runtime-9.0.9-win-x64.exe)

2. **Windows Desktop Runtime 9.0.9**
   - [Direkter Download](https://builds.dotnet.microsoft.com/dotnet/WindowsDesktop/9.0.9/windowsdesktop-runtime-9.0.9-win-x64.exe)

**Weitere Informationen:** [.NET 9 Downloads](https://dotnet.microsoft.com/en-us/download/dotnet/9.0)

---

## Installation

### Option A – Direkte Installation

1. Kopieren Sie die MSIX-Datei auf den Server, z. B.:
   ```
   C:\Deploy\UnsereApp.msix
   ```

2. **Installation per Rechtsklick:**
   - Rechtsklick auf die Datei → **Installieren**

3. **Installation per PowerShell:**
   ```powershell
   Add-AppxPackage -Path "C:\Deploy\UnsereApp.msix"
   ```
   > Hinweis: Adminrechte sind nicht zwingend erforderlich

---

### Option B – Zentrale Verteilung per PowerShell-Skript

Für Mehrbenutzer- oder RDS-Umgebungen kann die Installation automatisiert werden:

```powershell
# Beispielskript für zentrale Installation pro Benutzer
$PackagePath = "\\Server\Share\UnsereApp.msix"
Add-AppxPackage -Path $PackagePath
```

Das Skript kann ausgeführt werden:
- als Login-Skript (GPO User Script)
- über System Center Configuration Manager (SCCM)
- oder per Intune PowerShell Deployment

---

### Option C – Verteilung über Intune / Endpoint Manager

Falls eine moderne Verwaltung eingesetzt wird:

1. Öffnen Sie das **Intune Portal**
2. Navigieren Sie zu **Apps** > **Add App** > **Line-of-business App**
3. Laden Sie die MSIX-Datei hoch
4. Weisen Sie Zielgruppen (Benutzer / Geräte) zu
5. Intune verteilt und verwaltet das Paket automatisch

---

### Option D – Terminalserver-Umgebung (RDS)

MSIX wird benutzergebunden installiert.

**Empfehlung:** Installation über Login-Skript, sodass die App bei der ersten Anmeldung jedes Benutzers automatisch eingerichtet wird.

---

## Konfiguration

### 1. Installation des Server-Zertifikates (revioServer SSL)

#### Warum dieser Schritt notwendig ist

Unsere Anwendung kommuniziert verschlüsselt mit dem revioServer über HTTPS. Damit diese Verbindung auf dem jeweiligen System als vertrauenswürdig gilt, muss das vom Server verwendete SSL-Zertifikat im lokalen Zertifikatsspeicher des Rechners hinterlegt werden.

Da es sich hierbei um ein selbstsigniertes Zertifikat handelt, erkennt Windows es standardmäßig nicht automatisch als vertrauenswürdig. Daher muss das Zertifikat einmalig manuell importiert werden.

**Gründe:**
- Selbstsignierte Zertifikate besitzen keine Signatur einer öffentlichen Zertifizierungsstelle (z. B. DigiCert, GlobalSign)
- Ohne Import würde Windows die Verbindung als unsicher markieren und die App könnte HTTPS-Verbindungen ablehnen
- Durch das Importieren wird dem System explizit mitgeteilt, dass das Zertifikat und somit der Server vertrauenswürdig sind

#### Variante 1 – Grafisch (empfohlen für Administratoren mit GUI-Zugriff)

1. Kopieren Sie die Zertifikatsdatei `revioServer.crt` oder `revioServer.cer` auf den Zielserver
2. Doppelklicken Sie auf die Datei → **Zertifikat installieren**
3. Im Assistenten wählen Sie:
   - **Speicherort:** Lokaler Computer
   - **Zertifikatspeicher:** Vertrauenswürdige Stammzertifizierungsstellen
4. Schließen Sie die Installation ab und bestätigen Sie
5. Starten Sie die Anwendung oder ggf. den Server neu

#### Variante 2 – Per PowerShell (für automatisierte Deployment-Skripte)

```powershell
Import-Certificate -FilePath "C:\Deploy\revioServer.cer" -CertStoreLocation "Cert:\LocalMachine\Root"
```

> **Hinweis:** Erfordert Administratorrechte

Nach dem Import steht das Zertifikat systemweit als vertrauenswürdig zur Verfügung.

#### Wichtige Hinweise

- Der Import muss einmalig pro Server erfolgen
- Das Zertifikat sollte aktuell und gültig sein (Ablaufdatum prüfen)
- Bei Änderungen am Serverzertifikat (z. B. Erneuerung) muss der Import erneut durchgeführt werden

---

### 2. Konfigurationsdatei für Serververbindungen

Damit sich die Anwendung korrekt mit dem gewünschten Server verbinden kann, muss die Datei `ServerList.json` im Benutzerprofil abgelegt werden.

Diese Datei enthält die Liste der verfügbaren Serverinstanzen und die jeweilige Standardverbindung.

#### Speicherort

Die Datei muss sich im folgenden Pfad befinden:

```
%USERPROFILE%\revio\ServerList.json
```

Beim Start liest die Anwendung diese Datei automatisch aus, um die Serververbindung und Benutzerdaten zu initialisieren.

#### Beispielinhalt

```json
[
  {
    "Title": "revio AG",
    "Type": 1,
    "Url": "https://revioServer:27011",
    "IsDefault": true,
    "LastLoginUser": "info@revio.ch"
  }
]
```

#### Beschreibung der Felder

| Feld | Bedeutung |
|------|-----------|
| `Title` | Anzeigename des Servers in der Anwendung |
| `Type` | Verbindungstyp (z. B. 1 = Produktionsserver, 2 = Testumgebung) |
| `Url` | HTTPS-URL des Serverendpunkts |
| `IsDefault` | Gibt an, welcher Server beim Start automatisch ausgewählt wird |
| `LastLoginUser` | Benutzername oder E-Mail des zuletzt angemeldeten Benutzers |

#### Kopieren der Datei

1. Erstellen Sie den Ordner, falls nicht vorhanden:
   ```powershell
   New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\revio"
   ```

2. Kopieren Sie die Datei:
   ```powershell
   Copy-Item "C:\Deploy\ServerList.json" "$env:USERPROFILE\revio\ServerList.json" -Force
   ```

> **Wichtig:** Jeder Benutzer, der die Anwendung verwendet, benötigt eine eigene Kopie dieser Datei in seinem Profilordner.

---

## Updates

Neue Versionen können mit derselben Methode verteilt werden.

MSIX erkennt automatisch bestehende Installationen und führt eine saubere, inkrementelle Aktualisierung durch.

**Kein Deinstallieren erforderlich.**

### Update per PowerShell

```powershell
Add-AppxPackage -Path "C:\Deploy\UnsereApp_v2.msix" -ForceUpdateFromAnyVersion
```

Das System aktualisiert die Anwendung automatisch auf die neue Version, ohne dass Benutzerdaten oder Konfigurationen verloren gehen.

---

## Support und weitere Informationen

Bei Fragen oder Problemen wenden Sie sich bitte an den technischen Support.

**Version:** 1.0  
**Letzte Aktualisierung:** November 2025
