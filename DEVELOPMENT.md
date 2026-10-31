# Den dev-Branch verwenden

Im `dev`-Branch landen neue Änderungen, bevor sie in ein Release kommen. Jeder Push auf `dev` baut einen frischen Installer und veröffentlicht ihn im Release [`prerelease`](https://github.com/thelegendtubaguy/Qidi-Max-4-Optimized/releases/tag/prerelease) auf GitHub, sodass du den neuesten Stand auf deinem Drucker nutzen kannst, ohne selbst etwas zu bauen.

Der Haken: Er wurde nicht so ausgiebig getestet wie ein Release. An jedem beliebigen Tag kann etwas halb fertig oder schlicht kaputt sein. Wenn du einfach nur einen funktionierenden Drucker willst, nutze stattdessen die normale Installation aus der [README](README.md).

> [!CAUTION]
> Ich übernehme keine Verantwortung für deinen Drucker. Beobachte BITTE den Start jedes Drucks, damit der Drucker nichts Schlimmes anstellt, etwa dein Druckbett zerkratzt, und sei jederzeit bereit, den Strom abzuschalten. Für dev gilt das doppelt.

## Installation

1. Verbinde dich per SSH mit deinem Drucker: `ssh qidi@<IP deines Druckers>`
2. Füge das hier ein und führe es aus. Es ist ein einziger Befehl, der auf zwei Zeilen verteilt ist, kopiere also beide:

   ```bash
   TLTG_INSTALLER_ARCHIVE_URL=https://github.com/thelegendtubaguy/Qidi-Max-4-Optimized/releases/download/prerelease/tltg-optimized-macros-dev.tar.gz \
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/thelegendtubaguy/Qidi-Max-4-Optimized/dev/installer/release/install-latest.sh)"
   ```

3. Folge den Abfragen. Sie sind dieselben wie bei einer normalen Installation, mit einer Ausnahme: **Beantworte die Frage nach automatischen Updates mit Nein.** Auto-Updates folgen nur stabilen Releases; sind sie eingeschaltet, rutscht dein Drucker unbemerkt auf die stabile Version zurück.
4. Aktualisiere deinen Slicer. Kopiere den Maschinen-G-Code aus dem `dev`-Branch, damit er zu den gerade installierten Makros passt: [`orcaslicer_gcode/`](https://github.com/thelegendtubaguy/Qidi-Max-4-Optimized/tree/dev/orcaslicer_gcode) für OrcaSlicer oder [`qidistudio_gcode/`](https://github.com/thelegendtubaguy/Qidi-Max-4-Optimized/tree/dev/qidistudio_gcode) für QIDI Studio.

Um später neuere dev-Änderungen zu bekommen, führe denselben Befehl erneut aus. Mach dasselbe mit dem Slicer-G-Code, falls er sich geändert hat.

### Auto-Updates sind schon eingeschaltet?

Schalte sie vor (oder direkt nach) der dev-Installation aus, sonst ersetzt das nächste stabile Release deine dev-Installation:

```bash
~/tltg-optimized-macros/auto-update.sh --disable-systemd
```

### Erst mal schauen, bevor sich etwas ändert?

Hänge `-- --dry-run` an das Ende des Befehls an. Der Installer zeigt dann, was er tun würde, ohne etwas zu schreiben:

```bash
TLTG_INSTALLER_ARCHIVE_URL=https://github.com/thelegendtubaguy/Qidi-Max-4-Optimized/releases/download/prerelease/tltg-optimized-macros-dev.tar.gz \
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/thelegendtubaguy/Qidi-Max-4-Optimized/dev/installer/release/install-latest.sh)" -- --dry-run
```

## Zurück zur stabilen Version

Führe den normalen Installationsbefehl aus der [README](README.md) aus. Er ersetzt die dev-Installation durch das neueste Release, und dabei kannst du die Auto-Updates wieder einschalten, wenn du sie möchtest.

Um stattdessen alles zu entfernen:

```bash
~/tltg-optimized-macros/install.sh --uninstall --plain
```

Sicherungen, Wiederherstellung und Recovery funktionieren genauso wie bei einem Release. Siehe [Wenn etwas schiefgeht](README.md#wenn-etwas-schiefgeht) in der README.

## Sag mir, wie es gelaufen ist

Ob gut oder schlecht, ich will es hören. In [TESTING.md](TESTING.md) steht, was nützlich ist mitzuschicken. Eröffne ein Issue hier auf GitHub oder komm zum Plaudern in den [TubaMakes-Discord](https://l.tubaguy.com/tmdiscord).

## Am Code arbeiten

Der Rest dieser Seite richtet sich an Leute, die das Repository selbst ändern.

### Einen lokalen Build auf deinem Drucker testen

Das baut ein dev-Paket aus deinem Checkout, kopiert es auf den Drucker und führt dort einen Testlauf aus:

```bash
scripts/build_copy_dry_run.sh --host <Drucker-IP>
```

Standardmäßig verbindet es sich als `qidi`. Mit `--user <Name>` änderst du das, mit `--password '<Passwort>'` gibst du ein Passwort an, falls du keine SSH-Schlüssel nutzt (dafür muss `sshpass` lokal installiert sein), und `--debug` liefert zusätzliche Installer-Ausgaben.

### Prüfungen vor dem Öffnen eines PRs

Führe die aus, die zu deinen Änderungen passen:

```bash
# Installer-Verhalten
python3 scripts/run_installer_core_tests.py

# installer/package.yaml oder installer/supported_upgrade_sources.yaml
python3 scripts/check_installer_known_versions.py

# Slicer-G-Code in orcaslicer_gcode/ oder qidistudio_gcode/
python3 scripts/check_optimized_slicer_macros.py

# Alles in installer/klipper/tltg-optimized-macros/
python3 scripts/format_klipper_configs.py

# Änderungen an Paket oder Launcher: dev- und Release-Paket bauen und Smoke-Test ausführen
python3 scripts/build_installer_bundle.py --output-dir dist --channel dev --build-id local --smoke-test
python3 scripts/build_installer_bundle.py --output-dir dist --channel release --smoke-test
```

### THR-Firmware

Das Bauen und Prüfen des LIS2DW-FIFO-Fixes für den Max 4 ist in [`firmware/README.md`](firmware/README.md) beschrieben. Die Firmware ist vom Makro-Installer-Paket getrennt, und Bauen oder Prüfen flasht niemals einen Drucker.

### Den Installer auf einen simulierten Drucker richten

`installer/runtime/cli.py` liest diese Umgebungsvariablen, damit Tests gegen ein Fixture-Verzeichnis statt gegen einen echten Drucker laufen können:

- `TLTG_OPTIMIZED_PRINTER_DATA_ROOT`
- `TLTG_OPTIMIZED_FIRMWARE_MANIFEST`
- `TLTG_OPTIMIZED_MOONRAKER_URL`

`installer/tests/helpers.py` und `scripts/smoke_test_installer_bundle.py` nutzen beide diese Variablen.
