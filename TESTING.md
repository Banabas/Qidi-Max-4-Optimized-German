# Testen

Ich würde mich sehr über deine Hilfe freuen, das hier besser zu machen und Grenzfälle zu testen! Wenn du beim Testen helfen möchtest, findest du hier, was du wissen musst und was ich von dir brauche.

> [!CAUTION]
> Ich übernehme keine Verantwortung für deinen Drucker. Beobachte BITTE den Start jedes Drucks, damit der Drucker nichts Schlimmes anstellt, etwa dein Druckbett zerkratzt, und sei jederzeit bereit, den Strom abzuschalten.

1. Stelle sicher, dass du den neuesten Slicer-G-Code aus diesem Repo für deinen Slicer hast (hol ihn aus dem `dev`-Branch, wenn du den übrigen Anweisungen folgst).
2. Führe diesen Befehl auf deinem Drucker aus, um die neueste Vorabversion zu installieren (kopiere und füge alles in einem Rutsch ein):
```bash
TLTG_INSTALLER_ARCHIVE_URL=https://github.com/thelegendtubaguy/Qidi-Max-4-Optimized/releases/download/prerelease/tltg-optimized-macros-dev.tar.gz \
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/thelegendtubaguy/Qidi-Max-4-Optimized/dev/installer/release/install-latest.sh)"
```
3. Slice deinen Druck wie gewohnt, aber notiere dir, welche Einstellungen du bei Maschinen- und Filamentprofilen bzw. bei der Auswahl verwendet hast.
4. Senden, drucken und zuschauen.
5. Berichte, wie es gelaufen ist!

## Rückmeldung geben
Ich möchte von dir hören, auch wenn alles perfekt gelaufen ist! So kann ich einschätzen, ob die Sache auch bei anderen Leuten als mir gut funktioniert.

Am liebsten sammle ich Rückmeldungen hier auf GitHub, indem du ein Issue eröffnest. Du kannst dich auch im [TubaMakes-Discord](https://l.tubaguy.com/tmdiscord) dazu austauschen!
