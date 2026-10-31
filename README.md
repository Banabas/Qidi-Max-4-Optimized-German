![Tuba Makes](.github/images/tuba-makes-logo.png)

# Qidi Max 4 Optimized

Eigenwillige und optimierte Klipper-Makros sowie Slicer-Maschinen-G-Code für den QIDI Max 4.

> [!NOTE]
> Dies ist eine deutsche Übersetzung der Dokumentation von [thelegendtubaguy/Qidi-Max-4-Optimized](https://github.com/thelegendtubaguy/Qidi-Max-4-Optimized). Installer, Updates und Releases kommen weiterhin aus dem Original-Repository; Ausgaben auf der Konsole und im Installer sind deshalb auf Englisch.

> [!NOTE]
> Wenn du Inhalte wie diese unterstützen möchtest, abonniere gern den Kanal auf [YouTube](https://youtube.com/@TubaMakes)!

> [!CAUTION]
> Ich übernehme keine Verantwortung für deinen Drucker. Beobachte BITTE den Start jedes Drucks, damit der Drucker nichts Schlimmes anstellt, etwa dein Druckbett zerkratzt, und sei jederzeit bereit, den Strom abzuschalten.

## Kurzfassung
1. Verbinde dich per SSH mit deinem Drucker: `qidi@<IP deines Druckers>`.
2. Führe auf dem Drucker Folgendes aus:
```bash
/bin/bash -c "$(curl -fsSL https://github.com/thelegendtubaguy/Qidi-Max-4-Optimized/releases/latest/download/install-latest.sh)"
```
3. Folge den Abfragen.
4. Nutze Orca und abonniere das [hier](https://cloud.orcaslicer.com/b/4c4b3b74c745) geteilte OrcaCloud-Bundle. Wenn du nicht Orca ab Version 2.4.0 verwendest, lies den [Abschnitt zu den Slicer-Konfigurationen](#maschinen-g-code-für-den-slicer-aktualisieren).
5. Slice mit dem Druckerprofil `Qidi X-Max 4 0.4 nozzle - TLTG Optimized GCode`.
6. Optional: Lege eine Kopie des Maschinenprofils an und passe sie nach deinen Wünschen an.


### Testlauf des Installers (Dry-Run)
Wenn du vor einer vollständigen Installation lieber erst einen Testlauf machen möchtest, führe Folgendes aus:

```bash
/bin/bash -c "$(curl -fsSL https://github.com/thelegendtubaguy/Qidi-Max-4-Optimized/releases/latest/download/install-latest.sh)" -- --dry-run
```

### Optionaler THR-Firmware-Fix für den Beschleunigungssensor

Auf der QIDI-Firmware `01.01.07.01` fragt der interaktive Installer gleich zu Beginn, bevor er irgendeine Konfiguration ändert, ob der Fix für den LIS2DW-FIFO-Fehler auf die THR-Werkzeugkopfplatine geflasht werden soll. **Die Voreinstellung ist Nein**: Drücke Enter, um die Firmware unverändert zu lassen. Bei Ja wird geflasht, nachdem die Konfigurationsinstallation abgeschlossen ist. Der Installer prüft die tatsächlichen Hashes von Anwendung und Bootloader; eine Platine, die den Fix bereits hat, wird ohne erneuten Reset oder Flashvorgang verifiziert. **Führe nach dem Flashen der THR vor dem nächsten Druck die Input-Shaper-Kalibrierung erneut aus.** Der Installer entfernt die gespeicherte Input-Shaper-Kalibrierung aus der `printer.cfg`, sodass Input Shaping bis zur Neukalibrierung ausgeschaltet bleibt; die vorherige `printer.cfg` wird bei den THR-Wiederherstellungsdateien aufbewahrt. Der Installer führt die Kalibrierung nicht selbst aus.

Zum Flashen muss der Drucker im Leerlauf und bereit sein, und alle Heizungen müssen auf Zieltemperatur null stehen. Lass den Drucker eingeschaltet, bis die Verifizierung abgeschlossen ist. Der Installer bewahrt die Original-Wiederherstellungsdateien unter `~/printer_data/.tltg_thr_firmware_backups/` auf, flasht nur die THR und verifiziert die komplette Anwendung sowie den unveränderten Bootloader, bevor er die Dienste wiederherstellt. Wird der Bootloader nicht unterstützt, bleibt die THR-Firmware unverändert und die zuvor aktiven Dienste werden ohne Firmware-Sperre wiederhergestellt; die Konfigurationsinstallation bleibt erhalten. Der Installer speichert einen privaten, schreibgeschützten Mitschnitt und nennt eine teilbare `report.json`, die du an ein GitHub-Issue anhängen kannst. Hänge die Flash-Binärdateien nicht an. Die zugelassenen Bootloader-Kennungen und die Anforderungen an die Wiederherstellung findest du unter [Nur-THR-Vorgänge](firmware/FLASHING.md).

`--yes` allein und automatische Updates erlauben niemals das Programmieren von Firmware. Um dich ausdrücklich über die Kommandozeile dafür zu entscheiden, verwende `~/tltg-optimized-macros/install.sh --flash-thr-fifo`; ergänze `--dry-run` für eine Vorschau ohne seriellen Zugriff und ohne Änderungen an Diensten. Wiederherstellen der Konfiguration und Deinstallation setzen die MCU-Firmware nicht zurück.

### Automatische Updates

Der Installer fragt, ob stündliche automatische Updates der optimierten Konfiguration aktiviert werden sollen, bevor er fragt, ob Klipper neu gestartet werden soll. Auto-Updates verwenden einen systemweiten systemd-Timer. Zum Aktivieren ist einmalig sudo nötig, um `/etc/systemd/system/tltg-optimized-auto-update.service` und `/etc/systemd/system/tltg-optimized-auto-update.timer` zu installieren. Der Installer verwendet QIDIs öffentlich bekanntes Standard-sudo-Passwort (`qiditech`), sofern die Umgebungsvariable `TLTG_OPTIMIZED_SUDO_PASSWORD` nicht gesetzt ist, und fragt nach einem Passwort, falls der erste sudo-Versuch fehlschlägt.

Existieren diese systemd-Units bereits von einem Dev-Installer, aktualisiert eine Release-Installation sie auf das aktuelle `~/tltg-optimized-macros`-Paket, entfernt Umgebungsvariablen, die die Auto-Update-URL überschreiben, hinterlegt die Prüfsumme des neuesten Releases, wenn GitHub erreichbar ist, und aktiviert bzw. startet den Timer neu.

Jeder stündliche Lauf prüft die Prüfsumme des neuesten GitHub-Releases, setzt aus, solange der Drucker druckt oder pausiert ist, und führt dann den normalen Installer mit Vorabprüfungen und automatischer Bestätigung aus.

Auto-Updates deaktivieren:

```bash
~/tltg-optimized-macros/auto-update.sh --disable-systemd
```

Eine Auto-Update-Prüfung manuell ausführen:

```bash
~/tltg-optimized-macros/auto-update.sh --run
```

### Metadaten-Cache für 3MF-Dateien in Moonraker

Installation und Update patchen erkannten QIDI-Moonraker-Quellcode, sodass unveränderte 3MF-Dateien ihre Metadaten wiederverwenden, statt bei jedem Start neu entpackt zu werden. Das gilt auch, wenn die optionalen Betriebssystem-Optimierungen übersprungen werden. Moonraker startet neu, wenn sich sein File-Manager-Patch ändert; für die Installation muss der Drucker im Leerlauf sein.

Beim ersten Neustart werden die Metadaten vorhandener 3MF-Dateien einmalig neu aufgebaut. Spätere Scans durchlaufen zwar weiterhin die Verzeichnisse, entpacken aber nur neue, geänderte oder ungültig gewordene Archive. Eine Änderung am Metadaten-Extraktor oder an der Richtlinie zur Objektverarbeitung macht den Cache ungültig. Die Druckvorbereitung oder das Entpacken von Archiven, das der Drucker selbst benötigt, wird dadurch nicht umgangen.

Manuelle Neuscans der 3MF-Metadaten verwenden den Archiv-Extraktor und behalten vorhandene QIDI-Vorschaubilder. Ein Neuscan hängt sich an eine bereits laufende Extraktion desselben Archivs an, statt eine weitere zu starten; Fehler werden gemeldet und die bisherigen Metadaten bleiben erhalten. Normale G-Code-Neuscans sind unverändert.

Nicht erkannter Quellcode bleibt erhalten und wird gemeldet. Bei der Deinstallation lässt sich der Originalcode über die Abfrage zur Systemwiederherstellung zurückholen; nach der Installation geänderte Dateien bleiben erhalten.

### QIDI-Box-Filamente in Fluidd

Die QIDI-Firmware `01.01.07.01` speichert Box-Filamente mit neuen IDs wie `QFA90`, Farben als RGB-Werte und Marken wie Bambu, HATCHBOX, Overture und Polymaker über das Filament selbst. Das mitgelieferte Fluidd erwartet aber noch die älteren numerischen IDs: Geladene Slots zeigen `?`, und in Fluidd gewählte Filamente behandelt der Drucker als QIDI PLA Basic.

Installation und Update patchen QIDIs Fluidd, auch wenn die optionalen Betriebssystem-Optimierungen übersprungen werden:

- Box-Slots zeigen das Filament, den Hersteller und die Farbe, die der Drucker tatsächlich verwendet. Ein Slot, der mit einer älteren numerischen ID belegt wurde, zeigt QIDI PLA Basic, weil `01.01.07.01` ihn so behandelt; wähle das Filament erneut, um das zu korrigieren.
- Die Auswahlfelder für Hersteller (Vendor) und Filament entsprechen dem Touchscreen: Wähle zuerst einen Hersteller, dann eines seiner Filamente. Beim Wechsel des Herstellers wird, sofern vorhanden, dessen Filament gleichen Typs ausgewählt.
- In Fluidd gewählte Farben werden genauso gespeichert wie am Touchscreen.
- Die Schaltflächen Load, Unload und Eject der Box-Karte sowie das Auslesen des RFID-Tags senden die Befehle, die `01.01.07.01` verwendet, und Unload ist verfügbar, wenn der gewählte Slot geladen ist.

Ältere unterstützte Firmware behält die ursprünglichen Auswahlfelder und Befehle von Fluidd. Schließe nach der Installation alle Fluidd-Tabs und öffne Fluidd dann erneut; Browser, in denen Fluidd bereits geöffnet war, laden die aktualisierte Seite beim nächsten Aufruf.

Einen Fluidd-Build, den der Installer nicht erkennt, lässt er unverändert. Bei der Deinstallation lässt sich das Original-Fluidd über die Abfrage zur Systemwiederherstellung zurückholen; nach der Installation geänderte Dateien bleiben erhalten.

### Die QIDI-Filamentliste bearbeiten

Unter `01.01.07.01` stammen die Hersteller- und Filamentlisten des Touchscreens aus QIDIs Filamentdateien. Installation und Update fügen den Konfigurationsdateien von Fluidd einen Ordner `qidi_filament_config` hinzu, der diese Dateien öffnet, sodass du Filamente in Fluidd hinzufügen oder anpassen kannst:

- `filament-0.4.json` ist die Liste für die 0,4-mm-Düse; die anderen Dateien decken die übrigen Düsengrößen ab. Klicke mit der rechten Maustaste auf eine Datei und wähle Bearbeiten.
- Um ein Filament hinzuzufügen, kopiere einen Eintrag desselben Materials und gib ihm einen neuen Namen, eine neue, noch nicht verwendete `filament_id` und den Hersteller, unter dem es erscheinen soll. Ein neuer Herstellername fügt diesen Hersteller den Auswahlfeldern hinzu.
- Die Datei muss gültiges JSON bleiben; ein fehlendes Komma oder eine fehlende Klammer kann die Filamentliste des Touchscreens unbrauchbar machen.
- Starte den Drucker nach dem Speichern neu, damit Touchscreen und Box die Änderung übernehmen, und lade Fluidd dann neu.

Firmware-Updates ersetzen diese Dateien, bewahre also eine Kopie aller Einträge auf, die du hinzufügst. Die Dateien in diesem Ordner sind QIDIs eigene Dateien, keine Kopien: Sie sind nicht Teil der Konfigurationssicherungen des Installers, und wenn du eine davon in Fluidd löschst, wird sie auf dem Drucker gelöscht. Existiert `qidi_filament_config` bereits, lässt der Installer es unangetastet. Die Abfrage zur Systemwiederherstellung bei der Deinstallation entfernt nur den Link, niemals deine Änderungen.

### QIDI-Box-Temperatur über Fluidd

Die Zieltemperatur einer QIDI-Box-Heizung stellst du in den Zeilen `Heater Box1`/`Heater Box2` im Temperaturbereich von Fluidd ein. Das höchste erlaubte Ziel ist das niedrigste Box-Temperaturlimit aller Filamente, die in dieser Box geladen sind; eine leere Box erlaubt bis zu 65 °C, das angegebene Maximum der Box. Erlaubt ein geladenes Filament kein Heizen in der Box, ist das Zielfeld gesperrt; eine Box, die bereits heizt, kann trotzdem auf `0` gesetzt werden.

### Vollständige Anzeige in Fluidd

Installation und Update schalten QIDIs Einstellung **Enable Full Display** in Fluidd standardmäßig ein. Sie zeigt die Bettnetz-Karte (Bed Mesh) auf der Tune-Seite und fügt Sensoren ohne Zieltemperatur, etwa den thermischen Kammerschutz und die Elektroniksensoren, dem Temperaturbereich und dem Diagramm hinzu. Wenn du die Einstellung in Fluidd ausschaltest, bleibt das über Updates hinweg erhalten.

### QidiBox-Karte in Fluidd (Vorschau)

Installation und Update fügen dem Dashboard von Fluidd eine **QidiBox**-Karte hinzu, eine Vorschau auf eine neu gestaltete Box-Karte. Die bestehende Box-Karte bleibt unverändert, sodass du beide vergleichen und beliebig nutzen kannst. Bei bestehenden Dashboards landet die Karte unten in der ersten Spalte; im Layout-Modus von Fluidd kannst du sie an eine beliebige Stelle ziehen.

![QidiBox-Karte in Fluidd mit zwei Boxen, Auto-Nachlade-Gruppen, Filamentweg und Werkzeugzuordnung](.github/images/fluidd-qidibox-card.png)

- Die Slots aller Boxen und die externe Spule werden gleichzeitig angezeigt. Jede Spule zeigt oben ihre Farbe und die ihr zugeordneten Werkzeuge; die Slot-Beschriftung darunter erhält einen grünen Rahmen, wenn Filament am Einlass des Slots anliegt. Die Werkzeugbeschriftung des geladenen Slots ist hervorgehoben.
- Unter jeder Box stehen ihre Luftfeuchtigkeit, Temperatur und beim Heizen die Zieltemperatur.
- Der Filamentweg zeigt, wie weit das Filament des gewählten Slots reicht, anhand der Sensoren an Slot-Einlass, Hub und Werkzeugkopf. Hub und Werkzeugkopf leuchten nur für den Slot auf, dessen Filament sich tatsächlich dort befindet.
- Ist in den Box-Einstellungen **Automatic Reload** eingeschaltet, werden Slots, die einander beim Ausgehen des Filaments vertreten können, in ihrer Filamentfarbe umrandet und tragen unter ihrer Beschriftung denselben Nachlade-Buchstaben (↻ A, ↻ B, …): Box-Slots mit demselben Filament, Hersteller und derselben Farbe, so wie die AUTO-Ansicht der Box-Karte sie gruppiert. Der gewählte Slot listet zusätzlich seine Ersatz-Slots auf.
- Klicke auf einen Slot, um ihn auszuwählen, und dann auf **Load**, **Unload**, **Eject**, **Read RFID** oder **Edit filament**. Diese tun dasselbe wie die Schaltflächen und Auswahlfelder der Box-Karte und sind während eines Drucks nicht verfügbar.
- Die Werkzeugzuordnung unten zeigt, welchen Slot jedes Werkzeug lädt, mit einem Diagramm pro Box. Solange der Drucker im Leerlauf ist, klicke auf ein Werkzeug oder auf **Edit**, um die Zuordnung zu ändern; **Default** stellt T0 → 1A, T1 → 1B usw. wieder her. Der Druckdialog von Fluidd legt die Zuordnung für den gestarteten Druck fest, und ein normales Druckende stellt die Standardzuordnung wieder her.
- **Box settings** öffnet dieselben Einstellungen wie **Config** auf der Box-Karte, einschließlich der Lade-/Entladebewegung.

Schließe nach der Installation alle Fluidd-Tabs und öffne Fluidd erneut. Rückmeldungen zum Layout sind in GitHub-Issues willkommen.

### Druckerbildschirm in Fluidd

Installation und Update fügen dem Dashboard von Fluidd eine Karte **Printer screen** hinzu, die den Touchscreen des Druckers live zeigt, sodass du ihn aus einem anderen Raum prüfen oder bedienen kannst. Das gilt auch, wenn die optionalen Betriebssystem-Optimierungen übersprungen werden. Die Karte startet eingeklappt, und der Drucker erledigt keinerlei Bildschirmarbeit, bis du sie aufklappst. Sie stoppt, wenn du sie einklappst, das Dashboard verlässt oder in einen anderen Tab wechselst. Bei bestehenden Dashboards landet die Karte unten in der ersten Spalte; im Layout-Modus von Fluidd kannst du sie an eine beliebige Stelle ziehen.

![Karte „Printer screen“ in Fluidd mit der Startseite des Touchscreens](.github/images/fluidd-printer-screen-view-only.png)

- Der Bildschirm öffnet sich **nur zur Ansicht**: Klicken oder Tippen auf das Bild bewirkt am Drucker nichts.
- Wähle im Kopf der Karte **View only** und bestätige, um die Steuerung einzuschalten. Klicks, Tipps und Ziehbewegungen auf dem Bild drücken dann den echten Touchscreen, auch während eines Drucks. Beim Einklappen der Karte wird die Steuerung wieder ausgeschaltet.
- **Full screen** füllt damit den Bildschirm deines Browsers oder Handys.

![Bestätigung, die angezeigt wird, bevor Fluidd den Touchscreen bedienen darf](.github/images/fluidd-printer-screen-control-confirm.png)

Der Bildschirm steht nur Geräten zur Verfügung, die Fluidd ohnehin schon nutzen können: Der Drucker prüft jede Verbindung über Moonraker, und der Bildschirmdienst selbst lauscht nur auf dem Drucker. Schließe nach der Installation alle Fluidd-Tabs und öffne Fluidd erneut, um die neue Karte zu laden.

Hinter der Karte fügt der Installer einen kleinen Dienst `tltg-remote-screen` und eine Route `/tltg-screen/` in die Webserver-Konfiguration von QIDIs Fluidd ein. Der Dienst sendet nur dann ein neues Bild, wenn sich der Bildschirm ändert. Eine nginx-Konfiguration, die der Installer nicht erkennt, bleibt unverändert, und die Karte meldet dann, dass der Bildschirm nicht verfügbar ist. Bei der Deinstallation lassen sich der Dienst entfernen und die ursprüngliche Webserver-Konfiguration über die Abfrage zur Systemwiederherstellung zurückholen; nach der Installation geänderte Dateien bleiben erhalten.

### Gründe für das Herunterfahren von Klipper

Wenn Klipper wegen eines Fehlers anhielt, etwa einem Heizungs- oder Lüfterfehler oder einer verlorenen Box-Verbindung, konnte QIDIs Druckstatus-Modul Klipper während des Herunterfahrens zum Absturz bringen, sodass Bildschirm, Fluidd und Slicer nur `MCU shutdown` anzeigten. Installation und Update patchen QIDIs `print_stats_manager.py`, damit der ursprüngliche Grund auf dem Bildschirm bleibt; `FIRMWARE_RESTART` stellt wie gewohnt wieder her. Das gilt auch, wenn die optionalen Betriebssystem-Optimierungen übersprungen werden, und Klipper startet einmal neu, wenn der Patch angewendet wird. Nicht erkannter Quellcode bleibt erhalten und wird gemeldet. Bei der Deinstallation lässt sich die Originaldatei über die Abfrage zur Systemwiederherstellung zurückholen; nach der Installation geänderte Dateien bleiben erhalten.

### Überwachung der Box-MCU-Verbindung

Eine Box, die ihre USB-Verbindung verliert, erscheint als `Lost communication with MCU 'mcu_boxN'` ohne Grund für das Herunterfahren, sodass die Ursache standardmäßig nirgends festgehalten wird. Der Installer fügt eine Überwachung hinzu, die **standardmäßig ausgeschaltet** ist. Aktiviere sie über die Konsole von Fluidd:

```gcode
SAVE_VARIABLE VARIABLE=tltg_box_link_monitor VALUE=1
```

Solange sie aktiv ist, schreibt sie `printer_data/logs/tltg_box_link.log`, getrennt von `klippy.log`, damit sie die Log-Rotation übersteht. Sie protokolliert die seriellen Zähler der Box-MCUs und jede Änderung daran, Box-Heizung und Druckkontext, den Reset-Grund, den jede Box-MCU beim Verbinden meldet, sowie die erste Zeile `Got EOF when reading from device`, `Timeout with MCU` oder `shutdown:`, die in `klippy.log` auftaucht. Zu diesem Zeitpunkt werden auch die USB-Meldungen des Kernels erfasst.

Setze die Variable wieder auf `0`, um die Aufzeichnung zu beenden; das wirkt ohne Neustart. Das Log ist auf 2 MB begrenzt und wird einmal nach `tltg_box_link.log.1` rotiert.

Wenn du einen Verbindungsabbruch einer Box-MCU meldest, hänge diese Datei zusammen mit dem Ende von `klippy.log` an.

### Lade- und Entladegeschwindigkeit der QIDI Box

Du kannst einstellen, wie die Box Filament zum Werkzeugkopf schiebt und wieder zurückzieht. Beim Laden schiebt sie den größten Teil des Weges schnell und nähert sich dann langsam, bis der Filamentsensor am Werkzeugkopf auslöst. Installation oder Update speichern die unten aufgeführten Paket-Standardwerte, sofern du noch keine dieser Einstellungen gespeichert hast; andernfalls bleiben alle deine Werte erhalten. Stelle sie auf der Box-Karte in Fluidd unter **Config** → **Load/unload motion** ein oder mit `TLTG_BOX_MOTION` in der Konsole. Führe `TLTG_BOX_MOTION` ohne Parameter aus, um die aktuellen Einstellungen und das Profil jedes Slots zu sehen. Schließe nach der Installation alle Fluidd-Tabs und öffne Fluidd erneut, um die neue Box-Karte zu laden.

| Parameter | Was er einstellt | Paket-Standard | QIDI-Wert | Erlaubt |
|---|---|---|---|---|
| `FAST_LENGTH` | Wie weit der schnelle Vorschub geht, in mm | 950 | 930 | 100–2000 |
| `FAST_SPEED_1` | Geschwindigkeit der ersten schnellen Stufe, in mm/s | 120 | 85 | 10–150 |
| `FAST_SPEED_2` | Geschwindigkeit der zweiten schnellen Stufe, in mm/s | 100 | 65 | 10–150 |
| `APPROACH_SPEED` | Geschwindigkeit für den Rest des Weges bis zum Werkzeugkopf-Sensor, in mm/s | 15 | 10 | 2–40, unter beiden schnellen Geschwindigkeiten |
| `LOAD_ACCEL` | Beschleunigung beim Laden | wie QIDI | 100 | 20–400 |
| `UNLOAD_SPEED_1`, `UNLOAD_SPEED_2` | Geschwindigkeiten der beiden Entladestufen, in mm/s | 115, 120 | 65, 85 | 10–150, müssen sich unterscheiden |
| `UNLOAD_ACCEL` | Beschleunigung beim Entladen | wie QIDI | 100 | 20–400 |
| `UNLOAD_EXTRUDER_LENGTH` | Wie weit der Extruder das Filament zurückzieht, bevor die Box übernimmt, in mm. Nur für alle Slots. | 50 | 120 | 20–300 |

```gcode
TLTG_BOX_MOTION FAST_SPEED_1=100 FAST_SPEED_2=100 APPROACH_SPEED=20
```

Der Befehl lehnt Kombinationen ab, die die Box nicht ausführen kann, und nennt den Grund. Die Einstellungen gelten ab dem nächsten Laden oder Entladen, ohne Neustart. Mit `PARAM=vendor` setzt du eine einzelne Einstellung auf den QIDI-Wert zurück, mit `TLTG_BOX_MOTION RESET=1` kehrst du überall zur QIDI-Bewegung zurück, einschließlich der Slot-Einstellungen. Updates wenden die Paket-Standardwerte danach nicht erneut an.

Jede Einstellung lässt sich auch für einen einzelnen Slot festlegen; sie hat dann Vorrang vor der Einstellung für alle Slots. Um zum Beispiel Slot 2 sanfter zu laden:

```gcode
TLTG_BOX_MOTION SLOT=2 FAST_SPEED_1=60 FAST_SPEED_2=60 APPROACH_SPEED=8
```

Die Slots sind wie bei `LOAD_FILAMENT_CONTROL SLOT=` ab 0 nummeriert, 1A auf der Box-Karte ist also Slot 0 und 2A ist Slot 4. Mit `SLOT=2 FAST_SPEED_1=global` folgt Slot 2 wieder der Einstellung für alle Slots, mit `SLOT=2 RESET=1` löschst du alle Einstellungen von Slot 2.

`FAST_LENGTH` wird genau so verwendet, wie es eingestellt ist. Auf dem Testdrucker erreichte das Filament den Werkzeugkopf-Sensor nach 1001–1016 mm Vorschub, QIDIs 930 mm lassen also etwa 70–85 mm für die langsame Annäherung übrig. Stellst du den Wert über den Sensor hinaus ein, erreicht das Filament den Werkzeugkopf mit voller schneller Geschwindigkeit.

Ein Entladen beginnt damit, dass der Extruder das abgeschnittene Filament 30 mm zurückzieht, dann weitere `UNLOAD_EXTRUDER_LENGTH` mit 10 mm/s, bevor die Box mit deinen Entladegeschwindigkeiten zurückzieht. Auf dem Testdrucker verließ das Filament den Werkzeugkopf-Sensor nach 34 mm Entladen, also nur etwa 4 mm in diesen zweiten Rückzug hinein; von QIDIs 120 mm zieht die Box also größtenteils freies Filament mit 10 mm/s. Der Sensor sitzt oberhalb der Extruderzahnräder, sobald er frei ist, ist das Filament also aus den Zahnrädern heraus. Ein Wert von 20–30 mm lässt einen Sicherheitsabstand hinter dem Sensor und spart etwa 9 Sekunden pro Entladevorgang.

Hast du eine der Einstellungen mit `SAVE_VARIABLE` mit einem ungültigen Wert gespeichert, kehrt das betroffene Laden oder Entladen zur QIDI-Bewegung zurück und die Konsole nennt den Grund. Ändert ein QIDI-Firmware-Update die Art, wie die Box Filament bewegt, auf eine Weise, die die Einstellungen nicht erkennen, behält die Box die QIDI-Bewegung bei und die Konsole meldet das.

### Auswerfen (Eject) an der QIDI Box

QIDIs Eject parkt den Werkzeugkopf vor und nach dem Zurückziehen des Filaments durch die Box über dem Abfallschacht, sodass ein Eject bei einem nicht referenzierten Drucker X und Y referenziert. Wenn beim Eject nur die Box Filament zurückziehen muss, bleibt der Werkzeugkopf jetzt, wo er ist. Das ist der Fall, wenn nichts über den Hub der Box hinaus vorgeschoben ist oder wenn das Filament eines anderen Slots geladen ist. Ein Eject, der zuerst aufheizen und Filament aus dem Weg zum Werkzeugkopf entladen muss, parkt weiterhin so wie bei QIDI. In beiden Fällen schaltet der Eject danach das Hotend aus und deaktiviert den Extrudermotor.

### Hilfreiche Klipper-Werkzeuge

Nach Installation und Neustart von Klipper enthält der optimierte Makrosatz:

```gcode
TLTG_TEST_PROBE CONFIRM=1
TLTG_PROBE_ACCURACY_CENTER
TLTG_CORNER_BED_SCREW_CHECK
SCREWS_TILT_CALCULATE
TLTG_RESET_TOOL_MAPPINGS
```

`TLTG_TEST_PROBE CONFIRM=1` prüft, ob die Wägezelle an der Düse auf leichten Druck reagiert. Der Befehl bewegt nichts, referenziert nicht, heizt nicht und ändert keine Kalibrierung. Gib in den Makro-Steuerelementen von Fluidd im Feld `CONFIRM` eine `1` ein. Der Drucker muss im Leerlauf sein, alle Heizungen aus, Düse und Bett unter 40 °C. Stelle sicher, dass du die Düse erreichst und sie frei über dem Bett steht; **referenziere den Drucker zur Vorbereitung dieses Tests nicht**.

Folge den Anweisungen in der Konsole. Der Test dauert etwa 40 Sekunden:

1. Lass die Düse während `DO NOT TOUCH` und `Measuring...` in Ruhe.
2. Bei `GET READY TO PRESS` mach dich bereit. Bei `PRESS` drücke leicht von unten gegen die Düse und halte den Druck.
3. Wenn du aufgefordert wirst, nicht mehr zu drücken, lass los und lass die Düse während `DO NOT TOUCH` und `Measuring...` in Ruhe.

Das Ergebnis ist eines der folgenden:

- `RESPONSE_DETECTED`: Die Messwerte sehen korrekt aus.
- `NO_RELIABLE_RESPONSE`: Der Sensor hat nicht wie erwartet reagiert.
- `INCONCLUSIVE`: Der Unterschied zwischen Drücken und Nicht-Drücken war zu gering für eine Aussage, oder der Test wurde vorzeitig beendet.

**Eine erkannte Reaktion bestätigt weder den Auslöser fürs Referenzieren noch, dass das Referenzieren sicher ist.** Drücke bei jedem anderen Ergebnis nicht stärker und referenziere oder taste nicht, bis die Ursache geklärt ist. `QUERY_PROBE` kann diesen Sensor nicht prüfen. `TLTG_PROBE_ACCURACY_CENTER` und die Schraubenprüfung unten referenzieren beide, sie sind also keine sicheren Tests bei einem vermutlich defekten Sensor.

`TLTG_PROBE_ACCURACY_CENTER [SAMPLES=20]` referenziert, fährt auf `X195 Y195 Z10` und führt Klippers `PROBE_ACCURACY` aus.

`TLTG_CORNER_BED_SCREW_CHECK` referenziert, führt `Z_TILT_ADJUST` aus und anschließend `SCREWS_TILT_CALCULATE`.

`TLTG_RESET_TOOL_MAPPINGS` setzt die Werkzeug-zu-Slot-Zuordnungen der QIDI Box auf die Grundzuordnung zurück, solange der Drucker im Leerlauf ist. Während eines Drucks oder einer Pause wird der Befehl abgelehnt.

### Bettnetz beim Druckstart

Jeder optimierte Druckstart kalibriert standardmäßig ein frisches adaptives KAMP-Netz. Um bei jedem optimierten Start ein vorhandenes Klipper-Bettnetzprofil wiederzuverwenden, speichere dessen exakten Namen über die Klipper-Konsole:

```gcode
SAVE_VARIABLE VARIABLE=tltg_start_bed_mesh_profile VALUE='"default"'
```

Zur frischen adaptiven Kalibrierung kehrst du zurück, indem du einen leeren Wert speicherst:

```gcode
SAVE_VARIABLE VARIABLE=tltg_start_bed_mesh_profile VALUE='""'
```

Die Einstellung gilt auch für bereits geslicte Dateien und erfordert keine Änderung am Slicer-G-Code. Die Konsole meldet, ob die Startvorbereitung das benannte Profil lädt oder ein frisches adaptives Netz kalibriert. Ein eingestelltes Profil muss bereits existieren; Klipper bricht die Druckvorbereitung ab, wenn es den Namen nicht laden kann.

### Mindesttemperatur der Kammer

Verwende mit OrcaSlicer 2.4.2 oder neuer die aktualisierte `orcaslicer_gcode/start.gcode` und stelle im Filamentprofil die Kammertemperaturen **Target** und **Minimal** ein. Zum Beispiel erlauben Target `60°C` und Minimal `50°C` das Nivellieren und Drucken, sobald die Kammer `50°C` erreicht, während weiter auf `60°C` geheizt wird. Das Filament des ersten Werkzeugs liefert das Minimum; Werte über dem Ziel werden auf das Ziel begrenzt.

Minimal `0` behält das bisherige Warten auf Ziel minus 3 °C bei. Älterer Start-G-Code und QIDI Studio behalten dieses Verhalten auch mit aktualisierten Makros; aktualisierter Orca-Start-G-Code funktioniert auch mit älteren optimierten Makros, die das Minimum ignorieren und ihr bisheriges Warten beibehalten. Für die kürzere Wartezeit müssen sowohl der Start-G-Code als auch die optimierten Makros aktualisiert sein.

Das Minimum gilt auch für die Kammerstufe des gestaffelten Aufheizens. Ein niedrigeres Minimum erlaubt, dass die Düse schon aufheizt, während die Kammer noch auf ihr Ziel heizt.

### Gestaffeltes Aufheizen beim Druckstart

Gestaffeltes Aufheizen ist standardmäßig deaktiviert. So aktivierst du das Aufheizen in der Reihenfolge Bett, Kammer, Düse mit 10 Sekunden Wartezeit zwischen den aktiven Stufen:

```gcode
SAVE_VARIABLE VARIABLE=tltg_staggered_start_heating VALUE=1
SAVE_VARIABLE VARIABLE=tltg_staggered_start_heating_dwell_seconds VALUE=10
```

Stelle die Wartezeit auf eine andere nicht negative Anzahl Sekunden ein, oder verwende `0`, um die Reihenfolge ohne feste Wartezeit beizubehalten. Gestaffeltes Aufheizen deaktivieren:

```gcode
SAVE_VARIABLE VARIABLE=tltg_staggered_start_heating VALUE=0
```

Du musst deine Dateien mit dem neuesten Slicer-G-Code aus diesem Repo (oder aus der Orca Cloud) slicen.

### Filament am Druckende

Der Installer setzt `tltg_keep_loaded_between_prints` auf `1`, wenn die Einstellung noch nicht existiert. Damit bleibt das Filament nach der Installation standardmäßig geladen.

- `1`: Das aktuelle QIDI-Box-Filament bleibt nach einem abgeschlossenen Druck geladen.
- `0` oder nicht gesetzt: Das Filament wird nach einem abgeschlossenen Druck abgeschnitten und entladen.

Um das Geladenlassen zu deaktivieren und das Abschneiden und Entladen wie im Originalzustand zu nutzen:

```gcode
SAVE_VARIABLE VARIABLE=tltg_keep_loaded_between_prints VALUE=0
```

Um das Geladenlassen wieder einzuschalten:

```gcode
SAVE_VARIABLE VARIABLE=tltg_keep_loaded_between_prints VALUE=1
```

Der Installer behält eine vorhandene Einstellung bei Updates bei. Die Einstellung übersteht Klipper-Neustarts, funktioniert mit bereits geslicten Dateien und erfordert keine Änderungen am Slicer-G-Code.

### Filament-Auslaufsensor

```gcode
TLTG_FILAMENT_SENSOR ENABLE=0
TLTG_FILAMENT_SENSOR ENABLE=1
```

`ENABLE=0` lässt die Ereignisse des Filamentsensors am Werkzeugkopf und die Statusmeldung zum Filamentende der externen Spule aktiv, unterdrückt aber das automatische Pausieren bei der externen Spule. `ENABLE=1` stellt das automatische Pausieren bei der externen Spule wieder her. Nach einem Klipper-Neustart ist die Einstellung wieder aktiviert. Die Behandlung von Filamentende an der QIDI Box bleibt in beiden Modi aktiv.

> [!WARNING]
> Ignoriere bei `ENABLE=0` die Warnungen zum Filamentende auf dem Druckerbildschirm. Interagiere nicht mit diesen Warnungen; der Druck läuft weiter, und die Warnung bleibt nur als Statusanzeige des Sensorereignisses bestehen.

Die Konsole zeigt das Auslösen des Werkzeugkopf-Sensors und die aktive Pausenregel an.

![Konsolenausgabe des Filamentsensors](.github/images/tltg-filament-sensor-console.png)

### Maschinen-G-Code für den Slicer aktualisieren

> [!TIP]
> Wenn du Orca ab Version 2.4.0 verwendest, nutze das [geteilte OrcaCloud-Bundle, das du abonnieren kannst](https://cloud.orcaslicer.com/b/4c4b3b74c745)! So bekommst du künftige Updates ganz einfach in deinen Slicer und musst nichts von Hand kopieren und einfügen!

Damit du den optimierten Ablauf nutzen kannst, musst du den Maschinen-G-Code von Hand in den Slicer deiner Wahl kopieren. Der Original-Druckablauf bleibt aus Gründen der Abwärtskompatibilität, der Sicherheit und der allgemeinen Zufriedenheit erhalten :)

Verwende das Paket, das zu deinem Slicer passt. Beide Pakete sind funktional gleich, ihre Platzhalter-Syntax unterscheidet sich aber wegen unterschiedlicher Variablentypen.
   - OrcaSlicer: `orcaslicer_gcode/`
   - QIDI Studio: `qidistudio_gcode/`

## Deinstallation

Wenn `~/tltg-optimized-macros/` noch auf dem Drucker vorhanden ist:

```bash
~/tltg-optimized-macros/install.sh --uninstall --plain
```

Du kannst die Deinstallation auch ausführen, indem du das neueste Skript direkt aus dem Netz lädst:

```bash
/bin/bash -c "$(curl -fsSL https://github.com/thelegendtubaguy/Qidi-Max-4-Optimized/releases/latest/download/install-latest.sh)" -- --uninstall
```

## Wenn etwas schiefgeht

Lies zuerst die Ausgabe des Installers. Der Installer bricht vor jedem Schreibvorgang ab, wenn die Firmware-Erkennung, die Vorabprüfung, der Druckerstatus oder die Prüfung des freien Speicherplatzes fehlschlägt.

Wenn Klipper mit einem Fehler angehalten hat, etwa `... is not a valid config section`, kannst du den Installer trotzdem ausführen. Er installiert zuerst seine Dateien, startet Klipper neu und aktualisiert dann die gespeicherten Einstellungen. Startet Klipper immer noch nicht, meldet der Installer das; behebe den in Fluidd angezeigten Fehler und führe den Installer erneut aus.

Vom Installer angelegte Sicherungen als `.zip`-Dateien liegen unter `/home/qidi/printer_data/` mit den Bezeichnungen `tltg-optimized-macros-before-optimize-...zip` und `tltg-optimized-macros-before-uninstall-...zip`.

Wenn du per SSH mit dem Drucker verbunden bist, kannst du interaktiv wiederherstellen:

```bash
cd ~/tltg-optimized-macros && ./restore.sh
```

Eine bestimmte Sicherung wiederherstellen:

```bash
cd ~/tltg-optimized-macros && ./restore.sh --backup /home/qidi/printer_data/<Name-der-Sicherung>.zip
```

Wenn die Wiederherstellung abgeschlossen ist und die Wiederherstellungsmarkierung (Recovery Sentinel) noch vorhanden ist, entferne sie mit:

```bash
cd ~/tltg-optimized-macros && ./install.sh --clear-recovery-sentinel
```

## Testen

Unter [TESTING](TESTING.md) erfährst du, wie du beim Prüfen von Releases helfen kannst.

## Entwicklung

Die Entwicklungsdokumentation findest du unter [DEVELOPMENT](DEVELOPMENT.md).

## Lizenz

Die im Repository verfassten Inhalte stehen unter der [GPLv3](LICENSE). Komponenten Dritter und deren Lizenztexte sind in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) aufgeführt.
