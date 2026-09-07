# Rostock Life – Layout-Analyse

Analyse der Startseite von **Rostock Life – Das Stadtmagazin für Rostock**
auf Basis des MHT-Snapshots vom **7. September 2026**
(Quelle: `https://rostock-life-magazin.svenkrukau1966.chatgpt.site/`).

Diese Datei ist als Arbeitsgrundlage gedacht: Sie beschreibt das Layout so vollständig,
dass es sich lokal ohne Zugriff auf das Original weiterbauen oder in ein anderes
System (statisches HTML, Astro, WordPress-Theme) überführen lässt.

**Dateien in diesem Ordner**

| Datei | Inhalt |
|---|---|
| `layout-analyse.md` | dieses Dokument |
| `rostock-life-custom.css` | die projekteigenen Styles, aus dem Build-CSS herausgelöst (409 Regeln, ohne Tailwind-Ballast) |
| `rostock-life-snapshot.html` | das gespeicherte DOM der Startseite (ohne Skripte, Rubriken zugeklappt) |

> **Stand der Quellen:** Der MHT-Snapshot ist vollständig ausgewertet. Die Live-Seite und der
> ChatGPT-Share-Link konnten aus dieser Session heraus **nicht** geladen werden – der Egress-Proxy
> blockiert beide Hosts (HTTP 403 beim CONNECT). Alles Folgende stammt daher aus dem Snapshot,
> nicht aus dem Live-Zustand.

---

## 1. Technischer Aufbau

| Aspekt | Befund |
|---|---|
| Framework | React mit React Server Components, Vite/Rolldown-Build (`assets/framework-*.js`, `rolldown-runtime-*.js`, `layout-segment-context-*.js`) |
| Styling | **eine** handgeschriebene CSS-Datei, gebündelt mit Tailwind v4 Preflight + Theme-Layer. Die Tailwind-Utilities werden im Markup praktisch **nicht** genutzt – das Layout ist klassisches CSS mit sprechenden Klassennamen (`.weekend`, `.topic-grid`, `.rubric-shell` …). Nur `body.antialiased` stammt aus Tailwind. |
| Icons | `lucide-react` (`createLucideIcon-*.js`), inline als SVG gerendert |
| Dynamik | zwei eigene Chunks: `weather-widget-*.js` (Live-Wetter über **Open-Meteo**) und `rubric-*.js` (Akkordeon-Logik) |
| Sprache | `<html lang="de">` |
| CSS-Gesamtgröße | 189 KB gebündelt, davon **≈ 45 KB projekteigen** (Zeilen 1401–2156 der Build-Datei) |

**Konsequenz für die Weiterarbeit:** Das gesamte Layout hängt an genau einem Stylesheet und an
semantischem HTML. Es ist ohne React reproduzierbar – React liefert hier nur Wetter-Widget und
Akkordeon, und das Akkordeon ist ohnehin natives `<details>/<summary>`.

---

## 2. Seitenarchitektur

Reihenfolge im DOM (alles innerhalb von `<main id="top">`, durchgehend **randlos/full-bleed** –
es gibt keinen zentrierten Container mit `max-width`):

| # | Element | Klasse | Anker | Funktion |
|---|---|---|---|---|
| 1 | `div` | `.topline` | – | rote Servicezeile: „ROSTOCK AKTUELL“, Claim, Instagram-Link rechts |
| 2 | `header` | `.site-header` | – | Logo, 5 Hauptnavigationspunkte, Dropdown „Bereiche“ (`.section-jump`, 12 Ziele) |
| 3 | `section` | `.weekend` | `#termine` | Hero-Foto + rote „10 Tipps“-Spalte (`aside.today-panel`) |
| 4 | `section` | `.weather` | `#wetter` | Live-Wetter + 4-Tage-Vorhersage |
| 5 | `aside` | `.brotzeit-ad` | – | Werbebanner Brotzeit (als „ANZEIGE“ gekennzeichnet) |
| 6 | `div` | `.quicknav` | – | horizontal scrollende Sprungleiste „Direkt zu:“ (10 Ziele) |
| 7 | `section` | `.section .topics-section` | `#themen` | 6 Themen-Kacheln (`.topic-grid`) |
| 8–18 | `details` | `.rubric-shell` | siehe 2.1 | 11 aufklappbare Rubriken |
| 19 | `section` | `.pharmacy-section` | `#apotheken` | Apotheken-Notdienst |
| 20 | `section` | `.magazine-section` | `#magazin` | aktuelle Print-Ausgabe (Yumpu-Link) |
| 21 | `section` | `.advertise` | `#werben` | Werbe-/Mediadaten-Block |
| 22 | `footer` | – | – | Logo, Claim, Rechtslinks, Copyright |

