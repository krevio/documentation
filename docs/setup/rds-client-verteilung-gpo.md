# revio 4 Client – Verteilung auf Terminal-Servern (RDS) per Gruppenrichtlinie (GPO)

Diese Anleitung beschreibt, wie der revio 4 Client auf einem Remotedesktop-Sitzungshost
(RDS / Terminal-Server) für **alle Benutzer** bereitgestellt wird. Der Client wird als
MSIX-Paket **pro Benutzer bei der Anmeldung** installiert – ausgelöst durch ein
Anmeldeskript, das per **Gruppenrichtlinie (GPO)** verteilt wird.

> **Kurzfassung:** Das revio 4 Setup legt eine zentrale Ablage mit dem Client-Paket und
> einem Anmeldeskript an. Die Gruppenrichtlinie sorgt dafür, dass dieses Skript bei jeder
> Benutzeranmeldung läuft und den Client (und auf Server 2019 die benötigte Laufzeit) im
> Benutzerkontext installiert.

> **Warum ein Anmeldeskript und keine „einmalige Installation für alle"?** MSIX-Pakete werden
> **grundsätzlich pro Benutzer registriert** – es gibt keinen maschinenweiten Installationszustand
> wie bei klassischem MSI. Deshalb ist das Anmeldeskript auf jedem Betriebssystem der
> zuverlässigste Weg. Eine Übersicht aller Verteilungsmethoden (inkl. maschinenweitem
> Provisioning) und ihrer Grenzen finden Sie in [Abschnitt 9](#9-weitere-verteilungsmethoden-übersicht).

---

## 1. Voraussetzungen

| Voraussetzung | Details |
|---|---|
| **RD-Sitzungshost-Rolle** | Für den Mehrbenutzerbetrieb muss die Rolle **Remotedesktop-Sitzungshost** installiert und lizenziert sein (RDS-CALs). Ohne diese Rolle erlaubt Windows nur ~2 administrative Sitzungen. Prüfen: `Get-WindowsFeature RDS-RD-Server`. |
| **Domänenmitgliedschaft** | Für die GPO-Verteilung muss der Server Mitglied einer Domäne sein. Prüfen: `(Get-CimInstance Win32_ComputerSystem).PartOfDomain` → `True`. (Für einen einzelnen, eigenständigen Server siehe Abschnitt 6 „Lokale Gruppenrichtlinie".) |
| **Backend installiert** | Das revio-Backend sollte installiert sein; das Setup übernimmt dann die `revioClientConfig.zip` (Zertifikat + Serververbindung) automatisch. Alternativ kann eine `revioClientConfig.zip` manuell angegeben werden. |
| **Rechte** | Lokaler Administrator auf dem Server für das Setup; **Domänen-Administrator** (oder delegierte GPO-Rechte) für die Gruppenrichtlinie. |
| **Anmelderecht der Benutzer** | Die Benutzer müssen sich per RDP anmelden dürfen (Mitglied der Gruppe **Remotedesktopbenutzer**). Siehe Abschnitt 7. |

---

## 2. Schritt 1 – revio 4 Setup im Modus „Terminal-Server / RDS" ausführen

Das Setup (`RevioSetup.exe`) als Administrator auf dem Sitzungshost starten und durch den
Assistenten gehen:

1. **Startseite:** „Terminal-Server / RDS (Client-Verteilung)" auswählen.
2. **Voraussetzungen:** durchlaufen lassen.
3. **Client-Konfiguration** – folgende Einstellungen für die GPO-Verteilung:

| Einstellung | Wert | Grund |
|---|---|---|
| **Anmeldungs-Task registrieren** | **deaktivieren** | Steuert den GPO- statt den Aufgabenplanungs-Weg. Deaktiviert = das Setup legt nur das Anmeldeskript ab; die Verteilung erfolgt per GPO. |
| **SMB-Freigabe für die Ablage erstellen** | **aktivieren** | Das Anmeldeskript muss per UNC-Pfad erreichbar sein. |
| **Zentrale Ablage** | Standard `C:\Deploy\revioClient` | Enthält Paket, Abhängigkeiten, Zertifikat, Skript. |
| **Freigabename** | Standard `Deploy` | Ergibt die Freigabe `\\SERVER\Deploy`. |
| **Server-Zertifikat importieren** | aktivieren | Importiert das Backend-Zertifikat maschinenweit → für alle Benutzer vertrauenswürdig. |
| **.NET-Runtimes / Windows App Runtime** | aktivieren | Stellt die Laufzeiten bereit (auf Server 2019 zusätzlich pro Benutzer über das Anmeldeskript, siehe unten). |
| **Client zusätzlich sofort für den aktuellen Benutzer installieren** | optional | Praktisch als sofortiger Funktionstest für den anmeldenden Administrator. |

4. **Zusammenfassung** prüfen, Installation ausführen, **Fertigstellen**.

**Ergebnis auf dem Server:**
- Zentrale Ablage `C:\Deploy\revioClient` mit dem Client-Paket, `revioClient-Logon.ps1`,
  `ServerList.json`, `revioServer.cer` und `windowsappruntimeinstall-x64.exe`.
- Freigabe `\\SERVER\Deploy` mit Leserechten für alle authentifizierten Benutzer.
- Backend-Zertifikat im maschinenweiten Zertifikatspeicher (Vertrauenswürdige Stammzertifizierungsstellen).

Der für die GPO benötigte Skriptpfad lautet (Servername anpassen):

```
\\SERVER\Deploy\revioClient\revioClient-Logon.ps1
```

---

## 3. Schritt 2 – Gruppenrichtlinie erstellen (Domäne, GPMC)

Auf einem Domänencontroller oder einem System mit installierter **Gruppenrichtlinienverwaltung
(GPMC)** – als Domänen-Administrator:

> Ist GPMC nicht vorhanden: `Install-WindowsFeature GPMC`, dann `gpmc.msc`.

1. **GPMC öffnen** → `Domänen` → *Ihre Domäne* aufklappen.
2. Rechtsklick auf die **OU, die den Sitzungshost (Computerobjekt) enthält** →
   **„Gruppenrichtlinienobjekt hier erstellen und verknüpfen…"** → Name z. B. `revio 4 Client Logon`.
   *(Der Server muss in einer OU liegen – nicht im Standardcontainer „Computers", an den keine
   GPO verknüpft werden kann. Speicherort prüfen: `Get-ADComputer <SERVER> | Select DistinguishedName`.)*
3. Rechtsklick auf die neue GPO → **„Bearbeiten"** (öffnet den Gruppenrichtlinien-Editor).

Im Editor **drei** Einstellungen setzen:

**a) Anmeldeskript (PowerShell):**
```
Benutzerkonfiguration → Richtlinien → Windows-Einstellungen →
  Skripts (Anmelden/Abmelden) → Anmelden → Registerkarte „PowerShell-Skripts" → Hinzufügen
    Skriptname:  \\SERVER\Deploy\revioClient\revioClient-Logon.ps1
    (Parameter leer lassen)
```

**b) Skriptausführung erlauben** (das Skript ist nicht signiert und liegt auf einer Freigabe):
```
Computerkonfiguration → Richtlinien → Administrative Vorlagen →
  Windows-Komponenten → Windows PowerShell → „Skriptausführung aktivieren"
    → Aktiviert, „Alle Skripts zulassen"
```

