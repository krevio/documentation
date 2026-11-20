# revio AG — Dokumentation

Willkommen im Dokumentations-Repository der revio AG. In diesem Repository stellen wir Handbücher, Anleitungen und Referenzen zur Installation, zum Betrieb und zur Administration der revio 4 Software zur Verfügung.

Inhalt
- Installationsanleitungen für unterstützte Plattformen
- Troubleshooting und häufige Fehlerlösungen
- Release Notes und Versionshinweise

Zielgruppe
Diese Dokumentation richtet sich an Systemadministrator:innen, DevOps‑Teams, Support‑Mitarbeiter:innen sowie technische Ansprechpartner:innen bei Kund:innen.

Schnellstart
1. **Server (Backend) und Terminal-Server/RDS:** Nutzen Sie den Setup-Assistenten [`RevioSetup.exe`](https://reviofiles.blob.core.windows.net/revio4-installation-files/setup/RevioSetup.exe) – siehe [Installation und Wartung mit dem Setup-Assistenten](docs/setup/reviosetup.md). Er ist in erster Linie für die **Backend-Installation** und die **Client-Installation auf Terminal-Servern (RDS)** gedacht und übernimmt Voraussetzungen, Zertifikate und Konfiguration automatisch.
2. **Client auf einzelnen Arbeitsplätzen:** Laden Sie das Client-Paket [`revioClient.zip`](https://reviofiles.blob.core.windows.net/revio4-installation-files/latest/revioClient.zip) herunter, entpacken Sie es und führen Sie `UpdateRevioMsix.ps1` als Administrator aus – siehe [revio 4 Client – manuelle Installation](docs/client/clientInstall-WinServer.md).
   Auf einer normalen Windows-11-Arbeitsstation kann der Benutzer die enthaltene `revioClient_<Version>_x64.msix` auch einfach per Doppelklick installieren – ohne Administratorrechte. Die Windows App Runtime wird dabei vom MSIX-Installer automatisch mitinstalliert. Die .NET 10 Runtime muss vorhanden sein; fehlt sie, verweist die App beim Start auf den Download. Bei einer Neuinstallation wird im Anmeldebildschirm die `revioClientConfig.zip` aus der Backend-Installation eingelesen (Server-Zertifikat und Serververbindung).
3. Folgen Sie der Grundkonfiguration.

Dokumentation im Überblick
- **Setup-Assistent (Backend & Terminal-Server/RDS)**
  - [revio 4 Setup – Installation und Wartung](docs/setup/reviosetup.md)
  - [RDS-Client-Verteilung per GPO](docs/setup/rds-client-verteilung-gpo.md)
- **Backend**
  - [Öffentliches Server-Deployment (internetseitig)](docs/backend/revio-public-server-deployment.md)
- **Client (Einzelarbeitsplatz / manuelle Installation)**
  - Download: [`revioClient.zip`](https://reviofiles.blob.core.windows.net/revio4-installation-files/latest/revioClient.zip) (immer die aktuellste offizielle Version; ältere Versionen ab 4.0.380 im [Archiv](docs/client/clientInstall-WinServer.md#download))
  - [revio 4 Client auf Windows Server 2019+](docs/client/clientInstall-WinServer.md)
- **Migration**
  - [revio Migrator](docs/migration/revio-migrator.md)
  - [Migrationsdatei-Import](docs/migration/revio4-migration-file-import.md)

Support / Kontakt
Für Support kontaktieren Sie bitte das revio AG Support‑Team (z. B. support@revio.ch).