Zusätzlich global: `a.scroll-top` (runder Zurück-nach-oben-Button, `position: fixed`, unten rechts).

### 2.1 Die elf Rubriken (`<details class="rubric-shell">`)

| Anker | Kicker | Headline | Farbvariante |
|---|---|---|---|
| `#monatskalender` | September 2026 | Der Monat in Rostock. | `.rubric-dark` |
| `#nachtleben` | Nachtleben | Rostock nach Sonnenuntergang. | `.rubric-dark` |
| `#sport` | Rostock Sport | Vier Teams. Eine Sportstadt. | `.rubric-blue` |
| `#kino` | Kino in Rostock | Jetzt auf der großen Leinwand. | `.rubric-light` |
| `#theater` | Bühne & Theater – September | Der aktuelle Monatsplan. | `.rubric-red` |
| `#cafes` | Gastronomie – Cafés | Cafés, Frühstück und Bäckereien. | `.rubric-light` |
| `#restaurants` | Restaurants – Anzeigenpartner | Rostocker Restaurantempfehlungen. | `.rubric-dark` |
| `#restaurantfuehrer` | Weitere Restaurants – Kostenloser Service | Restaurantführer Rostock. | `.rubric-light` |
| `#ausfluege` | Ausflüge & Freizeit | Rauskommen – bei jedem Wetter. | `.rubric-light` |
| `#shopping` | Shopping | Einkaufszentren in und um Rostock. | `.rubric-blue` |
| `#maerkte` | Rostocker Wochenmärkte | Frisch einkaufen im Stadtteil. | `.rubric-light` |

**Wichtig:** Beim Speichern des MHT waren alle `<details>` geschlossen. Die Rubrik-**Inhalte**
stehen deshalb nicht im Snapshot-Markup – wohl aber ihre Styles. Was in den Klappen steckt,
lässt sich vollständig aus dem CSS rekonstruieren (Abschnitt 5).

---

## 3. Designsystem

### 3.1 Farben

Als CSS-Variablen auf `:root` definiert (shadcn-Namensschema), im Layout aber meist als
Literalwerte verwendet:

```css
--background: #f2f0eb;  --foreground: #111923;
--card: #ffffff;        --muted: #e7e3dc;   --muted-foreground: #65707a;
--accent: #aa2937;      --border: #d8d3cb;  --radius: .5rem;  /* faktisch ungenutzt */
```

Die tatsächlich tragende Palette:

| Farbe | Hex | Rolle |
|---|---|---|
| Rostock-Rot | `#AA2937` | Leitfarbe: Topline, Tipps-Panel, Kicker, Buttons, Kartenkanten, Icons, Links |
| Nachtblau | `#111923` | Grundfarbe für Text und dunkle Flächen (Hero-Grund, Themenkacheln, Werbeblock, Kalender) |
| Sand | `#F2F0EB` | Seitenhintergrund |
| Weiß | `#FFFFFF` | Karten, Header, Quicknav, helle Sektionen |
| Petrol | `#192D3C` | Sekundärfläche: Event-, Restaurant-, Sport-, Shopping-Sektion |
| Dunkelrot | `#8F2330` | Theater-Sektion |
| Eisblau | `#D9ECF7` (Rand `#BED8E7`) | Wetterleiste |
| Mint | `#E3EFE9` | Apotheken-Notdienst |
| Beige/Nacht | `#EEE9E1` / Karte `#151321` | Nachtleben |
| Salbei | `#EEF2F0` | Ausflüge |
| Warmgrau | `#E5E1DA` | Magazin-Sektion |
| Tiefschwarz | `#090F15` | Footer |
| Brotzeit-Gold | `#C8974F` auf `#101820`, Fließtext `#F0DAC0` | ausschließlich in der Brotzeit-Anzeige |
| Rosé-Akzente | `#FFBAC2`, `#FFD5DA`, `#F1CBD0` | Kicker und Sekundärtext **auf** rotem Grund |