**c) Loopbackverarbeitung** (damit die Benutzer-Einstellungen für **jeden** greifen, der sich am
Sitzungshost anmeldet – die GPO ist am Computer-Objekt verknüpft):
```
Computerkonfiguration → Richtlinien → Administrative Vorlagen → System →
  Gruppenrichtlinie → „Loopbackverarbeitungsmodus für Benutzergruppenrichtlinie konfigurieren"
    → Aktiviert, Modus = Zusammenführen (Merge)
```

Der Editor kann geschlossen werden – die GPO ist bereits mit der OU verknüpft.

---

## 4. Schritt 3 – Anwenden und prüfen

Auf dem Sitzungshost:
```powershell
gpupdate /force
```
Anschließend meldet sich ein **Testbenutzer** per RDP neu an (das Anmeldeskript läuft bei der
**nächsten** Anmeldung, nicht in einer laufenden Sitzung). In der Benutzersitzung prüfen:

```powershell
Get-Content "$env:LOCALAPPDATA\revio\logon.log"      # Protokoll des Anmeldeskripts
Get-AppxPackage *revioClient* | Format-Table Name, Version
Test-Path "$env:USERPROFILE\revio\ServerList.json"   # Serververbindung verteilt
```

Erwartet im Protokoll:
- Auf **Server 2019**: `Windows App Runtime im Benutzerkontext installiert (EXE).` gefolgt von
  `revioClient registriert: revioClient_<Version>_x64.msix`.
