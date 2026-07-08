# AI-Design-Guide — LoL OBS Overlay

Du (die AI) sollst das Design dieses League-of-Legends-Stream-Overlays
ändern. Diese Datei liegt im GitHub-Design-Repo neben der `overlay.html`.
**So läuft es:** Datei ändern → committen/pushen → fertig. Eine lokale EXE
beim Nutzer zieht sich alle Dateien dieses Repo-Ordners automatisch alle
30 s, und die Overlay-Seite in OBS lädt sich dann von selbst neu. Es gibt
keinen Build, kein Deployment, nichts weiter zu tun.

## Wo was geändert wird (schnellster Weg zuerst)

1. **Farben/Größen/Styles** → `custom.css` (anlegen, falls nicht da).
   Wird NACH den eingebauten Styles geladen und überschreibt sie —
   `!important` nur wenn nötig. Ideal für kleine, saubere Tweaks.
2. **Verhalten/Text/Logik nachrüsten** → `custom.js` (anlegen, falls
   nicht da). Läuft nach dem Overlay-Code; alle Funktionen und Variablen
   sind global erreichbar (`render`, `setPreset`, `PRESETS`, `TIER_SKIN`,
   `applyView`, `lastData` ...). Bestehende Funktionen kann man
   überschreiben/wrappen.
3. **Strukturelle Umbauten** (neue Elemente, Layouts, Presets) →
   direkt in `overlay.html`.
4. **Bilder/Fonts** → einfach als Datei ins Repo legen und relativ
   referenzieren (`<img src="/mein-logo.png">`). Erlaubte Endungen:
   .html .css .js .json .png .jpg .jpeg .gif .svg .webp .woff .woff2 .txt
   — max. 30 Dateien, max. 5 MB pro Datei.

## Aufbau der overlay.html

- **CSS-Variablen** in `:root` (Zeilenbereich ganz oben): `--glass`,
  `--gold`, `--win`, `--loss`, `--bar1..3` usw. — fast alle Farben hängen
  daran. Skins (`body.skin-ice`, `.skin-neon`, `.skin-diamond`,
  `.skin-master`) überschreiben nur diese Variablen.
- **Layout-Klassen auf `<body>`**: `small` (kleine Karte), `clean`
  (Chips-Panel), `layout-bar` (Mini-Bar) — gesteuert über `applyView()`.
- **8 Presets** (`PRESETS`-Objekt im JS): 1/2 Gold groß/klein, 3/4 Ice,
  5/6 Neon, 7 Clean Chips, 8 Mini Bar. OBS-URL wählt mit `?preset=N`.
- **Auto-Skin**: Farben folgen dem Rang (`TIER_SKIN`), `?autoskin=0`
  schaltet ab. **Auto-Klein im Match**: `?autosmall=0` schaltet ab.
- **Daten**: `poll()` holt alle 5 s `/api/data` und ruft `render(d)`.

Felder von `/api/data` (alles, was anzeigbar ist):

```
ok, error                      — Status; error als Text anzeigen
header_num, header_text        — "5" + "WINS TILL SILVER"
tier, rank_label, lp           — "gold", "Gold II", 53
progress                       — 0-100 (% im Tier, für den Balken)
left_label/left_tier,
right_label/right_tier         — Balken-Beschriftung links/rechts
last5[]                        — {champ_id, champ, win, remake, lp_delta}
                                 lp_delta kann null sein (kein Badge)
session                        — {wins, losses, winrate, net_lp, games}
grind_seconds, ingame_seconds  — Timer (Sekunden)
season_wins, season_losses     — Gesamt-Saison
in_game                        — läuft gerade ein Match
win_streak                     — Siege in Folge
html_hash                      — intern für Auto-Reload, nicht anzeigen
```

Eingebaute Assets vom lokalen Server: `/assets/ranks/<tier>.png`
(iron..challenger), `/assets/champs/<champ_id>.png`,
`/assets/fonts/montserrat.woff2`.

## Harte Regeln (nicht brechen)

- **Niemals externe URLs** (CDNs, Google Fonts, Bild-Hoster) — das
  Overlay muss offline funktionieren. Alles kommt aus diesem Repo oder
  von `/assets/`.
- **Nicht entfernen:** die Funktion `checkHtmlHash` und ihren Aufruf in
  `render()` (Auto-Update), die `poll()`-Schleife, die beiden Haken
  `<link href="/custom.css">` und `<script src="/custom.js">`.
- **OBS-Rahmenbedingungen:** Browser-Quelle ist ~320 px breit; `html/body`
  bleiben transparent (`background: transparent`), `overflow: hidden`,
  keine Scrollbars, Karte ~300 px breit.
- Die `lp_delta`-Badges: `null` = gar nichts anzeigen (nicht "0").
- `?preset=`-Nummern und die JSON-Feldnamen nicht umbenennen — sie sind
  Vertrag mit der EXE und mit bestehenden OBS-Szenen.
- Nichts löschen, was du nicht ersetzt: Nutzer streamen live damit;
  ein kaputtes HTML ist sofort im Stream sichtbar.

## Testen vor dem Push

Läuft die EXE lokal: `http://127.0.0.1:8077/?demo=1&preset=2` zeigt die
Seite mit festen Demo-Daten (auch `preset=1..8` durchklicken; mindestens
ein großes, ein kleines Layout und die Mini-Bar prüfen). Ohne EXE: HTML
zumindest auf Syntax/Struktur prüfen — im Zweifel klein committen, der
Rollback ist ein `git revert` und ist nach 30 s live.
