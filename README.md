# Helferstunden BRK – Erfassung

Kleine Web-App zur Erfassung von Helferstunden (Helfer, Anlass, Datum, Zeit, Bereich).
Läuft komplett im Browser, ohne eigenen Server. Einträge werden direkt in dieses
GitHub-Repo geschrieben (`data/entries.json`) und über GitHub Pages gehostet.

Die App ist bewusst **reines Erfassungsformular** – sie zeigt keine gespeicherten
Einträge, Summen oder Statistiken an. Die Auswertung findet direkt im GitHub-Repo
statt (siehe „Auswertung“ unten), nicht in der Anwendung selbst.

## Wie es funktioniert

- **Erfassen**: Jeder Helfer öffnet die Seite, trägt seine Stunden ein und klickt
  „Eintrag speichern“. Der Eintrag wird sofort per GitHub-API in `data/entries.json`
  geschrieben. Ohne Internet wird er lokal im Browser zwischengespeichert und beim
  nächsten Verbindungsversuch automatisch nachgesendet (alle 30 Sekunden sowie beim
  Wiederherstellen der Verbindung).
- **Token**: Zum *Schreiben* (Eintrag speichern) ist ein GitHub-Zugriffstoken nötig,
  das einmalig pro Gerät/Browser eingetragen wird (Karte „Verbindung einrichten“).
  Das Token wird nur lokal im Browser gespeichert (localStorage) und ausschließlich
  an die GitHub-API gesendet. Es wird auch intern für den Speichervorgang benötigt
  (der aktuelle Stand muss vor dem Schreiben einmal gelesen werden), die App zeigt
  diese Daten aber an keiner Stelle an.
- **Auswertung**: findet **nicht in der App** statt, sondern direkt im GitHub-Repo
  – siehe Abschnitt „Auswertung“ weiter unten.

## ⚠️ Wichtiger Hinweis zur Sicherheit & Datenschutz

- Das Repo ist **öffentlich** (Voraussetzung für kostenloses GitHub Pages). Alle
  Einträge (Namen, Stunden, Anlass, Datum) sind damit für jeden im Internet
  einsehbar. Bitte **keine sensiblen Daten** (Adressen, Gesundheitsdaten, etc.) im
  Feld „Anlass“ eintragen.
- Das gemeinsam genutzte Token gibt Schreibzugriff **nur auf dieses eine Repo**
  (kein Zugriff auf andere Repos oder das GitHub-Konto). Trotzdem gilt: Wer das
  Token besitzt, könnte theoretisch die Datei `data/entries.json` verfälschen oder
  löschen. Da alles über Git versioniert ist, lässt sich das jederzeit über die
  Commit-Historie rückgängig machen (`git revert` bzw. History auf GitHub).
- Token bei Verdacht auf Missbrauch sofort widerrufen und ein neues erstellen (siehe
  unten) – dann allen Helfern das neue Token mitteilen.

## Einmalige Einrichtung (durch die Leitung)

### 1. Repo auf GitHub erstellen und Code hochladen

```bash
cd /Users/timgenkinger/Desktop/Stundenerfassung-BRK
git init
git add .
git commit -m "Initial commit: Helferstunden BRK App"
git branch -M main
```

Dann auf github.com ein neues, **öffentliches** Repository namens
`stundenerfassung-brk` unter dem Konto `timgenkinger` anlegen (ohne README/Lizenz,
da lokal schon vorhanden), danach:

```bash
git remote add origin https://github.com/timgenkinger/stundenerfassung-brk.git
git push -u origin main
```

Beim Push nach Benutzername und Passwort gefragt – als Passwort ein **Personal
Access Token** verwenden (siehe Schritt 2, das gleiche Token funktioniert auch für
den Push).

### 2. Personal Access Token erstellen

1. Auf GitHub: **Settings → Developer settings → Personal access tokens →
   Fine-grained tokens → Generate new token**.
2. Name z. B. `stundenerfassung-brk-shared`.
3. **Repository access**: „Only select repositories“ → `stundenerfassung-brk`
   auswählen (nicht mehr).
4. **Permissions**: „Contents“ → **Read and write**. Alle anderen Berechtigungen auf
   „No access“ lassen.
5. Ablaufdatum nach Bedarf setzen (z. B. 1 Jahr, danach erneuern).
6. Token generieren und **sofort kopieren** (wird nur einmal angezeigt).
7. Dieses Token sicher (z. B. per Signal/verschlüsselt) an alle Helfer weitergeben,
   die Stunden eintragen sollen. Jeder trägt es einmalig in der App unter
   „⚙️ Verbindung zu GitHub einrichten“ ein.

### 3. GitHub Pages aktivieren

1. Im Repo: **Settings → Pages**.
2. **Source**: „Deploy from a branch“.
3. **Branch**: `main`, Ordner `/ (root)`.
4. Speichern. Nach ein bis zwei Minuten ist die App erreichbar unter:

   `https://timgenkinger.github.io/stundenerfassung-brk/`

Diesen Link an alle Helfer weitergeben.

## Laufender Betrieb

- Neue Helfer bekommen einfach den Link + das gemeinsame Token.
- Korrekturen an bestehenden Einträgen (z. B. Tippfehler) können direkt auf GitHub
  in `data/entries.json` vorgenommen werden (Datei im Repo öffnen → Stift-Symbol
  „Edit“ → Wert korrigieren → „Commit changes“).
- Token jährlich erneuern (Ablaufdatum) und neues Token verteilen.

## Auswertung (durch die Leitung, direkt auf GitHub)

Alle Einträge liegen als JSON-Array in
[`data/entries.json`](data/entries.json) im Repo. Möglichkeiten zur Auswertung:

- **Direkt auf GitHub ansehen**: Datei im Repo öffnen, GitHub stellt JSON lesbar
  formatiert dar.
- **Herunterladen & in Excel/Numbers importieren**: Datei über „Raw“ herunterladen
  und z. B. mit einer kurzen Formel/einem Skript oder einem Online-JSON-zu-CSV-
  Konverter in eine Tabelle umwandeln.
- **Am Jahresende**: Datei herunterladen, auswerten (Summen pro Helfer/Bereich/Jahr)
  und die Zahlen manuell oder per Skript ins DRK-System übertragen.
- **Historie/Änderungen nachvollziehen**: Die Commit-Historie der Datei auf GitHub
  zeigt jeden einzelnen Eintrag mit Zeitstempel.

## Lokale Vorschau vor dem Hochladen

```bash
cd /Users/timgenkinger/Desktop/Stundenerfassung-BRK
python3 -m http.server 8080
```

Dann `http://localhost:8080` im Browser öffnen. Das Speichern eines Eintrags
funktioniert erst, nachdem der Code auf GitHub liegt und ein gültiges Token
eingetragen wurde.
