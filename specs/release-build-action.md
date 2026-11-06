# Release-Build via GitHub Action

## Warum
Ein nicht-technischer Nutzer soll signierte AAPS-APKs (fullRelease) erstellen können, ohne lokale
Build-Umgebung. Lösung: GitHub Action in einem Fork. Die Action baut standardmäßig den Commit, der
upstream (nightscout/AndroidAPS) als letztes als Release getaggt wurde; optional kann ein Tag im
Startdialog angegeben werden. Der Fork selbst muss dafür nicht aktuell gehalten werden.

## Was die Action tut (`build-release.yml`)
1. Ermittelt den zu bauenden Tag: Eingabefeld `tag` falls ausgefüllt, sonst neuester Release-Tag
   der Upstream-API.
2. Checkt nightscout/AndroidAPS an diesem Tag aus (nicht den Fork).
3. Baut `assembleFullRelease` mit JDK 21.
4. Signiert die APK mit dem Keystore aus den Repo-Secrets.
5. Erstellt ein GitHub Release im Fork und hängt die APK an.

## Benötigte Secrets im Fork-Repo
| Secret | Inhalt |
|---|---|
| `KEYSTORE_BASE64` | Keystore-Datei, base64-kodiert (`base64 -w0 release.jks`) |
| `KEYSTORE_PASSWORD` | Keystore-Passwort |
| `KEY_ALIAS` | Key-Alias |
| `KEY_PASSWORD` | Key-Passwort |

## Einrichtung (einmalig)
1. Fork von nightscout/AndroidAPS anlegen, Branch `chore/release-build-action` mergen.
2. Secrets im Fork hinterlegen (Settings → Secrets and variables → Actions).
3. Nutzer als Collaborator mit `write` hinzufügen.

## Bedienung
1. `github.com/<fork>/actions` → Workflow **Ullis Signed Release** → *Run workflow*.
2. Optional: Feld `tag` mit gewünschter Version füllen (z.B. `3.4.2.6`); leer lassen = neueste.
3. Warten bis Lauf grün.
4. Fork-Startseite → *Releases* → APK herunterladen.

## Haltbarkeit
- Neue AAPS-Versionen, die JDK 21 benötigen (ab 3.3), funktionieren ohne Anpassung. Für künftige
  JDK-Wechsel muss `java-version` im Workflow angepasst werden (Upstream-Referenz:
  `.github/jdk-map.json`).
- Keystore und Passwörter nur in den GitHub-Secrets; nirgends sonst ablegen. Verlust des Keystores
  bedeutet, dass Updates nur nach Deinstallation der App installiert werden können.
- AAPS-APKs werden bei Android-Updates nur akzeptiert, wenn die Signatur identisch bleibt —
  derselbe Keystore muss für alle Builds erhalten bleiben.
