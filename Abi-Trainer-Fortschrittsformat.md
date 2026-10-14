# Abi-Trainer progress file, format `abi-trainer-progress` (schema 1)

Purpose: a plain JSON file that the app saves ("Fortschritt speichern") and loads again. It holds everything the app knows about one student, including every single answer, so that later versions of the app (more topics, a bigger question database, other apps) can read it and continue.

## Rules for readers and writers
- `format` must be `"abi-trainer-progress"`. `schema` is an integer. Only breaking changes raise it. New fields are added without raising it.
- Readers ignore unknown fields and unknown topic ids, and keep them when they write the file again.
- Loading a file never overwrites: it merges. Per topic, card and day the entry with more answers wins. XP is the maximum. Log entries are combined and de-duplicated on `ts + topic + kind + card + question`.
- All "day" numbers are days since 1970-01-01 in the student's local time (`dayEpoch`).
- Text is UTF-8. Math in questions is flattened to plain text: `x^2`, `a_n`, vectors as `(1, 2, 3)`, line breaks as ` | `.

## Top level
| field | meaning |
|---|---|
| `format`, `schema` | see above |
| `_readme` | short description for humans |
| `app` | `name`, `version` (date of the app build), `topics` (list of topic ids this build knew) |
| `exportedAt` | ISO 8601 timestamp |
| `student` | `{ "name": string }`, chosen by the student |
| `profile` | `xp`, `level`, `badges` (badge id to day earned), `goalDays` (days on which the daily goal of 15 answers was reached) |
| `topics` | topic id to `{ mastery, answered, right, dueDay }` |
| `cards` | `"<topicId>.<cardIndex>"` to `{ box, dueDay, seen }` (box 0 to 4, spaced repetition) |
| `daily` | day to `{ answered, right }` |
| `log` | every answer, oldest first (see below) |
| `extensions` | (since build 2026.10.14b) object, one key per add-on (`{ "<name>": {...} }`). Readers keep keys they do not know. |
| `app.stateVersion` | (since 2026.10.14b) version of the on-device data layout, currently 4 |
| `app.packs` | (since 2026.10.14b) ids and versions of question packs that were loaded |

Any other unknown top-level key is kept and written back unchanged.

`mastery` is 0 to 100 (null = never checked). Status used in the UI: below 45 weak, below 75 practising, 75 and above secure.

## Topic ids (build 2026.10.14)
Mathe: m1 Potenzen/Wurzeln, m2 Logarithmus, m3 Lineare Funktionen/LGS, m4 Quadratische Gleichungen, m5 Quadratische Funktionen, m6 Exponentialfunktionen, m7 Trigonometrie im rechtwinkligen Dreieck, m8 Sinus-/Kosinussatz, m9 Bogenmaß/trig. Funktionen, m10 Ableitung, m11 Extrem-/Wendepunkte, m12 Wahrscheinlichkeit, m13 Kombinatorik, m14 Vektoren, m15 Körper/Volumen.
Physik: p1 Geschwindigkeit, p2 beschleunigte Bewegung, p3 Kräfte, p4 Arbeit/Energie/Leistung, p5 Energieerhaltung, p6 Stromkreis, p7 Kreisbewegung, p8 Wärmelehre, p9 Schwingungen/Wellen, p10 Radioaktivität, p11 Optik, p12 Impuls/Stöße, p13 elektrisches Feld/Kondensator.

## Log entry
```json
{
  "ts": 1792000000000,       // ms since epoch
  "day": 20375,
  "kind": "question",        // or "card"
  "topic": "m10",
  "card": null,              // card index when kind = "card"
  "question": "Gegeben ist f(x) = 3x^3 - 2x. Berechne f'(2).",
  "given": "34",             // what the student typed or picked; null if skipped or a card
  "correct": "34",           // correct answer as text
  "right": true,
  "skipped": false,          // "Weiß ich nicht"
  "ms": 18400                // time needed, capped at 10 minutes
}
```
Questions are generated randomly, so the same topic produces different numbers each time. The `question` text is therefore the only exact record of what was asked. The app keeps the latest 4000 entries on the device. Older entries survive in exported files.

## Ideas for later versions
- A question database can add its own stable `qid` to log entries. Old entries without `qid` can be matched by `topic` plus `question`.
- Per-topic error analysis: group `log` entries with `right = false` by `topic`, look at `given` versus `correct` for typical mistakes (sign errors, unit errors).
- The CSV export ("Als Tabelle") has the same columns as the log and opens in Excel.
