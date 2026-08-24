# Matika s medvědy

Single-file, offline, Duolingo-style math-practice web apps for one kid (Jirka),
in Czech, themed around a bear band ("medvědí kapela"). Open one, walk the
learning path, complete short lessons, earn crowns, keep a streak.

Two apps live here — a course and a final revision built on top of it:

| File | What it is | Live |
| --- | --- | --- |
| `Matika-Jirka.html` | The course. Thirteen topics, one path node each. | <https://baskoj.github.io/school/Matika-Jirka.html> |
| `Matika-Jirka01.html` | The final revision. Six days, topics mixed into every lesson. | <https://baskoj.github.io/school/Matika-Jirka01.html> |

Both run offline in any modern browser: no server, no build step, no network —
markup, styles, logic, bear artwork and sounds all live in the one file.
Progress is saved in the browser's `localStorage`, under a **separate key per
app**, so playing one never disturbs the other.

## Matika-Jirka.html — the course

Thirteen math topics (whole Czech 4th-grade arithmetic), grouped into four
units on the path:

1. **Počítání do milionu** — zaokrouhlování, sčítání a odčítání, písemné
   násobení, písemné dělení, dělení s nulami, dělení se zbytkem, pořadí operací,
   slovní úlohy
2. **Zlomky** — čtení zlomku z obrázku, sčítání zlomků a část z celku
3. **Jednotky** — objem (l/hl), hmotnost (t/kg/g)
4. **Geometrie** — obvod a obsah čtverce a obdélníku (teorie, bez rýsování)

Each topic is a path node holding 4 lessons of 5 questions — 20 tasks per
crown. State lives under `matika_v2`.

## Matika-Jirka01.html — the six-day final revision

The same course, replayed as a **six-day plan** once every topic has been
learned. The path has six day nodes instead of thirteen topic nodes, and each
day mixes several topics into every lesson.

| | Day | Topics |
| --- | --- | --- |
| 1 | Sčítání a odčítání | zaokrouhlování, sčítání a odčítání, slovní úlohy |
| 2 | Násobení a dělení | písemné násobení, písemné dělení, dělení s nulami |
| 3 | Zbytky a závorky | dělení se zbytkem, pořadí operací, slovní úlohy |
| 4 | Zlomky | zlomek z obrázku, sčítání zlomků, část z celku, + dělení |
| 5 | Jednotky a geometrie | objem, hmotnost, obvod a obsah |
| 6 | Velký mix | all thirteen topics |

- **Same workload as a course node** — 4 lessons of 5 questions per day, so 20
  tasks a day and 120 over the six days.
- **Same question generators and the same number ranges** as the course. The
  revision is a re-mix, not an easier or harder variant.
- **Topic bag** — each day keeps a persisted, shuffled bag of topic slots that
  only refills once empty, so every topic of the day comes up before any topic
  repeats. On day 6 that means all thirteen topics appear before a single one
  comes round twice. `spread()` then reorders each lesson's draw so the same
  topic never sits back to back unless the draw forces it.
- **Mixed lessons carry their own context** — every question shows a topic chip
  and a "Den N · lekce K/4" tag, the bear's hint is the hint for that question's
  topic, and tap-choice tiles follow the question's topic rather than the day's.
- **Quitting costs nothing** — tapping ✕ mid-lesson hands the drawn topics back
  to the bag, so an abandoned lesson cannot strand a topic for the day.
- Days unlock in order: day N opens once day N−1 has a crown. State lives under
  `matika_final_v1`.

## How they play (Duolingo-style)

- **Learning path** — nodes unlock in order, with locked / current / done
  states. The current node shows a progress ring and a "ZAČÍT" popover.
- **Lessons + crowns** — 4 lessons of 5 questions per node. Finishing all 4
  earns a **crown**; a node can be leveled up to 3 crowns by redoing it.
- **🔥 Série (streak)** — +1 the first time a lesson is completed on a new day; a
  missed day resets it.
- **⚡ XP + úrovně** — +10 XP per lesson (+5 for a flawless one), daily goal, level.
- **❤️ Srdíčka (gentle)** — 5 per day, a wrong answer costs one, 3 correct in a
  row give one back, full refill each day, and the lesson is never hard-blocked.
- **🎉 Oslavy** — confetti, a dancing bear, and XP / streak / accuracy cards on
  completion, plus WebAudio sound effects (no audio files) and CSS animations.

Question inputs: numeric keypad, stacked-fraction entry, a two-field
podíl/zbytek answer, and tap-choice tiles (rounding & unit topics). Each lesson
avoids duplicate questions and, where the generator has enough variety, avoids
repeating recent ones.

## How they're built

Both files are **hand-editable single files** — plain HTML + CSS + vanilla
JavaScript, no framework and no build step. Structure inside each:

- `<style>` — the design system (Duolingo-like colors, chunky 3D buttons, path,
  lesson and celebrate screens).
- `#app` — a single container; a tiny state machine renders `path → lesson →
  complete` into it via `render()`, with click handling through one delegated
  listener reading `data-act` / `data-arg`.
- The **bear band** is inline SVG (guitar, accordion, singer).
- **Question generators** (`GEN`) produce the 13 topics' questions, each
  returning a display kind (`pie/expr/prose/word`) and an answer kind
  (`single/frac/pair/choice`). `Matika-Jirka01.html` carries them over verbatim.
- State persists to `localStorage` (XP, streak, hearts, daily goal, per-node
  crown/lesson progress, settings, and a resumable lesson session).

To edit, open the file and change the markup, CSS, generators, or the topic
tables — `TOPICS` / `UNITS` in the course, `TOPIC_META` / `DAYS` in the
revision, where a day's `mix` maps a topic id to how many slots it gets in that
day's bag. There is nothing to rebuild.

> The earliest version was a bundled `dc-runtime` artifact; it remains in git
> history if ever needed.