- Auf **Server 2022 / neuer**: direkt `revioClient registriert: …`.

Der revio 4 Client erscheint im Startmenü (Alle Apps) und – falls aktiviert – als
Desktop-Verknüpfung. Bei jeder weiteren Anmeldung prüft das Skript die Version und tut nichts,
wenn die aktuelle Version bereits installiert ist (idempotent).

---

## 5. Hinweis Windows Server 2019

Auf Server 2019/2016 lässt sich die **Windows App SDK-Laufzeit** nicht zuverlässig als
MSIX-Abhängigkeit pro Benutzer registrieren. Das Anmeldeskript erkennt diese Server automatisch
und installiert die Laufzeit stattdessen im **Benutzerkontext** über
`windowsappruntimeinstall-x64.exe` (aus der zentralen Ablage; ersatzweise Download von
`https://aka.ms/windowsappsdk/2.4/2.4.0/windowsappruntimeinstall-x64.exe`). Es ist dafür kein
manueller Eingriff nötig.

Das maschinenweite MSIX-Provisioning schlägt auf Server 2019 häufig fehl – das ist **unkritisch**
und wird im Setup nur als Warnung angezeigt. Die zuverlässige Verteilung läuft über das
Anmeldeskript, nicht über das Provisioning.

---

## 6. Alternative – einzelner Server ohne Domäne (lokale Gruppenrichtlinie)

Für einen einzelnen Sitzungshost (oder zum Testen) genügt die **lokale** Gruppenrichtlinie – ohne
Domäne, GPMC oder OU:

```
gpedit.msc →
  Benutzerkonfiguration → Windows-Einstellungen → Skripts (Anmelden/Abmelden) → Anmelden
    → PowerShell-Skripts → Hinzufügen → C:\Deploy\revioClient\revioClient-Logon.ps1
  Computerkonfiguration → Administrative Vorlagen → Windows-Komponenten → Windows PowerShell
    → „Skriptausführung aktivieren" → Aktiviert, „Lokale und remote-signierte Skripts zulassen"
```
Hier kann der **lokale Pfad** verwendet werden (kein UNC), und „Lokale … zulassen" genügt.
Danach `gpupdate /force` und Testbenutzer neu anmelden.

> Alternativ ganz ohne GPO: Beim Setup **„Anmeldungs-Task registrieren" aktiviert lassen** – dann
> legt das Setup einen geplanten Task an, der das Anmeldeskript bei jeder Anmeldung ausführt. Das
> ist der einfachste Weg für einen einzelnen Server.

---

## 7. Fehlerbehebung

