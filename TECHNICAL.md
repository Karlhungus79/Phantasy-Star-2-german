# Phantasy Star II (Mega Drive) – German Fan Translation: Technical Documentation

Version 1.0 · Last updated: 2026-10-23 · Deutsche Fassung: [TECHNIK.md](TECHNIK.md)

This document describes how the German translation of Phantasy Star II (Sega Mega Drive / Genesis) is built and how its quality-of-life (QOL) features work. It is written for people who want to understand, extend or port the patch. How to apply the patch and what it offers is described in the [README](../README.md).

## Contents

1. [Key facts](#1-key-facts)
2. [Origin and scope](#2-origin-and-scope)
3. [Build process](#3-build-process)
4. [Text system](#4-text-system)
5. [Translation](#5-translation)
6. [Quality-of-life features](#6-quality-of-life-features)
7. [Patch and distribution](#7-patch-and-distribution)
8. [Verification and testing](#8-verification-and-testing)
9. [Fallback and switches](#9-fallback-and-switches)
10. [Known limitations](#10-known-limitations)
11. [Credits and notes](#11-credits-and-notes)

---

## 1. Key facts

| Item | Value |
|---|---|
| Required source ROM | Phantasy Star II, US/EU, Rev A, header `GM 00005501-02` |
| Source ROM size / CRC32 | 786,432 bytes (768 KiB) / `904FA047` |
| Source ROM SHA-1 | `0711080e968490a6b8c5fafbb9db3e62ba597231` |
| Result ROM size | 1,048,576 bytes (1 MiB) |
| Result ROM CRC32 (release 1.0) | `8E7E380F` |
| Result ROM SHA-1 | `05af716d3204950c9ef97bd5b20eb8f2f46d4915` |
| Result ROM MD5 | `ee3197898a87461383c7971a01050889` |
| Patch formats | BPS (62,562 bytes, main patch, verifies the source ROM) and IPS (764,434 bytes, fallback) |
| Basis | lory90's disassembly, text engine of Excalibur_Z's English retranslation (version 1.0.1) |
| Font | the game's original fixed-width font (8 px), extended with Ä Ö Ü ä ö ü ß |
| Assembler | Macro Assembler AS (asl 1.42 Bld 89), `ps2p2bin`, `fixheader.exe` |
| Test emulators | Ares (play tests), Genesis Plus GX (automated tests in a container) |

Only the patch is distributed. No ROM files are part of this repository.

---

## 2. Origin and scope

### What was taken over

- **lory90's disassembly:** it rebuilds the original ROM bit-exactly. This was the precondition for the whole project, because the assembler recalculates all pointers. Text and code may grow without moving anything by hand.
- **Excalibur_Z's retranslation engine** ([github.com/ExcalZ/ps2-retranslation](https://github.com/ExcalZ/ps2-retranslation), v1.0.1): paged text buffer without the 255-byte limit per entry (among other things this fixes the buffer overflow at Lutz), width checking (`linecheck.py`), JSON text files, bug fixes from "PS2 Improvement v4.5" and some QOL features. It is used with Excalibur_Z's permission.
- **Content basis of the translation:** the German text was newly written; references were the Japanese original, the US release and the English retranslation (see [5.1](#51-sources-and-precedence)).

### What was deliberately not taken over

- **The retranslation's proportional font.** The German version keeps the original fixed-width font. This is the reason for the hard length limits in [4.2](#42-dimensions-and-placeholders).
- All features that are tied to the proportional font (PS4-style menus, battle box, long names, the retranslation's options and macros). They were **re-implemented** for the original font (see section 6) or dropped.
- Two retranslation features are not implemented: the two-line battle box (internally "Q8") and long names ("Q9").

---

## 3. Build process

The build is reproducible: original ROM + scripts + German text files = result, at the push of a button. It is built in the retranslation's working tree (a source patch on lory90's disassembly), not in the old `ps2.asm` in the main folder.

### 3.1 The twelve build steps

The build script `4_Originalschrift_bauen.bat` runs these in order. All scripts are **re-runnable** (a second run sets no new hooks).

| Step | Script | Purpose |
|---|---|---|
| 1 | `deopt/patch_deopt.py` | options window (EXP/Meseta, random battles, fast walking, fast-forward) |
| 2 | `umlaut/patch_umlaut.py` | insert umlauts and ß into the font |
| 3 | `fenster/patch_windows.py` | widen menu windows, set umlaut dots |
| 4 | `qol/patch_qol.py` | QOL features (speeds, flash, title menu, TP display, equip, shops, macros, damage window) |
| 5 | `abspann/patch_credits.py` | German staff roll, fade-out, parenthesis glyphs |
| 6 | `pruefen/textcheck.py` | rule check for all texts (**build stops on errors**) |
| 7 | `tools/gentext.py` | insert the German texts into the assembler source and check them |
| 8 | `abspann/patch_rolf.py` | Rolf's line in the ending picture (text ID 1829) |
| 9 | `grafik/patch_grafik.py` | graphics ENERGIE, DEZORIS, GEFAHR! |
| 10 | `grafik/patch_titel.py` | title text STARTTASTE DRÜCKEN |
| 11 | `planet/patch_planet.py` | destination planet in the spaceship flight |
| 12 | `tools/sourcebuild.py` | build the ROM |

Order matters: step 4 reads the window art of the title-menu texts that steps 3/7 generate. Step 6 runs before insertion so that faulty texts stop the build.

### 3.2 Build principles

- **Hooks are exactly as long as the original.** Every change to original code replaces instructions with ones of the same length (`move.w #…` → `jsr (…).l`, 6 bytes each) or extends a condition. Nothing in the original area moves. This is verified by comparing old and new ROM ("nothing moved": only hook sites and jump targets differ).
- **Extra code lives in the extension area.** New code and data sit in the engine's extra area (`ext/…`, from address `$CC180`). The ROM grows to 1 MiB; the assembler recalculates all pointers.
- **One switch per feature.** `ps2.options_originalschrift.asm` holds one switch per feature (`qol_*`, `opt_*`, `reward_options`, `walk_option`, `fast_battle_hold`). Hooks are written as `if <switch> … else <original> endif`. With a switch at 0 the original code is produced at that spot.
- **Text check as a build step.** Characters, end codes, placeholders, line width and lines per window are checked on every build; errors are reported with the entry ID.
- **Assembler pitfalls:** Macro Assembler AS reports "too many arguments" for more than about 12–20 values per `dc.b` line (the scripts split lines accordingly). Anonymous labels (`+`/`-`) must not sit between local labels (`.x`); constants (`=`) must be defined before first use.
- **Window buffers in RAM** lie one behind the other. If a window grows by an odd number of bytes, the following ones sit on odd addresses and `move.l` crashes. `patch_windows.py` therefore inserts a padding byte where needed.

### 3.3 Working folder (excerpt)

Not all of this is part of this repository.

```
01_Original_ROM\          unmodified original ROM (never edited)
02_Extracted\             retranslation source state, working tree of the build
03_Tools\                 scripts (deopt, umlaut, fenster, qol, abspann, grafik, planet,
                          pruefen, patch, dialog, dialog_editor) and backups (bak\)
05_German_Script\         dialogue_de.json, script_de.json, glossaries (CSV)
07_Output_ROM\            timestamped builds, patch output
08_Savestates\            test save files
ROM bauen\                .bat files: 4_ = build, 5_ = patch, 6_ = dialogue editor
```

---

## 4. Text system

### 4.1 Text format and control codes

Game text is plain ASCII in `dc.b "…"`; character codes go through `VDPCharacterMaps` (character → tile numbers). Control bytes in game text:

| Byte | Meaning | In the JSON files |
|---|---|---|
| `$BB` | name of the player character (4 characters) | `{NAME}` |
| `$BC` | second character | `{NAME2}` |
| `$BD` | enemy name | `{ENEMY}` |
| `$BE` | technique name (5) | `{TECH}` |
| `$BF` | item name (10) | `{ITEM}` |
| `$C0` | Meseta amount (6 digits) | `{MESETA}` |
| `$C1` | new line | `{BR}` |
| `$C2` | clear window | `{CLR}` |
| `$C3` | wait for button, **scroll up by one line only** | `{PAGE}` |
| `$C4` | end of text, close window | `{END}` |
| `$C5` | another window follows (e.g. Yes/No) | `{C5}` |
| `$C6`, `$C7` | window closes by itself | `{C6}`, `{C7}` |

### 4.2 Dimensions and placeholders

| Area | Size |
|---|---|
| Dialogue window | 24 characters × 2 lines |
| Large window | 24 × 4 |
| Battle message | 20 characters × 1 line (`{PAGE}` = next line) |
| Ending pictures | 18 × 7; final sentence 26 × 7 |
| Staff roll | own font, 40 tiles wide |

| Placeholder | Width in characters |
|---|---|
| `{NAME}` | 4 |
| `{ENEMY}` | 8 |
| `{TECH}` | 5 |
| `{ITEM}` | 10 |
| `{MESETA}` | 6 |

Name lengths in tables: characters 4, techniques 5, items 10, enemies **8** (10 in the original; shortened so that the battle info field HEILUNG/VERFEHLT can be 8 wide), jobs 8, teleport places 5.

### 4.3 Data model

- `05_German_Script/dialogue_de.json` (`"ps2 dialogue v1"`): **607 entries**. Only the field `en` (which holds the German text) is built. Further fields per entry: `jp` (original), `us` (US release), `ret` (retranslation), `window` (`dialogue` / `big` / `battle` / `ending` / `finale`), `ids`, `bank`.
- `05_German_Script/script_de.json`: window and table texts (segment `windows`, menus, jobs, places …).
- Glossaries as CSV (`Glossar_MD_Namen_Orte.csv`, `Glossar_MD_UI.csv`, …): original | English | German | width | decision | location.
- Chains of entries that run into each other (`falls_through`) are checked together.

### 4.4 Check rules (`textcheck.py`)

On every build, stops on errors:

- displayable characters only
- exactly one end code, same as the original
- placeholders of the same kind and count as the original
- line width (24 / 20 / 18 / 26) with placeholders at maximum width
- lines per window and per button press
- no ellipsis with the character "…" (a half-height dot in the original font); always `...` instead

### 4.5 Page breaks: `{PAGE}` and `{CLR}`

`{PAGE}` waits for a button and scrolls up by **one** line only. It does not clear the window. If the page after `{PAGE}` has fewer lines than the window, lines of the previous page remain at the top and show up twice. Rule since 2026-10-19: after `{PAGE}` the new page must fill the window, otherwise `{PAGE}{CLR}` is used (button, then clear window). The original does the same at comparable places. The rule was applied to 139 dialogue entries (158 × `{CLR}`) without changing the wording.

### 4.6 Umlauts and ß in the original font

Each character consists of two 8×8 tiles (top/bottom). In the US font the top tile is empty. Six kana tiles of the font `FontsIconsArt` (tiles `$581`–`$586`) that the US game never shows are redrawn:

| Tile | Content |
|---|---|
| `$581` | two dots – upper half of Ä and Ö |
| `$582` | two dots (different spacing) – upper half of Ü |
| `$583` | ä (dots inside the letter, like the i) |
| `$584` | ö |
| `$585` | ü |
| `$586` | ß |

Text bytes: `$48` Ä, `$49` ä, `$4B` Ö, `$4C` ö, `$4D` Ü, `$4E` ü, `$4F` ß. `$4A` stays the retranslation's arrow. The font kept its length, nothing moves.

- **Lower-case** letters carry their dots in the bottom tile (so they also work where only the bottom tile is drawn).
- **Capitals** carry their dots in the top tile. Windows that draw only bottom tiles (menu tables, `WinArt`) get the dots in the **blank line above**; `patch_windows.py` sets them anew on every build and reports an error if there is no blank line above an umlaut. This affects, among others, AUSRÜST., SPIEL LÖSCHEN, STÄRKE, GLÜCK, KÖRPER, FÜSSE and ZUFALLSKÄMPFE.
- **Jobs with umlauts** (JÄGER, ÄRZTIN, HÜTERIN): a custom routine `DE_JobRow` copies the row above, with the dots, into the LV/EXP window.
- **Staff roll:** own font (A–Z and space only, 40 tiles wide), no umlauts (Ü becomes UE). Two parenthesis glyphs for "(ANTHROPIC)" sit in free tiles `$4FF` / `$51A` (text bytes `$40` / `$5B`) and are written after the font is decompressed.
- **Title text:** the Ü in STARTTASTE DRÜCKEN is a separate dots tile above a full U.

### 4.7 Dialogue editor

For proofreading there is a local browser editor (`editor_server.py`, Python standard library only, `http://127.0.0.1:8765/`): original (JP/US/retranslation) next to the German, live checking with the same rules as the build, window preview in the real game font (placeholders at maximum width), filters (open / errors / not proofread / changed), a "proofread" flag, notes, saving with automatic backup and conflict protection, and a "build ROM" button. The preview flags leftover lines after `{PAGE}` in yellow (see 4.5). A home-network mode (`--lan`) serves private addresses only.

---

## 5. Translation

### 5.1 Sources and precedence

- All 607 dialogue entries, the battle, menu, message and system texts, the intro, the prologue and the credits are German.
- Content-wise, the intro, Rolf's house, the plot, the townspeople and the ending follow the Japanese original and the English retranslation (decision Q-1). Battle, item and shop messages are based on the US release, in individual cases on JP/retranslation where the US text does not fit the scene. Names and terms are unified throughout (no retranslation names and no English leftovers in the German text).
- Meaning-based and short; window width takes precedence over literalness.

### 5.2 Rules

- **Form of address:** "du" throughout (also strangers and officials); the party says "ihr/euch".
- **Ellipsis:** always `...`
- **Tense:** plot narration in the preterite, game actions (chest, switch) in the present.
- **Quotation marks:** speech in narrated scenes in „…“, always when speakers alternate; direct address without.
- **Table names:** UPPER CASE as in the US release; ß becomes "SS".
- **Items, techniques, enemies:** consistently 1:1 the US Mega Drive names (e.g. CERAM KNFE, TTNM ARMOR, AC SLASHR). This follows from the fixed widths and was a deliberate decision.
- **Character names:** as in the earlier German versions of Phantasy Star Generation:1 and :2: Rolf, Nei, Rudo, Amy, Hugh, Anna, Kain, Shir; places Algol, Motavia, Dezoris, Palma; Gouverneur; Odin (= Tyler); Esper; Climatrol (= AMeDAS); Gaira.
- **Dark Force:** always "Dark Force", never "Dark Falz".
- **Nei-Schwert** stays German in dialogue (the inventory calls it NEISWORD).
- **Dezorians** speak a light dialect ("is", "nich", "'n", "nix"); Motavians speak normally.
- **Item names in dialogue** are in CAPITALS only when the exact inventory name is meant (STAR MIST, MOON DEW, MAGIC CAP …).

### 5.3 Menus and windows

Several windows had to be widened so that German words fit. `patch_windows.py` changes the window art, the position (column) and, where the code hard-codes the position, the cursor X coordinate.

| Window | Inner width before → now | Reason |
|---|---|---|
| Main menu | 7 → 10 | AUSRÜST. |
| Item action | 5 → 11 | NUTZEN / GEBEN / WEGWERFEN; from column 27, cursor X `$178` → `$160` |
| Yes/No (3 digits) | 5 → 6 | NEIN |
| Status / row | 7 → 9 | ZUSTAND |
| Buy / sell | 6 → 11 | VERKAUFEN; from column 27, cursor X `$168` → `$160` |
| Central Tower (#017/#018) | 10 → 15 | LAGER / BIBLIOTHEK / FLUGFELD DACH / VERLASSEN; from column 23, cursor X `$150` → `$140` |
| Right / left | 6 → 8 | RECHTS |
| Stats | 11 → 15, one line higher | BEWEGLICHK. plus number; blank line on top for STÄRKE |
| Equipment (2 digits) | 15 → 17 | RECHTS/KÖRPER plus item name |
| Character profiles (#009–#016) | 22 → 20, column 1 | the stats window must not cover them |
| Character list at STATS/EQUIP | column 3 → 1, cursor X `$A0` → `$90` | the red cursor (a sprite) showed through the wider equipment window |
| Battle info field at top | 4 → 8 | HEILUNG / VERFEHLT; enemy name 8 instead of 10 |
| Battle menu | 4 (unchanged) | KMPF / TAKT; BEF. / MAK. / FLU. |

Short words: WER?, WEM? (only 4 characters of room), MEHR (item list from 9 items), REST (LV window), VERFEHLT, BEWEGLICHK., GESCHICK.

### 5.4 Graphics containing text

| Graphic | Before | Now |
|---|---|---|
| Title screen (blinking text, sprite object) | PUSH START BUTTON | STARTTASTE DRÜCKEN |
| Library (climate data) | ENERGY | ENERGIE |
| Radar in Gaira | DEZOLIS, DANGER! | DEZORIS, GEFAHR! (MOTAVIA, PALMA unchanged) |
| Title logo, (c) SEGA 1989, SEGA logo | – | unchanged English |

The graphics are decompressed with the game's tile format (`DecompressArt`: one byte n per tile, then n entries of value + 32-bit mask, remaining bytes raw), redrawn in the same style and repacked. For the radar, the total size of both graphics stays the same, so everything behind them stays at its address. The title text is not a tile picture but a sprite object (`Obj_PushStartButton`); the new tiles and the new mapping table sit in the extra area, and only two operands in the original code change.

### 5.5 Credits

- German staff roll, headings without umlauts: MITWIRKENDE, BUCH UND REGIE, REGIEASSISTENZ, GRAFIKLEITUNG, FIGURENDESIGN, MECHANIKDESIGN, GRAFIK, MUSIK, SOUNDEFFEKTE, PROGRAMMIERASSISTENZ, PROGRAMMIERUNG, PRODUKTION UND PROGRAMMIERUNG, EIN SPIEL VON SEGA, ENDE. Sega names are unchanged.
- German block: DEUTSCHE FASSUNG · LEITUNG UND TEXTE KARLHUNGUS · TESTS ZACKE · TECHNISCHE UNTERSTUETZUNG CLAUDE AI (ANTHROPIC) · ENGLISCHE RETRANSLATION EXCALZ · DISASSEMBLY LORY. Five blank lines separate the roles.
- A fade-out before the Sega picture was added (`PaletteFadeFrom`), because switching the plane from 64 to 32 lines otherwise made "LORY" flash through for a moment.
- Rolf gets his own line in his ending picture (text ID `1829`; the original has no text there). `patch_rolf.py` changes the picture sequence at the end: instead of waiting 360 frames, the text runs 360 frames plus a 60-frame pause, like the other ending pictures.
- The ending pictures of the other seven characters carry their own wording.

### 5.6 Deliberately untranslated

Entries `0810`, `1651` and `1652` are fantasy language and stay as in the retranslation.

---

## 6. Quality-of-life features

### 6.1 Overview

Column "Origin": **R** = logic from Excalibur_Z's retranslation, re-implemented for the original font; **O** = own addition of this project.

| Feature | Switch | Origin |
|---|---|---|
| Battle speed, 5 steps | `qol_battle_speed` | R |
| Text speed, 5 steps | `qol_msg_speed` | R |
| Damage flash on/off | `qol_flash` | R |
| Title menu: CONTINUE first | `qol_title` | R |
| TP cost in the technique list in battle | `qol_tech_tp` | R |
| Equip: stat comparison with arrow | `qol_equip` | R |
| Shops: party window with arrow and difference | `qol_shop` | R (variant) |
| Battle macros with editor in the field menu, 8 slots | `qol_macros` | O (modeled on Phantasy Star IV) |
| Fast-forward in battle (hold A) | `fast_battle_hold` | O |
| Damage window fits the number of digits | `qol_popup_fit` | O |
| Destination planet in the spaceship flight | `opt_planet_name` | O |
| EXP/Meseta 1x/2x/4x | `reward_options` | O |
| Random battles on/off | `reward_options` | O |
| Fast walking on/off | `walk_option` | O (engine: Improvement v4.5) |
| Red box for the selected options row | `opt_cursor_red` | O |
| Speed display counting upwards (5 = fast) | `opt_speed_up` | O |
| Cursor fix at EQUIP/STATS | – (entry `MOVE`) | O |
| FLUGFELD DACH in the Central Tower | – (entry `WIDEN`/`CURSOR_X`) | O |

### 6.2 Options window

Start on the open field opens the window (in scenes, texts and battles Start pauses as in the original). Six rows; up/down selects the row (wrapping; the selected row shows a blinking red box), left/right changes the value, B, C or Start closes. The window is drawn directly into plane A; the content underneath is saved first and restored on closing.

```
ERFAHRUNG/MESETA  1X          1X / 2X / 4X
ZUFALLSKÄMPFE     AN          AN / AUS  (on / off)
SCHNELL LAUFEN    AN          AN = fast (default), AUS = original speed
KAMPFTEMPO        4           5 = fast … 1 = slow, 4 = original
TEXTTEMPO         3           5 = fast … 1 = slow, 3 = original
SCHADENSBLITZ     AN          AN = red flash as in the original
```

All values live in the save block and are stored with the save file.

### 6.3 The features in detail

**EXP/Meseta.** One common factor 1x/2x/4x. At the end of a battle the sum is multiplied by the factor and divided by the number of living characters with a 32/16-bit division (the original division overflows at high values). The original "+1" stays. Total EXP is capped at 9,999,999, Meseta at 99,999,999.

**Random battles off.** `ProcessRandomBattle` returns immediately when off (also for the retranslation's search check).

**Fast walking.** The engine walks faster than the original by default (`fast_walking`). The switch redirects ten hook sites: step width in all four directions, frames per field (8 instead of 16), scrolling, Gaira alarm (half the time in fast mode only) and the followers' distance (8 frames fast, original otherwise). On closing the window, the walking trail (`$DD00`, 64 entries) is converted to the new speed so that followers do not jump.

**Battle fast-forward.** While A is held in battle, the battle loop continues immediately every second time (double speed for animations, battle text and waits; music plays at normal speed). Hooks in `BattleScreenLoop` and in the interrupt check (A removed from the mask; B, C and the D-pad still interrupt). **Guard against graphics glitches:** `loc_5F74` decompresses 16 enemy tiles per battle step into `decom_buffer`, which are only copied to VRAM in the VBlank. A fast pass without VBlank would overwrite them. The fast pass therefore runs only if `$F790 = 0` (no tiles waiting), `window_index = 0` and `$DE40 = 0` (no window being built). Otherwise the battle runs at normal speed.

**Battle speed.** Per step 1–5 (internal; the display counts the other way round, 5 = fast):

| Internal step | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| Battle message (frames) | 45 | 60 | 80 | 100 | 120 |
| Damage number (frames) | 30 | 45 | 60 | 75 | 90 |
| Pause before the next fighter (frames) | 0 | 0 | +10 | +20 | +30 |

Step 1 does not redraw the stat windows twice after a hit. The original corresponds to displayed 4. The fast-forward works on top of this.

**Text speed.** 2 / 1.5 / 1 / 0.75 / 0.5 characters per frame (displayed 5 … 1; the original corresponds to displayed 3, i.e. 1 character per frame). A/B/C shows the page completely at once as before.

**Damage flash.** Off: no red screen on hits.

**Title menu.** With a save file: FORTSETZEN / NEUES SPIEL / SPIEL LÖSCHEN (continue / new game / delete), the cursor on FORTSETZEN. Without a save file: NEUES SPIEL only. The wording comes from `WinArt_GameSelect`; `patch_qol.py` only changes the order.

**TP cost in battle.** The technique list is three cells wider: name (5), space, TP (2 digits), on both pages (REST). The red box of KMPF/BEF. disappears as soon as characters or commands are chosen (as in the retranslation).

**Equip.** The stat window is 15 cells wide: `ANGRIFF 218^230` = current, arrow (higher / same / lower), value with the marked piece. For a worn piece (E) the values without that piece are shown; for non-equippable pieces nothing; at RECHTS/LINKS the values for the chosen hand. The arrows use the kana tiles `$587`–`$589` of the window font (overwritten in VRAM when the windows are built).

**Shops.** Below the portrait a party window appears: `ALEX ^ 17` = attack (weapon shop) or defense (armor shop) rises by 17, `>` same, `v` drops, empty = cannot equip it. One-handed pieces are compared in the hand where they help most. Deviation from the retranslation: no additional comparison window at WER?.

**Battle macros.** Eight slots A–H, stored in the save file.
- *Assigning (field):* field menu → MAKRO (sixth entry). Next to the slots a view shows the commands of the slot under the cursor. C on a slot opens the command window of the first character (ANGRIFF / TECHNIK / GEGENST. / VERTEID. = attack / technique / item / defend). TECHNIK and GEGENST. open the battle's lists (with TP and MEHR). If the technique or item acts on a character, the target window follows. Then the next party member comes; after the last one the slot is saved (sound). B goes back one step.
- *Using (battle):* TAKT → BEF. / MAK. / FLU. → MAK. → slot → C. **A macro lasts for one round only:** MAK. first remembers the party's commands; at the start of the next round they are written back and the stat windows redrawn. The start of a battle clears the memory.
- *Technique:* hooks in the field menu (`QOL_MenuEntry` at `loc_9A20`, `Win_PlayerMenu` with six entries), at battle start (`loc_F550`, `QOL_BattleStart`) and in `Battle_EventIndex_Standby`. `patch_qol.py` generates the field-menu picture from `WinArt_PlayerMenu` plus MAKRO. Own window IDs: `$76` party window (shops), `$77` macro slots, `$78` preview, `$79` BEF./FLU. (no input), `$7A` KMPF/TAKT (no input), `$7B` commands, `$7C` target. In the field, the command and target windows use the shop list's buffer (`WinArt_StoreInventory`) and write it back from ROM on leaving. In battle, windows above row 15 sit on plane B.
- Macro state: `QOL_MAC_MODE` 0 = battle, 2 = field, 3 = field assigned; macro variables `$8DA0`–`$8DAF` (field only); memory `QOL_MacKeep` `$8DB8`–`$8DCF`, marker `$8DE8`.

**Damage window by digit count.** Width = digits + 1 tile: 1 digit 16 px, 2 digits 24 px, 3 digits 32 px (= the previous size), 4 digits 40 px. The first digit is always 5 px from the left edge (1 px border + 4 px padding); the window stays centered above the target, opens and closes as before, never wider than the target. Applies to damage and healing, enemies and party. 40 px are two sprites built from existing frame tiles (left three columns of the 4-digit window `$6EC` plus right two columns of the 3-digit window `$6E8`): no new graphics, no new memory. Hook in `ext/damagepopups.asm` (block `Popup_BuildReady` to `Popup_AgeSprite`). The healing display on the field (`ext/fieldheal.asm`) is unchanged.

**Destination planet in the spaceship flight.** Both flights (Paseo → Dezoris, Skure → Paseo) run through the same flight scene (`loc_C58A`, 120 frames). The destination follows from `level_index` (Skure = Dezoris, otherwise Paseo). At the bottom left, "2. Planet Motavia" or "3. Planet Dezoris" is shown in the game font (Algol system: Palma 1, Motavia 2, Dezoris 3), white, transparent background, plane A, row 25, column 2, for the whole flight; the 17 character tiles sit from `$022`, right behind the planet. Hook: `move.w #$78,(demo_timer).w` → `jsr DE_PlanetName`. The wording is in the table `TEXT` in `patch_planet.py` (at most 38 characters).

### 6.4 Memory map

| Address | Content | Saved |
|---|---|---|
| `$C696` | EXP/Meseta factor (0 = 1x, 1 = 2x, 2 = 4x) | yes |
| `$C697` | random battles off (0/1) | yes |
| `$C698` | fast walking (0 = fast, 1 = normal) | yes |
| `$C699` / `$C69A` / `$C69B` | battle speed `(step-1) xor 1` / text speed `(step-1) xor 2` / flash (1 = off); 0 = original | yes (party block `$C600`–`$C69D`) |
| `$C686` | marker `$4D37` = macros valid | yes |
| `$C6C0`–`$C6FF` | 8 macros × 4 words | yes |
| `$8C00`–`$8C77` | stat window for equip (field) | no |
| `$8C80`–`$8CE3` | party window for shops | no |
| `$8D00`–`$8DE6` | macro preview, line buffer, backup under the tall window, macro variables | no |
| `$8E00`–`$8E2F`, `$8E50`–`$8EDF` | options window (line image, rows 10–12) | no |
| `$8E50`–`$8EB3` | technique list with TP (battle) | no |
| `$8E7E`, `$8EE0`–`$8EF3` | title menu, speed variables, display marker | no |

These RAM areas are only free while the retranslation's `vwf_windows`, `party_menu_ps4`, `field_options` and `battle_macros` are off (the assembler aborts otherwise). The macros sit at `$C6C0`–`$C6FF`, the area of the copyright copy in the save file; the game now compares only 32 instead of 96 bytes there.

### 6.5 Save files

- **Old save files load in the new build.** The new bytes read as the original setting, the macros are empty (the marker is missing).
- **Not the other way round.** A slot saved by the patched build after "new game" or with macros contains macros instead of the copyright text in `$C6C0`–`$C6FF`. The original ROM (and any build without macros) then considers the slot damaged and **overwrites it at power-on** (original behavior, `loc_D700`). If you switch between the original ROM and the patch, use separate save files.
- The SRAM entry in the ROM header is as in the original; the header checksum is set anew after the changes.

---

## 7. Patch and distribution

- **Tool:** `make_patch.py` (Python, no extra packages). It aborts if the source ROM does not have CRC32 `904FA047`, creates BPS and IPS, applies both itself, compares byte by byte with the built ROM and writes `pruefsummen.txt` (CRC32, SHA-1, MD5, size). The patches were additionally applied independently with Flips (Alcaro).
- **BPS is the main patch:** it contains the CRC32 of source and target, verifies the original ROM on application and copies moved blocks (62,562 bytes). **IPS** is only a fallback and large (764,434 bytes) because the ROM grows to 1 MiB and data has moved.
- **Applying:** e.g. with Flips or RomPatcher.js (in the browser) to the unmodified US/EU ROM with CRC32 `904FA047`. The result is 1,048,576 bytes with CRC32 `8E7E380F`.

---

## 8. Verification and testing

**Automated (container test environment).** The build is reproduced with Macro Assembler AS (asl 1.42 Bld 89, like `asw.exe`) and `ps2p2bin` and is **bit-identical** to the builds on the development machine (original `904FA047`, retranslation reference `964194B3`). Emulation uses Genesis Plus GX headless: key sequences, memory and screen comparisons, forced battles, loading save files. Examples from the tests:

| Test | Result |
|---|---|
| All switches 0 | build bit-identical to the stage before the QOL features |
| Each feature switched off individually | builds; every feature can be built on its own |
| Patch scripts run twice | second run sets 0 new hooks, same CRC32 |
| Original area of the ROM before/after QOL | only hook sites and moved jump targets differ, nothing shifted |
| Text speed step 1/3/5 | 2.00 / 1.00 / 0.51 characters per frame |
| Battle speed step 1/2/3/5 | first round 278 / 296 / 326 / 895 frames |
| Damage flash on / off | 8 / 0 red frames |
| Damage window | numbers 7 / 12 / 345 / 1234 / 99 → 16 / 24 / 32 / 40 / 24 px |
| Fast-forward | battle loop 230 instead of 120 passes in 120 frames; no graphics glitches with the guard |
| Macros | assign in the field, technique / item / target, back steps, saving, MAK. in battle, second round without macro |

**Play tests.** During development each stage was confirmed in Ares on the game itself, among others: menus and windows, battle texts, prologue, Rolf's house, credits, library graphic ENERGIE, options window, fast-forward, fast walking, title text, cursor fix, destination planet, FLUGFELD DACH, damage window, macro editor and "macro for one round only". A point counts as confirmed only when the spot has been seen in the game, not merely built by a script. For release 1.0 the statement in the [README](../README.md) applies: the translation and all quality-of-life features were tested in the Ares emulator from start to finish (title screen, all systems, dialogue, credits).

---

## 9. Fallback and switches

Every feature can be switched off individually:

1. In `ps2.options_originalschrift.asm`, set the switch (`qol_battle_speed`, `qol_msg_speed`, `qol_flash`, `qol_title`, `qol_tech_tp`, `qol_equip`, `qol_shop`, `qol_macros`, `qol_popup_fit`, `opt_cursor_red`, `opt_speed_up`, `opt_planet_name`, `fast_battle_hold`, `walk_option`, `reward_options`) to 0 and rebuild. The hooks then produce the original code.
2. All `qol_*` switches plus `opt_cursor_red`, `opt_speed_up` and `opt_planet_name` at 0: the code corresponds to the stage before the retranslation's QOL features (build `78D452DE` plus the cursor fix). The entries without a switch (cursor fix, FLUGFELD DACH) can be removed from the tables `MOVE` and `WIDEN`/`CURSOR_X` in `patch_windows.py`.
3. **For a fallback that affects the macros, use the old save file first** (see 6.5), because a build without macros overwrites slots containing macros at power-on.

Milestone builds of the code stages (each with the text state of the time; later dialogue changes alter the CRC32): `78D452DE` (translation, umlauts, windows, graphics, credits, options window, fast walking, title text) · `DF748C58` (+ QOL Q1–Q7) · `5CCDCC64` (+ cursor, destination planet, FLUGFELD DACH) · `FA6A57C0` (+ damage window) · `3DDABB27` (+ macro editor) · `BEBC4C0E` (+ macro for one round only). Release 1.0: `8E7E380F`.

---

## 10. Known limitations

- **Fixed widths:** names of characters (4), techniques (5), items (10), enemies (8) and jobs (8) are limited by the original font. Items, techniques and enemies therefore carry the US names.
- The title logo, "(c) SEGA 1989" and the SEGA logo stay English. MOTAVIA and PALMA stay on the radar.
- The staff roll has its own font without umlauts and without lower case.
- Two retranslation features are not implemented: two-line battle box, long names.
- Save files with macros are not interchangeable with the original ROM (see 6.5). Ares save states (quick saves) are valid for exactly one ROM version.

---

## 11. Credits and notes

| Role | Name |
|---|---|
| Translation, project lead, additional QOL | Karlhungus (Marcel) |
| Testing | Zacke |
| English retranslation and part of the QOL basis | Excalibur_Z (ExcalZ) |
| Original disassembly | lory90 |

Thanks to Excalibur_Z for his English retranslation and for kindly allowing the use of his work for this version, and to lory90 for the disassembly everything is built on.

Phantasy Star II is a game by Sega. This is a non-commercial fan project distributed as a patch only; no original ROM is included and one is required.
