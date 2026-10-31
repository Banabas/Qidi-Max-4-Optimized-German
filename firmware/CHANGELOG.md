# Änderungsprotokoll der THR-Firmware

## max4-thr-lis2dw-fifo-v1

- Die invertierte Entscheidung „LIS2DW-FIFO leer“ im Max-4-THR-Image `02.02.01.08` aus der QIDI-Firmware `01.01.07.01` wurde korrigiert; das Aufweckverhalten des Schedulers bleibt erhalten, und es werden nur fünf Bytes des Images geändert.
- Hash-gesicherte Erzeugung eines separat benannten vollständigen Firmware-ZIPs und eines eigenständigen THR-Images, mit Prüfsummen und Herkunftsnachweis. Die Nutzdaten der übrigen Komponenten, das Firmware-Manifest und die Versionskennungen des Herstellers bleiben unverändert.
- Eingeschränkte Befehle nur für die THR hinzugefügt: Snapshot, nicht löschende Abfrage (Probe), bestätigte Übertragung und Verifizierung per Rücklesen – mit Zulassung nur bekannter Images, Prüfung der Dienstzuständigkeit, Aufbewahrung der Original-Wiederherstellungsdateien und begrenzten Offline-Übertragungsprüfungen. Eine mehrdeutige Bestätigung des ersten Datenpakets bricht die Übertragung ab, statt einen nicht idempotenten Schreibvorgang zu wiederholen. Der physische Einstieg in die Wiederherstellung muss verifiziert sein, bevor Firmware gelöscht wird.