| Symptom | Ursache / Lösung |
|---|---|
| *„… das Benutzerkonto ist nicht zur Remoteanmeldung autorisiert"* | Benutzer ist nicht Mitglied von **Remotedesktopbenutzer**. Hinzufügen: `Add-LocalGroupMember -SID S-1-5-32-555 -Member 'DOMÄNE\Benutzer'`. |
| Skript läuft nicht bei der Anmeldung | Skriptausführungs-Richtlinie fehlt (Abschnitt 3b) – bei UNC „Alle Skripts zulassen". GPO korrekt mit der OU des Servers verknüpft? Loopback = Zusammenführen gesetzt? `gpupdate /force` und neu anmelden. |
| Kein `logon.log` | Das Skript wurde nicht ausgelöst (siehe Zeile darüber) oder der UNC-Pfad ist nicht erreichbar/lesbar. |
| `logon.log` zeigt einen Add-AppxPackage-Fehler | Protokoll prüfen; auf Server 2019 muss der Runtime-EXE-Schritt vorher erfolgreich sein. Manueller Test siehe unten. |
| „Maschinenweites Provisioning fehlgeschlagen" im Setup | Unkritisch (siehe Abschnitt 5); die Verteilung erfolgt über das Anmeldeskript. |

**Manueller Test des Anmeldeskripts** (in der Sitzung des Testbenutzers):
```powershell
powershell -ExecutionPolicy Bypass -File "C:\Deploy\revioClient\revioClient-Logon.ps1"
Get-Content "$env:LOCALAPPDATA\revio\logon.log"
```

---

## 8. Updates / neue Client-Version verteilen

Um eine neue Client-Version auszurollen: das revio 4 Setup erneut im Modus
„Terminal-Server / RDS" ausführen (gleiche Einstellungen). Es aktualisiert die zentrale Ablage
mit dem neuen Paket. Beim nächsten Anmelden erkennt das Anmeldeskript die neuere Version und
aktualisiert den Client pro Benutzer automatisch. Auch geänderte Serververbindungen
(`ServerList.json`) werden dabei an alle Benutzer verteilt.

---

## 9. Weitere Verteilungsmethoden (Übersicht)

Der revio 4 Client ist ein **MSIX-Paket**. Entscheidend für alle Verteilungsmethoden ist eine
Eigenschaft von MSIX: Ein Paket wird **immer pro Benutzer registriert**. Es gibt – anders als
bei klassischem MSI – **keinen maschinenweiten Installationszustand**, den sich alle Benutzer
teilen. Die Paketdateien liegen zwar nur einmal auf dem System (`%ProgramFiles%\WindowsApps`),
die eigentliche **Registrierung** (Startmenü-Eintrag, Dateiverknüpfungen, benutzerspezifische
Daten) erfolgt jedoch je Benutzer – und zwar **bei dessen Anmeldung**.

Daraus ergeben sich drei grundsätzliche Muster, den Client auf einem Sitzungshost bereitzustellen:

| Muster | Wie | Deckt bestehende Benutzer ab? |
|---|---|---|
| **Pro Benutzer bei Anmeldung registrieren** (Anmeldeskript) | `Add-AppxPackage` je Anmeldung, idempotent | **Ja** – der zuverlässigste Weg auf allen Systemen |
| **Einmal bereitstellen, automatisch registrieren** (maschinenweites Provisioning) | `Add-AppxProvisionedPackage` | **Nein** – nur **neue** Benutzerprofile (siehe unten) |
| **Dynamisch einbinden** (App Attach) | Paket wird pro Sitzung aus einem Datenträgerabbild gemountet | Sonderfall, primär Azure Virtual Desktop |

### 9.1 Anmeldeskript (empfohlener Standard)

Das in den Abschnitten 2–6 beschriebene Anmeldeskript ist auf **jedem** unterstützten
Betriebssystem der empfohlene Weg. Es registriert den Client im Kontext des sich anmeldenden
Benutzers und ist idempotent (bereits aktuelle Version → keine Aktion). Es ist die **einzige**
Methode, die auch **bereits vorhandene** Benutzerprofile zuverlässig abdeckt, und es behandelt
auf Server 2019 zusätzlich die pro Benutzer nötige Windows App Runtime (siehe Abschnitt 5).

