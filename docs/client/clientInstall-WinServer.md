# Installation und Konfiguration - revio 4 Client auf Windows Server 2019+

> **Hinweis:** Für Neuinstallationen empfehlen wir den **Setup-Assistenten** (`RevioSetup.exe`) – siehe [revio 4 Setup](../setup/reviosetup.md) und, für die Verteilung auf Terminal-Servern, [RDS-Verteilung per GPO](../setup/rds-client-verteilung-gpo.md). Der Assistent übernimmt Voraussetzungen, Zertifikatsimport, MSIX-Installation und `ServerList.json` automatisch. Diese Anleitung beschreibt die **manuelle Installation** für Sonderfälle und zum Verständnis der einzelnen Schritte.

## Inhaltsverzeichnis

1. [Über MSIX](#über-msix)
2. [Systemvoraussetzungen](#systemvoraussetzungen)
3. [Vorbereitende Schritte](#vorbereitende-schritte)
4. [Installation](#installation)
5. [Konfiguration](#konfiguration)

---

## Über MSIX

Der revio 4 Client wird als **MSIX-Paket** ausgeliefert – dem modernen, von Microsoft empfohlenen Installationsformat für Windows-Anwendungen (Nachfolger von MSI/AppX). MSIX-Pakete sind digital signierte Container, die rückstandsfrei installiert, sauber inkrementell aktualisiert und über IT-Verteilungssysteme wie Intune, SCCM oder GPO verteilt werden können.

Da unser MSIX mit einem öffentlich vertrauenswürdigen Zertifikat (DigiCert) signiert ist, sind weder Developer Mode noch zusätzliche Trust-Konfigurationen auf den Zielsystemen erforderlich.

**Weitere Informationen:** [Microsoft MSIX Overview](https://learn.microsoft.com/en-us/windows/msix/overview)

---

## Systemvoraussetzungen

### Unterstützte Betriebssysteme

- **Windows Server 2022 oder höher** (empfohlen)
- **Windows Server 2019** (mit zusätzlichen Vorbereitungsschritten möglich)
- Ältere Versionen werden nicht unterstützt

---

## Vorbereitende Schritte

Folgende Komponenten müssen vor der Installation einmalig eingerichtet werden:

### 1. Windows App Runtime installieren

Damit unsere WinUI 3-Anwendung korrekt funktioniert, muss die Windows App Runtime installiert sein.

#### Für Windows Server 2019

Installieren Sie die **WindowsAppRuntimeInstall-x64.exe**, da hier die Paket- und Installationsinfrastruktur für MSIX nicht standardmässig vorhanden ist.

**Download:**
- [Windows App SDK Downloads](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/downloads)
- [Direkter Download (v1.8)](https://aka.ms/windowsappsdk/1.8/latest/windowsappruntimeinstall-x64.exe)

#### Für Windows Server 2022 und neuer

Das MSIX-Paket **Microsoft.WindowsAppRuntime.1.8.msix** ist ausreichend, da dort die Voraussetzungen bereits gegeben sind. Diese Runtime-Version wird mit unserem Client-Setup mitgeliefert (Dependencies-Folder).

---

### 2. Fluent Icons Font installieren

Unsere Anwendung nutzt die Fluent UI Icons – das ist das offizielle Symbol-Designsystem von Microsoft für moderne Windows-Apps.

Diese Symbole stammen aus der **Segoe Fluent Icons**-Schriftart, die normalerweise ab Windows 10 21H2 oder Windows 11 automatisch im System enthalten ist.

#### Warum diese Schriftart erforderlich ist

Da Windows Server 2019 auf dem älteren Windows 10 1809-Kernel basiert, fehlt diese Schriftart dort standardmässig. WinUI 3 verwendet sie aber für viele Bedienelemente (Buttons, Menüs, Navigation, Symbolleisten).

**Fehlt sie, erscheinen entweder leere Rechtecke (□) oder falsche Zeichen anstelle der Icons.**

**Download:** [Segoe Fluent Icons](https://aka.ms/SegoeFluentIcons)

---

### 3. .NET 9 Runtime installieren

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

Das Client-Paket wird als ZIP-Archiv (`revioClient.zip`) ausgeliefert und enthält alle für die Installation benötigten Artefakte:

- `revioClient_<Version>_x64.msix` – das signierte Anwendungspaket
- `Dependencies\` – Windows App Runtime in allen Architekturen (x64, x86, arm64, win32)
- `UpdateRevioMsix.ps1` – PowerShell-Installer für Erstinstallation und Updates
- `Install.ps1` / `Add-AppDevPackage.ps1` – Microsoft-Standardskripte (für Sonderfälle)

### Option A – Manuelle Installation auf einem Einzelsystem

1. `revioClient.zip` auf den Zielserver kopieren und entpacken, z. B. nach:
   ```
   C:\Deploy\revioClient\
   ```

2. PowerShell als Administrator öffnen und das Installationsskript ausführen:
   ```powershell
   C:\Deploy\revioClient\UpdateRevioMsix.ps1
   ```

#### Was das Skript `UpdateRevioMsix.ps1` macht

Das Skript ist ein schlanker Wrapper um `Add-AppxPackage` und übernimmt die folgenden Schritte:

1. Es ermittelt aus dem eigenen Pfad den Speicherort des MSIX-Pakets sowie der Windows App Runtime (`Dependencies\x64\Microsoft.WindowsAppRuntime.1.8.msix`).
2. Es prüft, ob beide Dateien vorhanden sind, und bricht andernfalls mit einer aussagekräftigen Fehlermeldung ab.
3. Es installiert die Anwendung mit den Flags `-ForceApplicationShutdown` (laufende Instanzen werden beendet) und `-ForceUpdateFromAnyVersion` (Upgrade aus jeder bisher installierten Version, auch Downgrades, sofern Signatur und Paketname identisch sind).

Damit deckt dasselbe Skript sowohl die **Erstinstallation** als auch alle **Folge-Updates** ab. Die Zielversion kann optional per Parameter überschrieben werden:

```powershell
.\UpdateRevioMsix.ps1 -Version 4.0.357.0
```

---

### Option B – Zentrale Verteilung über ein Logon-Skript (empfohlen für RDS / Mehrbenutzer)

Da MSIX pro Benutzerprofil registriert wird, eignet sich die Kombination aus zentral abgelegtem Installationsskript und einem benutzerbezogenen Logon-Skript besonders gut für Terminalserver- und Mehrbenutzerumgebungen.

#### Konzept

1. Der Inhalt von `revioClient.zip` wird einmalig in eine zentrale, für alle relevanten Benutzer lesbare Freigabe entpackt – entweder lokal auf dem Terminalserver (z. B. `C:\Deploy\revioClient\`, freigegeben als `\\<SERVER>\Deploy\revioClient`) oder auf einem Fileserver.
2. Ein Logon-Skript (per GPO als **User Logon Script** verteilt) ruft bei jeder Anmeldung `UpdateRevioMsix.ps1` aus dieser Freigabe auf. `Add-AppxPackage` mit `-ForceUpdateFromAnyVersion` ist idempotent: ist die aktuelle Version bereits registriert, kehrt der Aufruf praktisch ohne Last zurück; ist eine neue Version vorhanden, wird automatisch aktualisiert.
3. Für ein Rollout einer neuen Client-Version muss lediglich der Inhalt der zentralen Freigabe durch das neue, entpackte `revioClient.zip` ersetzt werden. Alle Benutzer erhalten das Update bei ihrer nächsten Anmeldung – ohne weiteren administrativen Eingriff pro Benutzer oder Maschine.

#### Beispiel-Logon-Skript

```powershell
# revioClient-Logon.ps1 – als User Logon Script per GPO verteilen
$Installer = "\\<SERVER>\Deploy\revioClient\UpdateRevioMsix.ps1"

if (Test-Path $Installer) {
    & $Installer
}
```

#### Hinweise zur Berechtigung

- Die Freigabe muss für die anmeldenden Benutzer mindestens **Lesezugriff** auf alle Dateien (inkl. `Dependencies\`) erlauben.
- `Add-AppxPackage` registriert das Paket im Benutzerkontext und benötigt keine erhöhten Rechte – das Logon-Skript läuft also ohne UAC-Prompt.
- Die Schreibrechte auf die Freigabe bleiben den Administratoren vorbehalten, die das Update-Paket austauschen.

#### Alternative Verteilungswege

Dasselbe Skript lässt sich identisch über andere Verteilungsmechanismen ausführen, falls keine GPO-Logon-Skripte eingesetzt werden:

- **Microsoft Intune** – PowerShell Script Assignment im Benutzerkontext
- **System Center Configuration Manager (SCCM/MECM)** – Package/Program oder Application im User-Context
- **Aufgabenplanung** – Trigger „Bei Anmeldung jedes Benutzers"

---

### Option C – Verteilung über Intune / Endpoint Manager

Falls eine moderne Verwaltung eingesetzt wird:

1. `revioClient.zip` entpacken und die enthaltene MSIX-Datei (`revioClient_<Version>_x64.msix`) sowie die Runtime-Abhängigkeit aus `Dependencies\x64\` bereithalten.
2. Im **Intune Portal** unter **Apps** > **Add App** > **Line-of-business App** das MSIX-Paket hochladen.
3. Zielgruppen (Benutzer oder Geräte) zuweisen.
4. Intune verteilt und verwaltet das Paket automatisch.

---

## Konfiguration

### 1. Installation des Server-Zertifikates (revioServer SSL)

#### Warum dieser Schritt notwendig ist

Unsere Anwendung kommuniziert verschlüsselt mit dem revioServer über HTTPS. Damit diese Verbindung auf dem jeweiligen System als vertrauenswürdig gilt, muss das vom Server verwendete SSL-Zertifikat im lokalen Zertifikatsspeicher des Rechners hinterlegt werden.

Da es sich hierbei um ein selbstsigniertes Zertifikat handelt, erkennt Windows es standardmässig nicht automatisch als vertrauenswürdig. Daher muss das Zertifikat einmalig manuell importiert werden. 

**Gründe:**
- Selbstsignierte Zertifikate besitzen keine Signatur einer öffentlichen Zertifizierungsstelle (z. B. DigiCert, GlobalSign)
- Ohne Import würde Windows die Verbindung als unsicher markieren und die App könnte HTTPS-Verbindungen ablehnen
- Durch das Importieren wird dem System explizit mitgeteilt, dass das Zertifikat und somit der Server vertrauenswürdig sind

> **Hinweis:** Das Zertifikat und die Serverkonfiguration werden im finalen Schritt der revio4 Backendinstallation erstellt (siehe revio4 Installationshandbuch). Im generierten Zip-File (revioClientConfig.zip) kann das Zertifikat extrahiert werden.

#### Variante 1 – Grafisch (empfohlen für Administratoren mit GUI-Zugriff)

1. Kopieren Sie die Zertifikatsdatei `revioServer.crt` oder `revioServer.cer` auf den Zielserver
2. Doppelklicken Sie auf die Datei → **Zertifikat installieren**
3. Im Assistenten wählen Sie:
   - **Speicherort:** Lokaler Computer
   - **Zertifikatspeicher:** Vertrauenswürdige Stammzertifizierungsstellen
4. Schliessen Sie die Installation ab und bestätigen Sie
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

Damit sich die Anwendung korrekt mit dem gewünschten Server verbinden kann, muss die Datei `ServerList.json` im Benutzerprofil jedes Benutzers abgelegt werden. Sie enthält die Liste der verfügbaren Serverinstanzen und die jeweilige Standardverbindung.

#### Speicherort

```
%USERPROFILE%\revio\ServerList.json
```

Beim Start liest die Anwendung diese Datei automatisch aus, um die Serververbindung und Benutzerdaten zu initialisieren.

#### Datei aus `revioClientConfig.zip` aufbereiten

Bei der revio4 Backendinstallation wird das ZIP-File `revioClientConfig.zip` erzeugt (siehe revio4 Installationshandbuch). Darin liegt eine Datei `serverConfig.json` mit einem einzelnen Serverobjekt, z. B.:

```json
{"Title":"revio AG","Url":"https://revioServer:27011","IsDefault":true}
```

Die Anwendung erwartet jedoch eine Datei mit dem Namen **`ServerList.json`**, deren Inhalt ein **Array** von Servereinträgen ist und die zusätzlich die Felder `Type` und `LastLoginUser` enthält. Die Datei aus dem ZIP muss daher vor der Verteilung manuell angepasst werden:

1. `serverConfig.json` aus `revioClientConfig.zip` extrahieren.
2. Inhalt in ein Array umschliessen und die fehlenden Felder ergänzen.
3. Datei in `ServerList.json` umbenennen.

#### Ziel-Inhalt der `ServerList.json`

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

#### Verteilung an die Benutzerprofile

1. Zielordner anlegen, falls nicht vorhanden:
   ```powershell
   New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\revio"
   ```

2. Aufbereitete Datei kopieren:
   ```powershell
   Copy-Item "C:\Deploy\ServerList.json" "$env:USERPROFILE\revio\ServerList.json" -Force
   ```

> **Wichtig:** Jeder Benutzer, der die Anwendung verwendet, benötigt eine eigene Kopie dieser Datei in seinem Profilordner. In RDS-/Mehrbenutzerumgebungen empfiehlt sich die Kombination mit dem in Option B beschriebenen Logon-Skript.

---

## Support und weitere Informationen

Bei Fragen oder Problemen wenden Sie sich bitte an den technischen Support.

**Version:** 1.1  
**Letzte Aktualisierung:** Mai 2026
