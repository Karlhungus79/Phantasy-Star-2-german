# Phantasy Star II (Mega Drive / Sega Genesis) — German Fan Translation

**[English](#english)** | **[Deutsch](#deutsch)**

---

## English

### About this project

This patch is a complete German fan translation of Phantasy Star II for the
Sega Mega Drive / Genesis — text, menus, graphics and credits. It is built
on lory90's public disassembly; the translation itself was newly written by
Karlhungus (Marcel) from the Japanese and US originals. The original fixed-
width font is kept; names, places and terms follow his earlier German
releases of Phantasy Star Generation:1 and Generation:2 (Rolf, Nei, Rudo,
Amy, Hugh, Anna, Kain, Shir; Algol, Motavia, Dezoris, Palma).

### Quality-of-life features

What sets this translation apart: it ships with a whole set of modern
quality-of-life features that the 1989 original never had — all of them
toggleable from the in-game options window or directly during play, and all
saved to the save file. Press Start while standing on the field (overworld)
to open the options window:

- Battle speed, adjustable in 5 steps
- Fast-forward: hold the A button to double the speed of everything in
  battle (music plays at normal speed) — as in the Phantasy Star
  Complete Collection (PlayStation 2)
- Message/text speed, adjustable in 5 steps
- Damage flash (red screen on hits) ON/OFF
- Title screen: "CONTINUE" is shown first and pre-selected whenever a save
  file exists
- Technique TP costs shown directly in the battle technique list
- Equip screen: stat comparison with an arrow shown right on the
  currently selected equipment
- Weapon/armor shops: a party window shows every member at a glance
  whether an item improves, keeps, or lowers their stats
- Battle macros (modeled on Phantasy Star IV): set fixed command
  sequences for the whole party from the field menu (MACRO) and execute
  them instantly in battle (TACTICS → MAC.) — 8 save slots
- Battle damage popup windows now resize automatically to fit the number
  of digits (smaller for single digits, larger from 100 and from 1000)
- Destination-planet display during the spaceship flight
- EXP/Meseta rate selectable (1x/2x/4x)
- Random battles ON/OFF
- Fast walking ON/OFF
- several smaller window fixes (e.g. a cursor glitch on the equip/status
  screens, a wider menu for the Central Tower's rooftop landing pad)

Some of these quality-of-life features (battle speed, message speed,
damage flash, the continue-first title screen, the TP display in battle,
and the equip/shop comparisons) are based on Excalibur_Z's English
retranslation and were ported into this German release's fixed-width
original font (the retranslation itself uses a proportional font engine).
The rest — the battle macros reworked in the style of Phantasy Star IV, the
fast-forward option, the destination-planet display, the adaptive damage
window, the EXP/Meseta/random-battle/fast-walking toggles, and the other
window fixes — are Karlhungus's own additions, not part of the
retranslation.

### Credits

| Role | Name |
|---|---|
| Translation, project lead & additional QOL | Karlhungus (Marcel) |
| Testing | Zacke |
| English retranslation & partial QOL base | Excalibur_Z (ExcalZ) |
| Original disassembly | lory90 |

Special thanks to Excalibur_Z (ExcalZ) for his English retranslation of
Phantasy Star II (https://github.com/ExcalZ/ps2-retranslation) and for some of the
quality-of-life features included in it, which form the technical basis for
part of the improvements listed above in this German release (see the QOL
section for which ones). Thank you as well for kindly granting permission
to use his work for this purpose.

Thanks also to lory90 for the disassembly this project is built on, and to
Zacke for his testing and feedback throughout the project.

### Applying the patch

You need the original, unmodified US/EU ROM of Phantasy Star II (header
`GM 00005501-02`, CRC32 `904FA047`). Patch format: BPS (recommended, verifies
the source ROM automatically) and IPS. Apply with, for example, Flips or
RomPatcher.js (browser-based).

### Status

Version 1.0 — translation and all quality-of-life features complete and
fully tested in the Ares emulator, start to finish (title screen, all
systems, dialogue and credits).

---

## Deutsch

### Über dieses Projekt

Diese Patch-Datei übersetzt Phantasy Star II für den Sega Mega Drive /
Genesis vollständig ins Deutsche — Texte, Menüs, Grafiken und Abspann.
Grundlage ist das öffentliche Disassembly von lory90, die Übersetzung selbst
wurde von Karlhungus (Marcel) neu aus dem japanischen und dem amerikanischen
Original gefasst. Die Originalschrift (feste Zeichenbreite) bleibt erhalten;
Namen, Orte und Begriffe wurden an die bereits veröffentlichten deutschen
Fassungen von Phantasy Star Generation:1 und Generation:2 angepasst
(Rolf, Nei, Rudo, Amy, Hugh, Anna, Kain, Shir; Algol, Motavia, Dezoris, Palma).

### Komfort-Funktionen (Quality of Life)

Das Besondere an dieser Übersetzung: Sie bringt eine ganze Reihe moderner
Komfort-Funktionen mit, die es im Original von 1989 nicht gab — alle
schaltbar über das Optionsfenster bzw. direkt im Spiel, alle im Spielstand
gespeichert. Das Optionsfenster öffnet sich mit der Starttaste, während man
auf dem Feld (der Weltkarte) steht:

- Kampftempo in 5 Stufen einstellbar
- Zeitraffer: Taste A halten verdoppelt die Geschwindigkeit von allem im
  Kampf (Musik läuft normal weiter) — wie in der Phantasy Star Complete
  Collection (PlayStation 2)
- Texttempo in 5 Stufen einstellbar
- Schadensblitz (roter Bildschirm bei Treffern) AN/AUS
- Titelmenü: bei vorhandenem Spielstand steht "FORTSETZEN" direkt oben
- TP-Kosten der Techniken werden im Kampf direkt in der Liste angezeigt
- Ausrüsten: Werte-Vergleich mit Pfeil direkt am markierten Ausrüstungsteil
- Waffen-/Rüstungsladen: Gruppenfenster zeigt allen Mitgliedern auf einen
  Blick, ob ein Gegenstand ihre Werte verbessert, gleich lässt oder senkt
- Kampf-Makros (nach dem Vorbild von Phantasy Star IV): feste
  Befehlsfolgen für die ganze Gruppe im Feldmenü (MAKRO) anlegen und im
  Kampf per Tastendruck (TAKT → MAK.) sofort ausführen — 8 Speicherplätze
- Schadensfenster im Kampf passt seine Breite automatisch der Ziffernzahl
  an (klein bei einstelligem Schaden, größer ab 100 und ab 1000)
- Zielplanet-Anzeige während des Raumschiff-Flugs
- Erfahrung/Meseta mit 1x/2x/4x einstellbar
- Zufallskämpfe AN/AUS
- Schnelles Laufen AN/AUS
- mehrere kleine Fensterkorrekturen (u. a. Cursor-Fehler bei Ausrüsten/
  Werte behoben, breiteres Menü am Flugfeld Dach im Zentralturm)

Ein Teil dieser Komfort-Funktionen (Kampftempo, Texttempo, Schadensblitz,
das Titelmenü mit FORTSETZEN zuerst, die TP-Anzeige im Kampf und die
Vergleiche bei Ausrüsten/Laden) stammt inhaltlich aus der englischen
Retranslation von Excalibur_Z und wurde für diese deutsche Fassung in die
Originalschrift mit fester Zeichenbreite übertragen (die Retranslation
nutzt selbst eine Proportionalschrift). Der Rest — die nach dem Vorbild
von Phantasy Star IV umgebauten Kampf-Makros, der Zeitraffer, die
Zielplanet-Anzeige, das sich anpassende Schadensfenster, die Schalter für
Erfahrung/Meseta, Zufallskämpfe und schnelles Laufen sowie die übrigen
Fensterkorrekturen — sind eigene Ergänzungen von Karlhungus und nicht Teil
der Retranslation.

### Danksagung

| Rolle | Name |
|---|---|
| Übersetzung, Projektleitung & weitere QOL | Karlhungus (Marcel) |
| Tests | Zacke |
| Englische Retranslation & Teil der QOL-Basis | Excalibur_Z (ExcalZ) |
| Original-Disassembly | lory90 |

Ein besonderer Dank geht an Excalibur_Z (ExcalZ) für seine englische
Retranslation von Phantasy Star II (https://github.com/ExcalZ/ps2-retranslation) und
einen Teil der darin enthaltenen Komfort-Funktionen (Quality of Life), die
die technische Grundlage für einen Teil der oben genannten Verbesserungen
dieser deutschen Fassung bilden (welche genau, steht im Abschnitt
"Komfort-Funktionen"). Vielen Dank auch für die freundliche Erlaubnis, seine
Arbeit hierfür zu verwenden.

Außerdem Dank an lory90 für das Disassembly, auf dem dieses Projekt aufbaut,
sowie an Zacke für seine Tests und Rückmeldungen während des gesamten
Projekts.

### Patch anwenden

Benötigt wird die originale, unveränderte US/EU-ROM von Phantasy Star II
(Header `GM 00005501-02`, CRC32 `904FA047`). Patch-Format: BPS (empfohlen,
prüft die Quell-ROM automatisch) und IPS. Anwenden z. B. mit Flips oder
RomPatcher.js (im Browser).

### Stand

Version 1.0 — Übersetzung und alle Komfort-Funktionen vollständig und
durchgängig im Emulator Ares getestet (Titelbildschirm, alle Systeme,
Dialoge und Abspann).
