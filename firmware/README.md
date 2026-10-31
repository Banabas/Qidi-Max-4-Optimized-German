# FIFO-Fix für die Max-4-THR

`max4-thr-lis2dw-fifo-v1` korrigiert die Entscheidung „LIS2DW-FIFO leer“ im Max-4-THR-Image `02.02.01.08`, das mit der QIDI-Firmware `01.01.07.01` ausgeliefert wird. Er übernimmt das Verhalten des [Upstream-Klipper-Fixes `6269dda56b`](https://github.com/Klipper3d/klipper/commit/6269dda56bb7c08263153b18715d08e2d964bcab), ohne die separate Änderung an der FIFO-Stapelverarbeitung.

Wie du den Fix auf einem Drucker wieder rückgängig machst, steht unter [Original-THR-Firmware aus der Sicherung wiederherstellen](#original-thr-firmware-aus-der-sicherung-wiederherstellen).

Der Generator ändert fünf Bytes des THR-Images an zwei Befehlsstellen. Alle übrigen Komponenten-Nutzdaten, das Firmware-Manifest, interne Dateinamen und die Versionsstrings des Herstellers bleiben unverändert. Der ZIP-Container wird neu gepackt. Die unterstützten Hashes der Eingangs- und Ausgangs-Images sind in [`max4-thr-lis2dw-fifo-v1.json`](max4-thr-lis2dw-fifo-v1.json) hinterlegt.

## Das Paket erzeugen

Verwende Python 3.11 oder neuer auf einem Entwicklungsrechner. Lade genau dieses [Max-4-Release-Archiv](https://github.com/thelegendtubaguy/Qidi-Max4-Defaults/releases/tag/qidi-max4-firmware-01.01.07.01-0cc97445e77295116adaa665ff463e77608d4929520bd7cd46e732fbb0a6ecf2) herunter und führe dann im Stammverzeichnis des Repositorys aus:

```bash
python3 scripts/build_thr_firmware.py \
  --stock-archive QD_MAX4_01.01.07.01_20261016_Release.zip \
  --output-dir dist/max4-thr-fifo-v1
```

Das Ausgabeverzeichnis darf noch nicht existieren. Der Generator lehnt unbekannte, veränderte und bereits gepatchte Archive ab. Er lässt das Originalarchiv unangetastet und erzeugt:

- `QD_MAX4_01.01.07.01_20261016_TLTG_THR_FIFO_v1.zip`: das vollständige Herstellerpaket mit den gepatchten THR-Nutzdaten.
- `QD_MAX4_THR_02.02.01.08_20260303_TLTG_FIFO_v1.bin`: dasselbe gepatchte THR-Image als Einzeldatei.
- `max4-thr-lis2dw-fifo-v1.report.json`: Herkunft der Eingabe, ersetzte Befehle und Hashes der Ausgaben.
- Zu jeder erzeugten Datei eine `.sha256`-Begleitdatei.

Der SHA-256 des gepatchten THR-Images lautet `9d2360ba47ebd720dbef07a0a4ebaf26004360bca090997d7b4a118c41460ac9`.

## Das Paket prüfen

Installiere die Abhängigkeiten für die Prüfung in einer virtuellen Umgebung:

```bash
python3 -m venv .venv-thr
.venv-thr/bin/python -m pip install -r firmware/check-requirements.txt
.venv-thr/bin/python scripts/check_thr_firmware.py \
  --stock-archive QD_MAX4_01.01.07.01_20261016_Release.zip \
  --patched-dir dist/max4-thr-fifo-v1
```

Der Prüfer vergleicht alle Nutzdaten im Archiv und das THR-Protokollwörterbuch, prüft Prüfsummen und Herkunftsnachweis der erzeugten Dateien, testet die Ablehnungsgrenzen und emuliert die originalen und gepatchten FIFO-Abfragebefehle über jedes FIFO-Statusbyte und zwei Zustände des Berichtspuffers. SPI-Übertragungen, Berichte an den Host und Timer-Registrierung werden nur simuliert; das misst weder die tatsächliche Abtastrate noch validiert es die übrigen Firmware-Pfade des Druckers.

## Vor dem Flashen

Erzeugen und Prüfen kontaktieren keinen Drucker und flashen nichts. Der Installer der optimierten Makros bietet eine interaktive THR-Firmware-Frage mit der Voreinstellung Nein sowie die ausdrückliche Option `--flash-thr-fifo`. Er prüft die tatsächlichen Bytes der Anwendung und überspringt das Programmieren, wenn der Fix bereits installiert ist. Bootloader werden nur anhand ihrer exakten, validierten Build-Kennung zugelassen; ein unbekannter Build wird ohne Reset oder Programmieren erfasst, mit einer teilbaren `report.json` für ein GitHub-Issue. Seine Firmware bleibt unverändert, die vorherigen Dienste werden wiederhergestellt, und die Konfigurationsinstallation bleibt erhalten. `--yes` allein und automatische Updates erlauben kein Programmieren von Firmware.

Das Paket behält QIDIs Versionsstrings bei und erzwingt kein erneutes Flashen der THR mit gleicher Version. Ein abgeschlossenes Update über den Touchscreen beweist nicht, dass die gepatchte THR geflasht wurde. Diese Build-Prüfungen belegen weder die Installation noch eine physische Wiederherstellungsmöglichkeit. Das eingeschränkte Werkzeug, die Verifizierung per Rücklesen und die erforderliche Validierung des Wiederherstellungseinstiegs sind unter [Nur-THR-Vorgänge](FLASHING.md) beschrieben.

Das ZIP ist ein **vollständiges QIDI-Update**, einschließlich des unveränderten SOC und der übrigen MCU-Komponenten. Seine Installation kann die Druckerkonfiguration ersetzen und die optimierten Makros entfernen, genau wie ein Original-Update. Bewahre das Originalarchiv zur Wiederherstellung auf und sichere die Druckerkonfiguration vor jeder Firmware-Installation. Verwende das einzelne Image nur mit einem verifizierten Flash- und Wiederherstellungsverfahren für die Max-4-THR.

## Original-THR-Firmware aus der Sicherung wiederherstellen

Mit diesem Verfahren entfernst du den FIFO-Fix und spielst QIDIs Original-THR-Anwendung wieder auf. Konfigurationswiederherstellung (`restore.sh`), Deinstallation und eine erneute Installation von QIDIs Update `01.01.07.01` stellen die THR-Firmware nicht wieder her. Führe alle Befehle auf dem Drucker per SSH als `qidi` aus.

Beim Wiederherstellen wird die THR-Anwendung gelöscht und neu programmiert, derselbe Vorgang wie beim Flashen des Fixes. Lass den Drucker eingeschaltet, bis die Verifizierung abgeschlossen ist. Existiert `~/printer_data/.tltg_thr_firmware_pending.json`, wurde ein Firmware-Vorgang des Installers nicht abgeschlossen; folge dann stattdessen dem Fehlerverfahren unter [Nur-THR-Vorgänge](FLASHING.md).

### 1. Die Original-Sicherung finden

Bevor der Installer eine Platine im Originalzustand flasht, speichert er die Original-Anwendung und den Bootloader in einem Ordner `before/` unter `~/printer_data/.tltg_thr_firmware_backups/`. Liste die Sicherungen auf, die die Original-Anwendung enthalten:

```bash
grep -l 1d34b4b0142f2a2047f08c2fae59d827bcee0adbd1563b4bd1de311dff8b2d62 \
  ~/printer_data/.tltg_thr_firmware_backups/thr-*/before/snapshot.json
```

Setze `RECOVERY` auf den `thr-…`-Ordner einer aufgelisteten Datei, zum Beispiel:

```bash
RECOVERY=~/printer_data/.tltg_thr_firmware_backups/thr-abcd1234
```

Ein Installer-Lauf, der den Fix bereits installiert vorfand, sichert nur die gepatchte Anwendung und wird deshalb nicht aufgelistet. Wird nichts aufgelistet, erstelle die Original-Sicherung aus QIDIs [`QD_MAX4_01.01.07.01_20261016_Release.zip`](https://github.com/thelegendtubaguy/Qidi-Max4-Defaults/releases/download/qidi-max4-firmware-01.01.07.01-0cc97445e77295116adaa665ff463e77608d4929520bd7cd46e732fbb0a6ecf2/QD_MAX4_01.01.07.01_20261016_Release.zip). Kopiere das Archiv auf den Drucker und nimm den Bootloader aus einem beliebigen `thr-…`-Sicherungsordner:

```bash
RECOVERY=~/thr-stock-recovery
mkdir -p "$RECOVERY/before"
unzip -p QD_MAX4_01.01.07.01_20261016_Release.zip QD_MAX4_THR_02.02.01.08_20260303.bin \
  > "$RECOVERY/before/application.bin"
cp ~/printer_data/.tltg_thr_firmware_backups/thr-abcd1234/before/bootloader.bin "$RECOVERY/before/"
```

Das Werkzeug lehnt Anwendungen oder Bootloader ab, deren SHA-256 nicht hinterlegt ist; eine falsche oder beschädigte Datei schlägt also fehl, bevor irgendetwas gelöscht wird.

### 2. Die Druckerdienste stoppen

Vergewissere dich, dass kein Druck läuft oder pausiert ist und alle Heizungen, auch die Kammer, auf Zieltemperatur null stehen. Notiere, welche Dienste aktiv sind, und stoppe sie dann:

```bash
systemctl is-active klipper.service qidi-client.service tltg-optimized-auto-update.timer
sudo systemctl stop tltg-optimized-auto-update.timer
sudo systemctl stop qidi-client.service
sudo systemctl stop klipper.service
```

### 3. Die Original-Anwendung flashen

```bash
/home/qidi/klippy-env/bin/python ~/tltg-optimized-macros/scripts/flash_thr_firmware.py transfer \
  --image "$RECOVERY/before/application.bin" \
  --backup-dir "$RECOVERY/before" --reset \
  --printer-idle-confirmed --confirm-write
```

Eine erfolgreiche Übertragung gibt `"transfer_acknowledged": true` und `"verified": false` aus. Die Wiederherstellung ist erst abgeschlossen, wenn Schritt 4 erfolgreich ist.

### 4. Die wiederhergestellte Firmware verifizieren

Warte mindestens zwölf Sekunden, bis die THR-Anwendung zurückkehrt, lies sie dann zurück und vergleiche sie mit dem Original-Image:

```bash
sleep 12
/home/qidi/klippy-env/bin/python ~/tltg-optimized-macros/scripts/flash_thr_firmware.py snapshot \
  --output-dir "$RECOVERY/restored" --printer-idle-confirmed
/home/qidi/klippy-env/bin/python ~/tltg-optimized-macros/scripts/flash_thr_firmware.py verify \
  --image "$RECOVERY/before/application.bin" --backup-dir "$RECOVERY/restored" \
  --original-backup-dir "$RECOVERY/before"
```

`--output-dir` darf noch nicht existieren; verwende einen neuen Namen, wenn du diesen Schritt wiederholst. Die Wiederherstellung war nur dann erfolgreich, wenn verify `"verified": true` und `"application_sha256": "1d34b4b0142f2a2047f08c2fae59d827bcee0adbd1563b4bd1de311dff8b2d62"` ausgibt. Schlägt ein Schritt fehl oder kehrt die Anwendung nicht zurück, lass die Dienste gestoppt und folge [Nur-THR-Vorgänge](FLASHING.md).

### 5. Die Druckerdienste neu starten

Starte nur die Dienste, die in Schritt 2 aktiv waren:

```bash
sudo systemctl start klipper.service
sudo systemctl start qidi-client.service
sudo systemctl start tltg-optimized-auto-update.timer
```

Vergewissere dich vor dem Drucken, dass Klipper bereit ist. Die Wiederherstellung ändert die `printer.cfg` nicht. Die in einem `thr-…`-Ordner gespeicherte `printer.cfg` ist die Kopie von vor dem Entfernen der Input-Shaper-Kalibrierung durch den Installer; kopiere sie nicht über die aktuelle Datei, denn damit verwirfst du jede seitdem vorgenommene Konfigurationsänderung. Führe `M901` erneut aus, wenn du die Input-Shaper-Kalibrierung mit der Original-Firmware messen möchtest.