Die Sektionsfarben wechseln bewusst hart – jede Rubrik bekommt eine eigene Grundfläche.
Das ersetzt Abstände als Gliederungsmittel.

### 3.2 Typografie

| Ebene | Werte |
|---|---|
| Schriftfamilie | `Arial, Helvetica, sans-serif` – **kein Webfont**. Ausnahme: `Georgia, serif` **nur** in der Brotzeit-Anzeige (Markenzitat). |
| Hero-H1 | `clamp(3.4rem, 7vw, 7rem)` / `line-height: .86` / `letter-spacing: -.075em` |
| Sektions-H2 | `clamp(2.5rem, 5vw, 4.9rem)` / `.96` / `-.06em` |
| Rubrik-H2 | `clamp(2rem, 4vw, 4rem)` / `.94` / `-.055em` |
| Kicker | `.74rem`, `font-weight: 900`, `text-transform: uppercase`, `letter-spacing: .18em`, rot bzw. rosé auf dunklem Grund |
| Karten-H3 | 0.95–1.6rem, `letter-spacing: -.035em` |
| Fließtext | 0.8–1.12rem, `line-height` 1.5–1.7, Grau `#65707A`/`#6D757C` |
| Kleinstschrift | 0.55–0.7rem für Quellen, Zeiten, Hinweise |
| Auszeichnung | durchgehend `font-weight: 800/900`; es gibt praktisch kein `500`/`600` |

Die Typo-Handschrift ist damit eindeutig: **sehr große, extrem eng laufende, fast auf
Versalhöhe gesetzte Schlagzeilen in Arial Black-Anmutung, darunter sehr kleine Fließtexte.**
Boulevard-/Stadtmagazin-Look, bewusst ohne Webfont-Ladezeit.

### 3.3 Raster, Abstände, Formen

* **Kein zentrierter Container.** Jede Sektion setzt ihr eigenes horizontales Padding:
  * Kopf-/Fußbereiche, Anzeige: `clamp(1.25rem, 5vw, 5rem)`
  * Inhaltssektionen: `clamp(1.25rem, 7vw, 7.5rem)`
  * Magazin-Sektion: `clamp(1.25rem, 10vw, 11rem)`
* **Vertikalrhythmus:** `.section` = `6.5rem` oben/unten, Spezialsektionen 6–7rem, in geöffneten
  Rubriken auf `4rem` reduziert (`.rubric-body > … { padding-top: 4rem }`).
* **Gaps sind auffällig klein** (0.45–1rem). Kacheln stehen fast auf Stoß – der typische
  „Magazin-Mosaik“-Eindruck.
* **Keine Rundungen.** `border-radius` kommt nur an drei Stellen vor: Scroll-Top-Button (50 %),
  Club-Logos (3 px) und Sport-/Brotzeit-Logokreise. `--radius: .5rem` bleibt ungenutzt.
* **Farbkanten statt Schatten:** Karten werden über 3–6 px starke Ober-, Unter- oder Seitenkanten
  in Rot markiert (`border-top: 6px solid #AA2937` usw.). Schatten nur beim Hover und beim Cover.

### 3.4 Wiederverwendbare Bausteine

| Klasse | Beschreibung |
|---|---|
| `.kicker` / `.kicker-white` | Vorzeile über jeder Headline (rot / rosé) |
| `.button` + `.button-white` / `.button-dark` / `.button-red` | 50 px hoher Inline-Flex-Button, kein Radius, Hover `translateY(-2px)` |
| `.text-link` / `.outline-link` / `.source-actions a` | Textlink mit 2 px roter Unterkante |
| `.section-heading` | Flex-Zeile: links Kicker + H2, rechts Erläuterungstext (max. 390 px) |
| `.tile-image` + `.tile-copy` | generisches Bildkachel-Paar (Shopping, Märkte, Ausflüge) |
| `.rubric-shell` | Akkordeon-Rahmen mit vier Farbvarianten |

---

## 4. Sektionen im Detail

### 4.1 Topline & Header

