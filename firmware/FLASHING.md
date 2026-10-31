# Nur-THR-Vorgänge beim Max 4

`scripts/flash_thr_firmware.py` arbeitet ausschließlich auf `/dev/ttyS4` und nur mit der bekannten Max-4-THR-Anwendung und einem zugelassenen Bootloader-Profil. Es verwaltet keine Dienste, ändert keine Druckerkonfiguration, führt keine Bewegungs- oder Heizbefehle aus und aktualisiert keine anderen Platinen. Führe Hardware-Vorgänge auf dem Drucker mit dessen vorhandenem `/home/qidi/klippy-env/bin/python` aus; behalte die Verzeichnisstruktur des Repositorys bei, damit das Skript sein Firmware-Manifest lesen kann.

**Lösche keine Firmware, bevor ein platinenspezifischer Wiederherstellungseinstieg verifiziert wurde.** Beide zugelassenen Bootloader haben beim Start nur kurze Empfangsfenster, und der Linux-Start kann länger dauern. Ein Aus- und Einschalten des ganzen Druckers ist kein verifiziertes Wiederherstellungsverfahren. Sicherungen von Original-Image und Bootloader allein stellen noch keinen physischen Zugang zur Wiederherstellung sicher.

## Zustimmung über den Installer

Auf der unterstützten Firmware `01.01.07.01` fragt die interaktive Installation, ob der FIFO-Fix geflasht werden soll, mit der Voreinstellung Nein. Ein ausdrückliches y/yes gilt nur für diesen einen Aufruf. Führe nach dem Flashen der THR vor dem nächsten Druck die Input-Shaper-Kalibrierung erneut aus; der Installer führt die Kalibrierung nicht aus. `install.sh --flash-thr-fifo` erteilt dieselbe Zustimmung, ohne dass die Frage beantwortet werden muss; `--yes` allein und automatische Updates erlauben niemals Zugriff auf die Firmware. `--flash-thr-fifo --dry-run` zeigt eine Vorschau ohne UART-Zugriff und ohne Änderungen an Diensten.

Der Installer liest die tatsächliche Anwendung und den Bootloader, bevor er entscheidet, ob programmiert wird. Eine bereits gepatchte Anwendung wird verifiziert, ohne sie zurückzusetzen oder neu zu flashen. Bei Original-Firmware bewahrt der Installer die Wiederherstellungsdateien auf, erzeugt den hash-gesicherten Patch aus dem Rückgelesenen, überträgt nur die THR und verifiziert das vollständige Rücklesen. Auf dem Build `31f8344d…` läuft die Übertragung im Wiederherstellungsmodus: Fällt sie auf die Original-Anwendung zurück, verifiziert der Installer das Rücklesen der Original-Firmware, stellt die vorherigen Dienste ohne Firmware-Sperre wieder her, behält die Input-Shaper-Kalibrierung bei und meldet, dass der Fix nicht angewendet wurde. Private Nachweise bleiben unter `~/printer_data/.tltg_thr_firmware_backups/`; Konfigurationswiederherstellung und Deinstallation setzen die MCU-Firmware nicht zurück. Ein unbekannter Bootloader wird ohne Reset oder Programmieren erfasst, danach werden die zuvor aktiven Dienste wiederhergestellt und die Betriebsbereitschaft geprüft. Dabei entsteht keine Firmware-Sperre, und die Konfigurationsinstallation bleibt gültig. Hänge an ein GitHub-Issue nur die `report.json` des Mitschnitts an; halte die Flash-Binärdateien privat.

Ein unterbrochener oder nicht verifizierter Firmware-Vorgang hinterlässt `~/printer_data/.tltg_thr_firmware_pending.json` und sperrt weitere Installer-Vorgänge. Der Eintrag nennt das Wiederherstellungsverzeichnis und die vorherigen Dienstzustände. Falls das Programmieren bereits begonnen haben könnte, bleiben die Dienste, die die serielle Schnittstelle belegen, gestoppt. Verwende für das manuelle Wiederherstellungsverfahren unten die Original-Anwendung und den Bootloader aus dem Snapshot `before/` dieses Verzeichnisses. Entferne die Markierungsdatei erst nach vollständiger Verifizierung von Anwendung und Bootloader, Wiederherstellung der aufgezeichneten Dienstzustände und bestätigter Betriebsbereitschaft des Druckers bei Heizungs-Zieltemperatur null. Verwende nicht den Befehl für die Wiederherstellungsmarkierung der Konfiguration (Recovery Sentinel), um eine Firmware-Sperre zu umgehen.

