# revio 4 - Migrationsimport

Der **Migrationsimport** in revio 4 liest die vom revio Migration Tool erstellte ZIP-Exportdatei ein und überführt die Legacy-Daten in die neue revio 4 Datenstruktur.

## Migrationsablauf

Die Migration von revio Legacy nach revio 4 erfolgt in zwei Schritten:

1. **Datenextraktion (revio Migration Tool):** Das Migration Tool extrahiert alle relevanten Daten für das gewählte Umstellungsjahr aus der Legacy-Datenbank und erstellt eine ZIP-Exportdatei. Siehe dazu die [Dokumentation zum Migration Tool](revio-migrator.md).

2. **Import in revio 4 (diese Dokumentation):** Die erstellte ZIP-Datei wird in revio 4 hochgeladen und verarbeitet. revio 4 transformiert die Daten und erstellt die entsprechenden Strukturen.

> **Wichtig:** Vor dem Import müssen alle Mitarbeiter in revio 4 angelegt sein. Verwenden Sie dabei **dieselben Visa** wie in revio 3, damit die Zuordnungen korrekt übernommen werden.

## Voraussetzungen

- Eine gültige ZIP-Exportdatei vom revio Migration Tool
- Alle Mitarbeiter müssen in revio 4 angelegt sein (mit identischen Visa)
- Ausreichende Berechtigungen für den Datenimport (Administrator)

## Import durchführen

### Schritt 1: Migration starten

1. Melden Sie sich in revio 4 an
2. Klicken Sie unten links auf **Einstellungen** ![Einstellungen](images/settings-icon.png)
3. Wählen Sie **Migration starten** ![Migration starten](images/start-migration.png)