```
.topline      display:flex · min-height:36px · Grund #AA2937 · Schrift .72rem
              span (900, tracking .12em, rechte Trennlinie) · p (opacity .85) · a (margin-left:auto)
.site-header  display:flex · height:82px · sticky top:0 · z-index:40 · weiß · 1px Unterkante #E0DCD5
              .logo-link 215×70 px · nav (margin-left:auto, gap clamp(1rem,2.3vw,2rem), 900)
              nav a: 1.9rem Padding oben/unten + 3 px transparente Unterkante → beim Hover rot
```

Die Navigation nutzt fünf Punkte (Termine, Entdecken, Stadtleben, Magazin, Werben). Rechts
daneben liegt `details.section-jump` („Bereiche“) mit einem **zweispaltigen Dropdown** (270 px
breit, rote 4-px-Oberkante) und zwölf Direktsprüngen.

### 4.2 Hero „Was ist los in Rostock?“ (`#termine`)

```
.weekend        display:grid · grid-template-columns: minmax(0,1fr) minmax(340px,30vw) · min-height:610px
.weekend-photo  Hintergrundbild /rostock-city-skyline-daylight.png · position 50% 52% · cover
.photo-shade    zwei überlagerte Verläufe (links dunkel → rechts klar, unten leicht abgedunkelt)
.weekend-copy   max-width:760px · padding clamp(3rem,7vw,7rem) · Kicker → H1 → Lead → weißer Button
```

Rechte Spalte `aside.today-panel` (roter Grund, min-height 610 px):

```
.panel-title    Flex, Unterlinie rgba(255,255,255,.3) · „AKTUELL · 7.–13. SEPTEMBER“ + H2 + Kalender-Icon
.top-ten-list   display:grid · grid-template-rows: repeat(10,1fr)   ← exakt 10 Einträge, gleich hoch
  article       grid-template-columns: 29px 1fr · Unterlinie rgba(255,255,255,.2)
    b           29×29 px weißes Quadrat mit roter Ziffer 01–10
    strong      Datumszeile .57rem, #FFD5DA
    h3          Titel .76rem
    p           Zeit/Ort .61rem, #F1CBD0
```

Das ist die inhaltliche Kernidee der Seite: **eine feste Top-10-Wochenliste, die immer neben dem
Hero steht.** Sie ist bewusst auf zehn Zeilen dimensioniert; mehr Einträge sprengen das Raster.

### 4.3 Wetterleiste (`#wetter`)

```
.weather      grid-template-columns: minmax(310px,.75fr) 1.4fr · Grund #D9ECF7 · padding 1.25rem …
.weather-now  Sonnen-Icon · „LIVE-WETTER · ROSTOCK“ + H2 (Temperatur, 2.3rem) · p mit linker Trennlinie
.forecast     grid-template-columns: repeat(4,1fr) · Karten rgba(255,255,255,.62)
              strong (Tag) · span (Lage) · b (Max) mit small (Min) · em (Regenwahrscheinlichkeit)
.weather-source  absolut positionierte Quellenangabe „Vorhersage: Open-Meteo“
```

### 4.4 Brotzeit-Anzeige

Eigenes Mini-Designsystem innerhalb der Seite, damit die Anzeige sich sichtbar vom Redaktionsteil
absetzt: Georgia statt Arial, Gold `#C8974F` statt Rot, eigener Bildlook.

```
.brotzeit-ad        Rahmen #101820, padding 1.15rem clamp(1.25rem,5vw,5rem)
  a                 min-height:360px, Bild absolut gefüllt, Hover scale(1.025)
  .brotzeit-ad-shade  Verlauf 90deg: .94 → .76 (42 %) → .08 (75 %) – Text links lesbar, Bild rechts frei
  .brotzeit-ad-copy   max-width:690px
    small           „ANZEIGE“, umrandet, .58rem  ← Kennzeichnung vorhanden
    .brotzeit-mark  48-px-Goldkreis „B“ + Wortmarke + Untertitel (2 Zeilen, Grid 48px 1fr)
    h2              Georgia 700, clamp(2rem,4vw,4.4rem)/.94
    strong          goldene CTA-Fläche
```

Mobil (≤ 700 px) kippt der Verlauf von horizontal auf vertikal, der Text rutscht an den unteren
Bildrand (`position:absolute; bottom:0`), die Bildhöhe wächst auf 500 px.