## Zuständigkeit und Sicherungen

Frage vor jeder Kontaktaufnahme mit dem Drucker dessen Druckstatus ab. Brich ab, wenn der Status nicht verfügbar ist oder einen laufenden oder pausierten Druck anzeigt. Prüfe, nachdem der Leerlauf bestätigt ist, die Bereitschaft von Klipper und alle Heizungs-Zieltemperaturen, einschließlich der Kammerheizung; alle Ziele müssen null sein.

Notiere, welche Dienste aktiv sind. Stoppe den Timer für optimierte Updates, den Touchscreen-Client und Klipper, bevor du den UART öffnest:

```bash
sudo systemctl stop tltg-optimized-auto-update.timer
sudo systemctl stop qidi-client.service
sudo systemctl stop klipper.service
```

Das Werkzeug verweigert den UART-Zugriff, solange nicht alle drei inaktiv sind. `--printer-idle-confirmed` ist deine Zusicherung als Bediener, dass die vorstehenden Prüfungen erfolgt sind, keine API-Prüfung durch das Werkzeug. Bleib während dieser Vorgänge beim Drucker und starte keinen Auftrag.

Lege im Stammverzeichnis des Repositorys auf dem Drucker ein neues Sicherungsverzeichnis an:

```bash
/home/qidi/klippy-env/bin/python scripts/flash_thr_firmware.py snapshot \
  --output-dir dist/thr-before --printer-idle-confirmed
```

Der Snapshot liest den Bootloader und die vollständige, 35.056 Byte große Anwendung über den installierten Klipper-Transport. Er setzt die MCU weder zurück noch schreibt er auf sie. Kopiere das Verzeichnis auf einen separaten Rechner, bevor du weitermachst. Das Original-Wiederherstellungsverzeichnis muss enthalten:

- `bootloader.bin`: 32.768 Byte, SHA-256 `46358f306156edbdee1822dcca17206f78241b01a7e96587215860c4b0d34fc6` oder `31f8344db7b091df5429960bc9cd06065e2188c3b8092e43fb65eb3f2833723c`.
- `application.bin`: SHA-256 der Original-THR `1d34b4b0142f2a2047f08c2fae59d827bcee0adbd1563b4bd1de311dff8b2d62`.
- `snapshot.json`: die erfassten Hashes.

Der Snapshot-Befehl akzeptiert für das Rücklesen nach der Übertragung auch die bekannte gepatchte Anwendung. Löschende Vorgänge lassen nur die oben genannten exakten, validierten Bootloader-Hashes zu, nicht allein die Antworten auf Identitätsabfragen.

Ein Snapshot eines unbekannten Bootloaders speichert `bootloader.bin`, `application.bin`, die vollständige `application-region.bin` und eine teilbare `report.json` und beendet sich mit Status `3`. Die Dateien haben den Modus `0600`; die Zusammenfassung enthält Hashes, Bootloader-Hashes pro Seite, Klipper-Build-Informationen und nicht eindeutige MCU-ID-Register, aber keine Flash-Binärdaten und keine ausgelesenen eindeutigen Gerätekennungen. Es wird kein Reset-, Lösch- oder Programmierbefehl gesendet. Das manuelle Werkzeug stellt die Dienste nicht wieder her; stelle nach diesem Ergebnis ohne Änderung die aufgezeichneten aktiven Dienste wieder her und bestätige die Betriebsbereitschaft. Unbekannte Anwendungsbytes erlauben niemals eine Übertragung. Halte private Sicherungen aus Git heraus.

## Nicht löschende Bootloader-Abfrage

Bei weiterhin gestoppten Diensten und aufbewahrter Original-Sicherung:

```bash
/home/qidi/klippy-env/bin/python scripts/flash_thr_firmware.py probe \
  --backup-dir dist/thr-before --reset --printer-idle-confirmed
```

`--reset` liest und prüft zuerst die installierte Anwendung und den Bootloader und sendet dann nur den `reset`-Befehl der THR-Anwendung. Die Abfrage sendet die einfache Identitätsanfrage mit 115.200 Baud; sie sendet weder den löschenden Startbefehl noch irgendwelche Firmware-Daten. Die erwartete Antwort lautet `01020102` (Modell `02`, Bootloader `1.2`) für den Build `46358f30…` und `0102` für den Build `31f8344d…`.

