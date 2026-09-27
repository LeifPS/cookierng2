# Cookie RNG 2

Browserspiel, gehostet auf **Firebase Hosting**. Rangliste, Live-Feed und Spielstand-Sync laufen über Firebase (anonyme Anmeldung, Google-Anmeldung und Firestore). Ohne Firebase läuft das Spiel offline weiter, nur ohne Online-Funktionen.

```
cookie-rng-2/
├─ public/               ← wird veröffentlicht
│  ├─ index.html         ← das komplette Spiel
│  ├─ firebase-config.js ← deine Firebase-Werte (schon eingetragen: Projekt cookie-rng)
│  └─ favicon.svg
├─ firebase.json         ← Hosting- und Regel-Konfiguration
├─ .firebaserc           ← verknüpft den Ordner mit dem Projekt cookie-rng
└─ firestore.rules       ← Sicherheitsregeln für die Datenbank
```

## 1. Firebase-Konsole (einmalig)

Auf https://console.firebase.google.com im Projekt **cookie-rng**:

1. **Build → Authentication → Loslegen → Anmeldemethode**: **Anonym** und **Google** aktivieren (bei Google eine Support-E-Mail auswählen).
2. **Build → Firestore Database → Datenbank erstellen**: Produktionsmodus, Standort z. B. `eur3 (europe-west)`.

Die Domains `cookie-rng.web.app` und `cookie-rng.firebaseapp.com` sind automatisch für die Anmeldung freigegeben.

## 2. Veröffentlichen

Du brauchst einmalig Node.js (https://nodejs.org). Dann im Ordner `cookie-rng-2` im Terminal:

```
npm install -g firebase-tools
firebase login
firebase deploy
```

`firebase deploy` lädt die Seite **und** die Datenbankregeln hoch. Danach läuft das Spiel unter:

**https://cookie-rng.web.app**

Für spätere Updates reicht wieder `firebase deploy` (oder `firebase deploy --only hosting`, wenn sich nur das Spiel geändert hat).

Eigene Domain: Konsole → **Hosting → Benutzerdefinierte Domain hinzufügen**, danach die Domain zusätzlich unter **Authentication → Einstellungen → Autorisierte Domains** eintragen.

## 3. Testen

Seite öffnen → Tab **Rangliste** → Namen eingeben → **Beitreten**. In der Konsole unter Firestore erscheint die Sammlung `scores`. Fehler siehst du in den Entwicklertools des Browsers (F12 → Konsole).

Lokal testen: `firebase serve` oder `firebase emulators:start --only hosting` und `http://localhost:5000` öffnen.

## Auf einem anderen Gerät weiterspielen

Im Spiel unter **Einstellungen → Auf anderem Gerät weiterspielen** (oder über den Knopf ganz unten):

- **Mit Google anmelden:** Der Spielstand wird mit dem Google-Konto verknüpft. Auf dem anderen Gerät ebenfalls anmelden, dann wird der Stand geladen. Beim Wechsel zwischen Geräten lädt das Spiel automatisch den neuesten Stand. Gibt es auf beiden Seiten Fortschritt, fragt das Spiel, welcher bleiben soll.
- **Spielstand-Code (ohne Konto):** Code kopieren, auf dem anderen Gerät einfügen.

Nicht gleichzeitig auf zwei Geräten spielen: Es gewinnt der zuletzt gespeicherte Stand.

## Limits im kostenlosen Spark-Plan

Solange du im Spark-Plan bleibst, kann **nichts kosten**. Wird ein Limit erreicht, hört die jeweilige Funktion bis zum Reset auf zu gehen, es entsteht keine Rechnung.

| Dienst | Kostenlos | Was das Spiel verbraucht |
|---|---|---|
| Hosting-Speicher | 10 GB | ca. 0,15 MB |
| Hosting-Datentransfer | 10 GB pro Monat | ca. 45 KB pro Seitenaufruf (komprimiert) → rund 200.000 Aufrufe/Monat |
| Firestore lesen | 50.000 pro Tag | Seitenaufruf ca. 20, Rangliste öffnen bis zu 100 (höchstens alle 2 min) |
| Firestore schreiben | 20.000 pro Tag | pro aktivem Spieler ca. 12–30 pro Stunde |

Beim Hosting-Datentransfer gibt Firebase nach Überschreitung eine Karenzzeit und schaltet die Seite dann bis zum Monatsende ab. Bei Firestore schlagen Anfragen bis zum Tagesreset fehl, das Spiel selbst läuft weiter (lokal gespeichert).

Grob reicht das für einige Dutzend bis wenige Hundert aktive Spieler pro Tag. Den Verbrauch siehst du in der Konsole unter **Nutzung und Abrechnung**. Wird es knapp, kannst du auf den Blaze-Plan wechseln (Hosting: 0,15 $ pro zusätzlichem GB Transfer) und dort ein Budget-Limit mit Warnung setzen.

## Was wo gespeichert wird

| Sammlung | Inhalt | Wer darf |
|---|---|---|
| `scores/{uid}` | Ranglisten-Eintrag | alle lesen, jeder nur den eigenen schreiben |
| `feed/{uid}_0…7` | Live-Funde (max. 8 pro Spieler) | alle lesen, jeder nur die eigenen schreiben |
| `saves/{uid}` | Spielstand für Sync und Backup | nur der Spieler selbst |

## Gut zu wissen

- **Schummeln:** Das Spiel läuft komplett im Browser, deshalb kann ein technisch versierter Spieler seine Werte fälschen. Die Regeln prüfen nur Format und Grenzen. Einträge kannst du in der Konsole unter `scores` löschen.
- **Ofen-Events** werden aus der Uhrzeit berechnet: Alle Spieler sehen dieselben Events zur selben Zeit, ohne Server.