### 4.5 Quicknav & Themenkacheln (`#themen`)

```
.quicknav     Flex, white-space:nowrap, overflow-x:auto, min-height:60px – Sprungleiste
.topic-grid   grid-template-columns: repeat(3,1fr) · gap .9rem
.topic-card   min-height:250px · grid-template-rows: auto 1fr auto · 6 px rote Oberkante
              Bild als .topic-image (z-index:-2) + ::after-Verlauf (z-index:-1, bis rgba(5,11,17,.94))
              Icon → H3 (1.45rem, Textschatten) → Text → Pfeil-Icon unten
              Hover: Karte -4px, Bild scale(1.045)
```

Sechs Kacheln: Heute & Wochenende, Essen & Trinken, Warnemünde, Rostock Sport,
Freizeit & Ausflüge, Kultur & Stadtleben. Die Fotos werden **von Unsplash gehotlinkt**
(`photo-…?auto=format&fit=crop&w=1200&q=82`), liegen also nicht auf dem eigenen Server.

### 4.6 Rubrik-Akkordeon

```
.rubric-shell           weiß, 1 px Unterkante, scroll-margin-top:92px (wegen Sticky-Header)
  > summary             display:flex · min-height:190px · padding 2.7rem clamp(1.25rem,7vw,7.5rem)
                        list-style:none (+ ::-webkit-details-marker ausgeblendet)
    div                 max-width:880px: p.Kicker (.7rem/900/uppercase) → h2 → span (Erläuterung, max 720px)
    b                   Schaltfläche „Inhalte anzeigen“, min-width:156px, rot, mit Chevron
  [open] > summary > b svg   → rotate(180deg)
```

Farbvarianten: `.rubric-light` (Verlauf Weiß → `#F1EEE8`), `.rubric-dark` (`#0D151D → #202E39`),
`.rubric-blue` (`#142A3A → #284B61`), `.rubric-red` (`#7F1E2A → #AA2937`, Button invertiert weiß).

### 4.7 Rest der Seite

```
.pharmacy-section  grid 0.7fr 1.4fr, Grund #E3EFE9 · links Icon + H2, rechts:
  .pharmacy-cards  2 Karten (Notdienst-Apotheken) mit 5 px roter Oberkante, tel:-Link
  .four-weeks      weiße Box, 4 Wochen-Chips, roter Button zur aponet-Notdienstsuche
.magazine-section  grid 330px / max 650px, zentriert, Grund #E5E1DA
  .edition-cover   aspect-ratio .704/1, Schatten 0 25px 55px, Hover -5px + rotate(-.5deg)
  .magazine-facts  drei Kennzahlen mit linker roter 3-px-Kante: 40.000 Exemplare · 11× im Jahr · 18 Jahre
.advertise         grid 1.3fr/0.7fr auf #111923, Kicker weiß-rosé, roter Button „Mediadaten anfragen“ (mailto)
footer             grid auto 1fr auto auf #090F15; Logo per filter:brightness(0) invert() weiß gefärbt;
                   small spannt über alle Spalten (grid-column: 1/-1) mit 1 px Oberlinie
```

---

## 5. Layouts der eingeklappten Rubriken (aus dem CSS rekonstruiert)

Diese Klassen sind vollständig gestylt, im Snapshot-Markup aber nicht sichtbar, weil die
`<details>` geschlossen waren. Für den Nachbau ist das die entscheidende Referenz:

| Rubrik | Container | Raster (Desktop) | Kartenaufbau |
|---|---|---|---|
| Monatskalender | `.month-calendar` | `minmax(280px,.7fr) 1.5fr`, links `.calendar-intro` **sticky** (`top:120px`) | `.month-event-list` 2 Spalten, Karte `58px 1fr` mit `.calendar-day` (Tag groß rot + Monatskürzel) |
| Nachtleben | `.nightlife-section` | `.nightlife-grid` 4 Spalten | dunkle Karte `#151321`, weißer Logo-Kasten (72 px), rosé Kicker; Variante `.nightlife-card-muted` = grau/ausgegraut |
| Sport | `.sport-section` | `.sport-grid` 4 Spalten | `.sport-visual` (165 px Foto) + `.sport-logo` (72 px Kreis, ragt −1rem heraus) + `.sport-copy` |
| Kino | `.cinema-section` | `.cinema-grid` 4 Spalten | helle Karte `#F0EDE7`, petrolfarbene 4-px-Oberkante |
| Theater | `.theater-section` | `.theater-list` 2 Spalten | Zeilenkarte `115px 1fr auto`: Datum/Zeit · Titel/Ort · Icon |
| Cafés | `.sparre-section` | `.sparre-grid` 3 Spalten | Karte mit **linker** roter 3-px-Kante, Icon + Adressblock (15 Sparre-Filialen) |
| Restaurants (bezahlt) | `.restaurant-section` | `.restaurant-grid` 2 Spalten | Karte `minmax(190px,.8fr) minmax(260px,1.2fr)`, Bild links mit Bildnachweis-Label, rechts `<dl>` mit Küche/Adresse/Telefon |
| Restaurantführer (frei) | `.restaurant-directory` | `.directory-grid` 3 Spalten | textliche Karte auf Sand, 4-px-Oberkante, `<dl>` ohne Bild – **bewusst sichtbar anders als die Anzeigen** |
| Ausflüge | `.excursion-section` | `.info-card-grid` 3 Spalten | `.tile-image` (155 px) + `.tile-copy` mit `<dl>` und Link unten |
| Shopping | `.shopping-section` | `.shopping-grid` 4 Spalten | Bildkachel + Adresse/Öffnungszeiten |
| Wochenmärkte | `.market-section` | `.market-grid` 5 Spalten | Bildkachel (120 px) + Zeiten, 4-px-Unterkante |

Zusätzlich im CSS vorhanden, im aktuellen Markup **nicht** verwendet – vermutlich Reste einer
früheren Fassung oder für Detailseiten gedacht: `.weekly-grid` / `.weekly-card-copy`,
`.city-grid` / `.story-icon` / `.story-kicker`, `.event-section` / `.event-cards`,
`.brotzeit-feature` / `.brotzeit-gallery` / `.brotzeit-thumbs`, `.sail-recap`, `.header-button`.
Vor einem Refactoring prüfen, ob diese Blöcke zurückkommen sollen – sonst rund 6 KB CSS streichen.

---

## 6. Responsives Verhalten

Drei Breakpoints, moderne Range-Syntax (`@media (width <= 1050px)`), also **Desktop-first**.

| Bereich | ≥ 1051 px | ≤ 1050 px | ≤ 700 px | ≤ 410 px |
|---|---|---|---|---|
| Hauptnavigation | 5 Links sichtbar | **ausgeblendet** (nur noch „Bereiche“-Dropdown) | Dropdown wird `position:fixed`, volle Breite | – |
| Hero | Foto + Panel nebeneinander | einspaltig, Panel darunter | Höhe 570 px, H1 auf 3.7rem | H1 3.25rem |
| Top-10-Liste | 10 Zeilen à 1fr | 5 Zeilen × 2 Spalten | 1 Spalte, freie Höhe | – |
| Themenkacheln | 3 Spalten | 2 Spalten | 1 Spalte, min-height 190 px | – |
| Sport/Kino/Shopping/Nachtleben/Ausflüge/Cafés | 4 bzw. 3 Spalten | 2 Spalten | 1 Spalte | – |
| Wochenmärkte | 5 Spalten | 3 Spalten | 1 Spalte | – |
| Restaurants | 2 Spalten, Karte zweispaltig | 1 Spalte | Karte einspaltig, Bild 280 px hoch | – |
| Wetter | 2 Spalten, Vorhersage 4er | 1 Spalte | Vorhersage 2×2 | – |
| Rubrik-Kopf | Flex-Zeile mit Button rechts | – | Spalte, Button unten, 170 px | – |
| Brotzeit-Anzeige | horizontaler Verlauf, 360 px | – | vertikaler Verlauf, 500 px, Text unten | – |
| Footer | 3 Spalten | – | 1 Spalte | – |
| Header-Button | sichtbar | – | kompakt | **ausgeblendet** |

**Lücke:** Zwischen 700 px und 1050 px gibt es *keine* Hauptnavigation mehr, obwohl auf einem
Tablet reichlich Platz wäre – der `nav` wird bereits bei 1050 px komplett abgeschaltet.
Das ist der auffälligste Layout-Bruch und der erste Kandidat für eine Korrektur.