Lass nach einer erfolgreichen Abfrage mindestens zwölf Sekunden Zeit, damit der Bootloader zur Anwendung zurückkehrt. Erstelle einen weiteren Snapshot in einem neuen Verzeichnis und verifiziere ihn gegen das Original-Image, um nachzuweisen, dass die Flash-Bytes unverändert sind. Nimm den normalen Betrieb nicht wieder auf, wenn die Anwendung nicht zurückkehrt.

## Bestätigte Übertragung und Rücklesen

Erst nachdem du den nicht löschenden Einstieg, die Rückkehr der Anwendung und einen unabhängigen Wiederherstellungszugang verifiziert hast, überträgst du ein zugelassenes Image:

```bash
/home/qidi/klippy-env/bin/python scripts/flash_thr_firmware.py transfer \
  --image dist/max4-thr-fifo-v1/QD_MAX4_THR_02.02.01.08_20260303_TLTG_FIFO_v1.bin \
  --backup-dir dist/thr-before --reset \
  --printer-idle-confirmed --confirm-write
```

Der Startbefehl löscht den gesamten Anwendungsbereich `0x08008000`–`0x0801ffff`, einschließlich der Bytes jenseits des übergebenen Images. Der Bootloader-Bereich ist kein Ziel des Updates. Das Werkzeug lässt nur die hinterlegten Original- und gepatchten Images zu und prüft die Bestätigungen anhand von Index und Byteanzahl jedes Pakets. Bei einer mehrdeutigen Bestätigung von Paket null bricht es ab, statt blind zu wiederholen, denn diese Wiederholung ist bei keinem der beiden validierten Bootloader idempotent.

Eine erfolgreiche Übertragung meldet `transfer_acknowledged: true` und `verified: false`. Lass die Anwendung zurückkehren und erstelle und verifiziere dann ein vollständiges Rücklesen:

```bash
/home/qidi/klippy-env/bin/python scripts/flash_thr_firmware.py snapshot \
  --output-dir dist/thr-after --printer-idle-confirmed
/home/qidi/klippy-env/bin/python scripts/flash_thr_firmware.py verify \
  --image dist/max4-thr-fifo-v1/QD_MAX4_THR_02.02.01.08_20260303_TLTG_FIFO_v1.bin \
  --backup-dir dist/thr-after --original-backup-dir dist/thr-before
```

Die Verifizierung läuft offline und erfordert die gewählten Anwendungsbytes sowie einen zugelassenen Bootloader. `--original-backup-dir` vergleicht zusätzlich den Bootloader Byte für Byte mit seiner Sicherung von vor der Übertragung; der Installer führt diesen Vergleich immer durch. Bytes jenseits des Anwendungs-Images sind kein Nachweis der Installation: Unterschiedliche Bootloader können unterschiedliches Auffüllen im letzten Paket hinterlassen. Der Versionsstring des Herstellers bleibt unverändert und ist ebenfalls kein Nachweis der Installation.

Ergänze beim Build `31f8344d…` den Befehl `transfer` um `--recover-to-stock`. Bei einem fehlgeschlagenen Austausch setzt das Werkzeug dann das angeforderte Image fort, wenn das fehlgeschlagene Paket idempotent ist; andernfalls lässt es die Sitzung des Bootloaders ablaufen und programmiert die Original-Anwendung aus der Sicherung erst, nachdem eine neue Sitzung bestätigt ist. Es bricht mit Status `1` ab, statt den Startbefehl in eine Sitzung zu senden, die noch Daten enthalten könnte. Exit-Status `0` bedeutet, dass das angeforderte Image bestätigt wurde, `4`, dass stattdessen die Original-Firmware als Rückfall programmiert wurde, `5`, dass jedes Datenpaket, aber nicht das Ende bestätigt wurde, und `1`, dass die Wiederherstellung fehlgeschlagen und der Zustand der Platine unbekannt ist. Bei jedem Status sind trotzdem die zwölf Sekunden Wartezeit, ein Snapshot und `verify` gegen das jeweils genannte Image nötig. Der Installer nutzt diesen Modus für diesen Build und behandelt Status `4` als unveränderte Platine. Für den Build `46358f30…` wird er verweigert, da dessen Timing nicht validiert ist. Seine Wiederherstellungspfade liefen bisher nur in der Emulation (`scripts/check_thr_recovery.py`), und kein Wiederherstellungsmodus deckt Stromausfall oder ein Abbrechen des Werkzeugs mitten in der Übertragung ab.

