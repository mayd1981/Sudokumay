# Kinder-Varianten für Sudoku – Design

Datum: 2026-11-06
Status: Entwurf abgestimmt

## Ziel

Das bestehende 9×9-Sudoku (`sudoku.html`) bekommt Varianten für Kinder:

- **Kind 4×4** und **Kind 6×6** für kleinere Kinder, inklusive kindgerechtem Design.
- **Sehr leicht** (9×9, ca. 50 Vorgaben) für größere Kinder, eingeordnet unter „Klassisch“.

Alle Varianten erscheinen in einem einzigen Auswahlfeld mit zwei Gruppen.

## Nicht-Ziele

- Keine Symbol-Figuren (Tiere/Formen) statt Zahlen – Zahlen bleiben.
- Keine getrennte Engine oder zweiter Codepfad.
- Kein Aufteilen des Single-File-Spiels in separate CSS/JS-Dateien.
- Keine Online-Ranglisten, Profile oder Mehrspieler-Funktionen.

## Varianten

Konfiguration als Objekt `VARIANTS`:

| Key | Gruppe | Label | Gitter | Blöcke | Vorgaben | Kinder-Regeln |
|---|---|---|---|---|---|---|
| `kind4` | Kinder | Kind 4×4 | 4 | 2×2 | 9 | ja |
| `kind6` | Kinder | Kind 6×6 | 6 | 2×3 | 20 | ja |
| `sehrleicht` | Klassisch | Sehr leicht | 9 | 3×3 | 50 | kein Game Over |
| `leicht` | Klassisch | Leicht | 9 | 3×3 | 40 | – |
| `mittel` | Klassisch | Mittel | 9 | 3×3 | 34 | – |
| `schwer` | Klassisch | Schwer | 9 | 3×3 | 29 | – |
| `extrem` | Klassisch | Extrem | 9 | 3×3 | 25 | – |

Die Vorgaben-Zahlen sind Startwerte und dürfen nach Spieltests angepasst werden.

Auswahl über ein `<select>` mit zwei `<optgroup>`:

- **Kinder:** Kind 4×4, Kind 6×6
- **Klassisch:** Sehr leicht, Leicht, Mittel, Schwer, Extrem

Ein Variantenwechsel startet ein neues Spiel. Existiert Fortschritt, wird vorher nachgefragt (bestehendes Verhalten).

## Architektur

Der bestehende Kern wird parametrisiert statt dupliziert. Ein Codepfad bedient 4×4, 6×6 und 9×9.

### Kern

`candidates`, `generateSolution`, `countSolutions` und `makePuzzle` erhalten `size`, `boxRows` und `boxCols` als Parameter.

- Zellindex: `r = i / size`, `c = i % size`
- Blockgrenzen: `br = r - (r % boxRows)`, `bc = c - (c % boxCols)`
- Kandidaten und Ziffernbereich: `1 … size`
- Die bestehende Eindeutigkeitsprüfung bleibt erhalten; jedes erzeugte Rätsel ist eindeutig lösbar.
- 9×9 ruft den Kern mit `9 / 3 / 3` auf und verhält sich wie bisher.

### Rendering

- Board, Notizraster und Ziffernblock werden dynamisch aus `size` aufgebaut.
- CSS nutzt `repeat(var(--n), 1fr)` für Board und Ziffernblock.
- Blockgrenzen werden als Klassen in JS gesetzt (`box-r`, `box-b`, `edge-r`, `edge-b`) statt über feste `[data-c="2"]`/`[data-r="2"]`-Regeln.
- Notizraster: `boxCols` Spalten, `ceil(size / boxCols)` Zeilen.
  - 4×4 → 2×2
  - 6×6 → 3×2
  - 9×9 → 3×3
- Ziffernblock wird bei Variantenwechsel neu aufgebaut (`buildPad(size)`); Zähler zeigt `size − gesetzt`.
- Tastatur akzeptiert nur Ziffern `1 … size`; höhere Ziffern werden ignoriert.

### Zustand

- `state.variant` (Key aus `VARIANTS`) ersetzt `state.difficulty`.
- Grid-Größe, Blockmaße und Regeln werden über `VARIANTS[state.variant]` abgeleitet.
- `blankState(variantKey, puzzle, solution)` erzeugt den Zustand.
- Bestehende Felder (`grid`, `notes`, `errors`, `hints`, `mistakes`, `seconds`, `done`, `over`, `history`) bleiben erhalten.

### Persistenz

- Weiterhin ein Speicherplatz unter `sudoku.v1`.
- Neues Feld `variant`.
- Beim Laden: `d.variant || d.difficulty || 'mittel'`; unbekannte Keys fallen auf `mittel` zurück.
- Ein Variantenwechsel überschreibt den laufenden Stand nach Rückfrage.

## Regeln

### Kinder-Modi (4×4, 6×6)

- Kein Game Over; Fehler werden rot markiert und dürfen korrigiert werden.
- Fehlerzähler und Timer-Anzeige sind ausgeblendet.
- Die Zeit läuft intern weiter und erscheint im Gewinn-Screen.
- 4×4: keine Notizen (Button ausgeblendet, Notizmodus aus).
- 6×6: Notizen als „Merkzettel“ verfügbar.
- Hilfen: Undo, Tipp, Radierer.

### Sehr leicht (9×9)

- Kein Game Over.
- Fehlerzähler bleibt sichtbar.
- Timer, Notizen und alle Hilfen wie im klassischen Modus.

### Klassische Modi

Unverändert: 3 Fehler, danach Game Over, sichtbarer Timer und Fehlerzähler.

## Kinder-Design

Automatisch aktiv bei Varianten mit `kid: true` über `data-kid="true"` am `<html>`-Element. `sehrleicht` hat `kid: false` und bleibt klassisch gestaltet.

- Weiche, warme Farbpalette; größere Zellen und Ziffern; rundere Ecken.
- Beschriftung „Radierer“ statt „Löschen“; bei 6×6 „Merkzettel“ statt „Notizen“.
- Sparsame Akzente: ⭐/🎉 und „Super gemacht!“ im Gewinn-Screen.
- Keine Dauer-Animationen oder Konfetti.
- Der Theme-Umschalter (hell/dunkel) bleibt unabhängig und funktioniert auch im Kinder-Design.

## Fehlerfälle

- Unbekannter oder alter Varianten-Key → `mittel`.
- Fehlgeschlagene Generierung → erneuter Versuch über die bestehende Logik.
- Notizmodus wird automatisch deaktiviert, wenn die Variante keine Notizen erlaubt.
- Ein alter Spielstand ohne `variant` wird anhand von `difficulty` migriert.

## Testplan

Manuell im Browser, plus ein temporäres Node-Skript für den reinen Kern:

1. Jede Variante erzeugt ein eindeutig lösbares Rätsel.
2. Jede Variante lässt sich per Tipp vollständig lösen; Gewinn-Screen erscheint korrekt.
3. 4×4 und 6×6: kein Game Over, Fehlerzähler und Timer versteckt.
4. 4×4: kein Notizen-Button. 6×6: Notizen mit 3×2-Raster.
5. 6×6: Blockgrenzen (2×3) und Ziffernblock 1–6 korrekt.
6. Ziffernblock-Zähler stimmt mit der Gittergröße überein.
7. Speichern/Laden pro Variante; Migration eines alten 9×9-Stands.
8. Klassische Modi unverändert (Regression).
9. Theme-Umschalter und Tastatur funktionieren in allen Varianten.