---

## 7. Interaktion & Motion

* `html { scroll-behavior: smooth }`, dazu `scroll-margin-top` von 80–92 px auf allen Ankerzielen –
  sauber auf den 82 px hohen Sticky-Header abgestimmt.
* Hover: Buttons `translateY(-2px)`, Karten `-3/-4px` plus weicher Schatten, Bilder
  `scale(1.025–1.045)` über 0.4–0.45 s. Zurückhaltend und einheitlich.
* Akkordeon: natives `<details>` – funktioniert ohne JavaScript, Chevron dreht per CSS.
* Navigation: 3-px-Unterkante wird beim Hover rot – kein Layout-Sprung, weil die Kante
  transparent vorhanden ist. Gut gelöst.
* `.scroll-top`: fixer roter Kreis (48 px) unten rechts, `z-index: 35`.

---

## 8. Assets

| Datei | Größe | Einbindung |
|---|---|---|
| `rostock-city-skyline-daylight.png` | **8,6 MB** | CSS-`background-image` im Hero |
| `brotzeit-banner.jpg` | **4,2 MB** | `<img>` in der Anzeige |
| `rostock-life-logo.png` | 774 KB | Header + Footer (im Footer per Filter weiß gefärbt) |
| `rostock-life-august-2026.jpeg` | 733 KB | Cover der aktuellen Ausgabe |
| Unsplash-Fotos | extern | 6 Themenkacheln, direkt von `images.unsplash.com` gehotlinkt |

Ergibt **rund 14 MB eigene Bilddaten allein für den ersten Bildschirm** – siehe Abschnitt 9.
Verschärfend: Logo, Brotzeit-Banner und Ausgaben-Cover stehen im `<head>` als
`<link rel="preload" as="image">`. Der 4,2-MB-Banner wird also mit höchster Priorität geladen,
noch bevor der Hero-Hintergrund überhaupt angefordert wird. Die Themenkacheln sind dagegen
korrekt mit `loading="lazy"` ausgezeichnet.

Externe Ziele: Instagram `@rostocklife`, `brotzeit-rostock.de`, `aponet.de/notdienstsuche/18055-Rostock`,
Yumpu-Ausgabe, `mailto:info@rostock-life.de`. Copyright: „© 2026 K-Verlag GmbH · Titelfoto: TZRW / Holger Martens“.

---

## 9. Bewertung

### Stark

* **Klare redaktionelle Hierarchie.** Hero + feste Top-10-Wochenliste beantworten die Kernfrage
  „Was ist heute los?“ oberhalb der Falz. Danach Wetter, dann Themen – die Reihenfolge stimmt.
* **Sauberer Werbe-/Redaktions-Trennstrich.** „ANZEIGE“-Label, eigene Anzeigen-Typografie und die
  bewusste Unterscheidung zwischen bezahlten Restaurant-Präsentationen (mit Bild und Link) und
  dem kostenlosen Restaurantführer (nur Text). Das ist presserechtlich sauber und wird auch
  visuell durchgehalten.
* **Kein Webfont, ein Stylesheet, native `<details>`.** Wenig Abhängigkeiten, robust.
* **Konsistentes Formvokabular:** keine Rundungen, Farbkanten statt Schatten, enge Gaps.
* **Elf Rubriken hinter Akkordeons** halten eine sehr lange Seite navigierbar; `scroll-margin-top`
  ist überall korrekt gesetzt.

### Schwach / zu erledigen

1. **Bildgewicht.** 8,6 MB PNG als Hero-Hintergrund ist der mit Abstand größte Fehler.
   Skyline als WebP/AVIF in 2–3 Breiten (`image-set()`), Zielgröße < 300 KB. Banner (4,2 MB)
   und Logo (774 KB PNG → SVG) ebenso. Realistisch sind ~14 MB → unter 600 KB.
2. **Navigation verschwindet ab 1050 px** ersatzlos (nur „Bereiche“ bleibt). Tablet-Nutzer
   verlieren die Hauptnavigation. Entweder Burger-Menü mit allen Punkten oder Umbruch statt Ausblenden.