Lass bei einem Fehler die Dienste, die die serielle Schnittstelle belegen, gestoppt. Sende nicht sofort aus einem anderen Programm erneut ein Start- oder Datenpaket. In der begrenzten Emulation löschen beide Bootloader die erste Anwendungsseite, nachdem eine unterbrochene Datensitzung abgelaufen ist. Der Build `46358f30…` erlaubt eine erneute Übertragung der Original-Firmware; der Build `31f8344d…` fordert einen System-Reset an und öffnet damit erneut sein vier Sekunden langes Empfangsfenster beim Start. Das beweist keine Wiederherstellung nach einem Stromausfall: Ein Kaltstart kann in eine teilweise programmierte Anwendung springen. Nutze den unabhängig verifizierten, platinenspezifischen Wiederherstellungseinstieg und übertrage dann `dist/thr-before/application.bin` mit demselben Werkzeug und der Original-Sicherung. Lass `--reset` nur weg, wenn die Wiederherstellung die Platine bereits in ihren Bootloader versetzt hat. Lies das Original-Image zurück und verifiziere es, bevor du den Betrieb wieder aufnimmst.

## Normalen Betrieb wiederherstellen

Stelle nur die Dienste wieder her, die vor dem Vorgang aktiv waren. Nach verifizierter Rückkehr der Anwendung:

```bash
sudo systemctl start klipper.service
sudo systemctl start qidi-client.service
sudo systemctl start tltg-optimized-auto-update.timer
```

Bestätige vor jeder Messung die Betriebsbereitschaft des Druckers und Heizungs-Zieltemperaturen von null. Lässt sich der Druckstatus nicht ermitteln, beende den Zugriff auf den Drucker und prüfe den Leerlauf direkt vor Ort. Eine Aufzeichnung des Beschleunigungssensors im Stillstand kann dann die gelieferte Abtastrate und die Fehlerzähler vergleichen; sie erfordert kein Referenzieren, Heizen, keine Bewegung und keinen Resonanz-Sweep.

## Offline-Übertragungsprüfungen

Installiere auf einem Entwicklungsrechner `firmware/check-requirements.txt` und führe dann aus:

```bash
python3 scripts/check_thr_flashing.py \
  --stock-image stock-thr.bin \
  --patched-image dist/max4-thr-fifo-v1/QD_MAX4_THR_02.02.01.08_20260303_TLTG_FIFO_v1.bin
```

Ergänze `--bootloader-backup bootloader.bin`, um den tatsächlichen Parser, die Löschschleife, die Programmiergrenzen und den Antwortaufbau eines erfassten Bootloaders auszuführen. Wiederhole diese Option, um beide exakten Profile in `scripts/thr_bootloaders.py` zu prüfen. Beide Profile werden auf vollständige Übertragungen, Wiederherstellung nach Unterbrechung, doppelte Pakete, verlorene Bestätigungen, nicht löschende Identitätssitzungen und Zulassungsgrenzen geprüft. Das erste lässt das Auffüllen kurzer Pakete gelöscht, das zweite programmiert Nullen. UART-Ausgabe, Registeroperationen des Flash-Controllers, Sprünge in die Anwendung und System-Resets werden emuliert; diese Prüfungen validieren weder das elektrische Verhalten noch den physischen Wiederherstellungszugang. Im Repository wird keine Bootloader-Binärdatei ausgeliefert.

Für einen Build mit validiertem Wiederherstellungs-Timing nimmt `scripts/check_thr_recovery.py` dieselben drei Optionen, wobei `--bootloader-backup` Pflicht ist. Es treibt den SysTick des erfassten Bootloaders mit der Uhr des Werkzeugs an und erzeugt eine tote Verbindung oder verlorene Antworten beim Start, bei den Paketen 0, 1, 17 und 34 sowie beim Ende, jeweils für 0,5 bis 40 Sekunden. Jeder Fehler durchläuft sowohl `transfer` als auch `transfer_recovering`. Außerdem werden die Fehler mit 3 s und 12 s mit einer um 8 % und 3 % zu langsamen sowie einer um 3 % und 10 % zu schnellen Bootloader-Uhr ausgeführt, weil der Oszillator des Bootloaders nicht verifiziert ist. Die Prüfung schlägt fehl, wenn die wiederherstellende Übertragung eine unbrauchbare Anwendung hinterlässt, obwohl die Verbindung innerhalb des Rettungsfensters des Bootloaders nutzbar war, oder wenn sie Flash programmiert, der nicht gelöscht wurde. `--quick` führt nur die Fehler mit 3 s und 12 s aus; der vollständige Satz dauert mehrere Minuten.