### 9.2 Maschinenweites Provisioning (Ergänzung für Server 2022+)

Mit `Add-AppxProvisionedPackage` wird das Paket **einmalig maschinenweit bereitgestellt**
(*Staging*). Windows registriert es anschliessend über den App-Readiness-Dienst automatisch –
**allerdings nur für _neue_ Benutzerprofile bei deren erster Anmeldung**.

```powershell
# Einmalig, als Administrator auf dem Sitzungshost (Server 2022+):
Add-AppxProvisionedPackage -Online `
  -PackagePath        "C:\Deploy\revioClient\revioClient_<Version>_x64.msix" `
  -DependencyPackagePath "C:\Deploy\revioClient\Dependencies\x64\Microsoft.WindowsAppRuntime.2.msix" `
  -SkipLicense

# Prüfen:
Get-AppxProvisionedPackage -Online | Where-Object DisplayName -like "*revioClient*"

# Entfernen (nur Auto-Registrierung; bereits registrierte Benutzer bleiben unberührt):
Remove-AppxProvisionedPackage -Online -PackageName <PackageFullName>
```

> **Wichtig – neue vs. bestehende Benutzer:** Provisioning registriert den Client **nur für
> Benutzer, die sich nach dem Provisioning erstmals anmelden**. Benutzer, die sich zuvor bereits
> am Server angemeldet haben, erhalten den Client dadurch **nicht** zuverlässig. Da die meisten
> RDS-Umgebungen bereits über bestehende Profile verfügen, ersetzt Provisioning das
> Anmeldeskript **nicht** – beide ergänzen sich: Provisioning beschleunigt die Erstanmeldung
> **neuer** Benutzer, das Anmeldeskript deckt **bestehende** Benutzer (und Server 2019) ab.

> **Hinweis – Server 2019:** Das Provisioning-Verfahren selbst funktioniert ab
> Windows Server 2016 (mit Desktopdarstellung). Auf Server 2019 lässt sich jedoch die
> **Windows-App-Runtime-Abhängigkeit** nicht zuverlässig maschinenweit bereitstellen; das
> Provisioning schlägt dort häufig fehl. Auf Server 2019 daher ausschliesslich das Anmeldeskript
> verwenden (siehe Abschnitt 5). Zuverlässig ist das Provisioning erst ab **Server 2022**.

### 9.3 Weitere Wege (situationsabhängig)

| Methode | Eignung für klassische RDS-Server |
|---|---|
| **Microsoft Configuration Manager (SCCM/MECM)** | Sinnvoll nur, wenn ohnehin im Einsatz. Verteilt das MSIX an Geräte-/Benutzersammlungen; Registrierung bleibt pro Benutzer. |
| **App Attach** | Primär für **Azure Virtual Desktop** bzw. Enterprise-Editionen konzipiert; auf klassischen Terminal-Servern nur mit erheblichem Skripting-Aufwand. Für eine einzelne LOB-App überdimensioniert. |
| **Microsoft Intune** | Nur für Entra-verwaltete Endgeräte / AVD relevant – **nicht** für klassische Windows-Server-Sitzungshosts. |
| **Microsoft Store** | Nur Installation pro Benutzer, keine Verteilung für alle. Für LOB-Apps nicht geeignet. |

> **Fazit:** Für einen typischen On-Premises-Terminal-Server decken das **Anmeldeskript**
> (Standard) und – auf Server 2022+ – **ergänzendes maschinenweites Provisioning** den
> Grossteil aller Fälle ab. Die übrigen Methoden lohnen sich nur, wenn die betreffende Plattform
> (ConfigMgr, AVD) im Unternehmen bereits vorhanden ist.