3. **Unsplash-Hotlinking** für die Themenkacheln: fremder Host (Datenschutz/DSGVO-Hinweis nötig),
   keine Kontrolle über Verfügbarkeit, kein Bildnachweis im Markup. Bilder lokal spiegeln.
4. **Barrierefreiheit:**
   * Alt-Texte sind sauber gesetzt – inhaltstragende Bilder beschreibend, die dekorativen
     Themenkacheln bewusst `alt=""` mit `loading="lazy"`. Das ist richtig gelöst.
   * Rosé `#F1CBD0` auf Rot `#AA2937` in der Top-10-Liste liegt deutlich unter 4,5:1 → aufhellen.
   * Roter Text `#AA2937` auf Weiß erreicht ~5,7:1 – für Fließtext ok, für die vielen
     0.6–0.7rem-Kicker aber grenzwertig klein.
   * `.top-ten-list p` mit 0.61rem/`line-height:1.15` ist zu klein für einen Fließtext.
   * Der Zifferblock `<b>01</b>` ist reine Dekoration, wird aber vorgelesen.
   * **Kein einziger `:focus-visible`-Zustand im gesamten Stylesheet.** In Verbindung mit
     `summary { list-style: none }` und den randlosen Buttons ist die Seite per Tastatur
     kaum bedienbar. Das ist der gravierendste A11y-Punkt.
5. **Semantik:** Die Sprungleisten (`.quicknav`, `.section-jump > div`) sind reine `div`-Linklisten;
   besser `<nav aria-label="…">` mit `<ul>`.
6. **Toter CSS-Code:** rund 6 KB für Blöcke, die im Markup nicht mehr vorkommen (Abschnitt 5).
7. **Datumsabhängige Inhalte** („AKTUELL · 7.–13. SEPTEMBER“, „Stand 6. September“, Wochenchips)
   stehen als Fließtext im Markup – braucht einen Redaktionsprozess oder Datenanbindung,
   sonst veraltet die Seite sichtbar.
8. **Impressum und Datenschutz zeigen beide auf `#top`** – Platzhalter, die vor dem Livegang
   zwingend echte Seiten brauchen.
9. **Meta-Ausstattung unvollständig:** `<title>`, `description`, `lang` und `favicon.svg` sind
   gesetzt – aber es fehlen Open Graph/`og:image` (Links in Messengern und sozialen Netzen
   erscheinen ohne Vorschaubild), `theme-color`, `canonical` und strukturierte Daten. Für ein
   Veranstaltungsportal wäre `schema.org/Event` auf Top-10-Liste und Monatskalender der
   größte Sichtbarkeitshebel.

---

## 10. Lokal weiterarbeiten

Empfohlener Startpunkt für die Arbeit auf dem eigenen Rechner:

```
docs/rostock-life/
├── layout-analyse.md          ← dieses Dokument
├── rostock-life-custom.css    ← Styles als Ausgangsbasis (Werte browser-normalisiert)
└── rostock-life-snapshot.html ← DOM-Referenz, Skripte entfernt, Rubriken leer
```

1. `rostock-life-snapshot.html` und `rostock-life-custom.css` in ein Arbeitsverzeichnis kopieren,
   `<link rel="stylesheet" href="rostock-life-custom.css">` einsetzen – dann steht die Startseite
   ohne Build-Kette und ohne React im Browser (es fehlen nur Bilder und Wetterdaten).
2. Bilder lokal ersetzen und dabei gleich optimieren (Punkt 9.1) – die Pfade im Snapshot sind
   bereits auf relative URLs zurückgesetzt.
3. Rubrik-Inhalte nach der Tabelle in Abschnitt 5 nachbauen; die Styles sind vollständig da.
4. Danach die Punkte aus Abschnitt 9 abarbeiten, in dieser Reihenfolge:
   **Bilder → Tablet-Navigation → Fokus-Stile und Kontraste → Meta/OG → CSS-Aufräumen.**

> Hinweis zum CSS-Extrakt: Die Werte stammen aus der vom Browser serialisierten Fassung
> (`rgb()` statt Hex, Langschreibweise, aufgelöste Shorthands). Funktional identisch, aber
> für die dauerhafte Pflege lohnt ein Durchlauf mit Prettier und die Rückführung der
> Leitfarben auf CSS-Variablen (`--accent` statt 60× `rgb(170, 41, 55)`).
