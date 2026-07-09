# Matika s medvědy

A single-file, offline, Duolingo-style math-practice web app for one kid
(Jirka), in Czech, themed around a bear band ("medvědí kapela"). Open it, walk
the learning path, complete short lessons, earn crowns, keep a streak.

## Run it

Open `Matika-Jirka.html` in any modern browser. No server, no build step, no
network — everything (markup, styles, logic, bear artwork, sounds) lives in the
one file. Progress is saved in the browser's `localStorage`.

Hosted on GitHub Pages: <https://baskoj.github.io/school/Matika-Jirka.html>.

## What it covers

Thirteen math topics (whole Czech 4th-grade arithmetic), grouped into four
units on the path:

1. **Počítání do milionu** — zaokrouhlování, sčítání a odčítání, písemné
   násobení, písemné dělení, dělení s nulami, dělení se zbytkem, pořadí operací,
   slovní úlohy
2. **Zlomky** — čtení zlomku z obrázku, sčítání zlomků a část z celku
3. **Jednotky** — objem (l/hl), hmotnost (t/kg/g)
4. **Geometrie** — obvod a obsah čtverce a obdélníku (teorie, bez rýsování)

## How it plays (Duolingo-style)

- **Learning path** — skills unlock in order; each skill is a node with lesson
  states (locked / current / done). The current node shows a progress ring and a
  "ZAČÍT" popover.
- **Lessons + crowns** — each skill has 4 lessons of 5 questions. Finishing all
  4 earns a **crown**; a skill can be leveled up to 3 crowns by redoing it.
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
repeating the previous run.

## How it's built

`Matika-Jirka.html` is a **hand-editable single file** — plain HTML + CSS +
vanilla JavaScript, no framework and no build step. Structure inside the file:

- `<style>` — the design system (Duolingo-like colors, chunky 3D buttons, path,
  lesson and celebrate screens).
- `#app` — a single container; a tiny state machine renders `path → lesson →
  complete` into it via `render()`, with click handling through one delegated
  listener reading `data-act` / `data-arg`.
- The **bear band** is inline SVG, ported 1:1 from the previous version (guitar,
  accordion, singer — all details preserved).
- **Question generators** (`GEN`) produce the 13 topics' questions; they are the
  same verified generators as before, each returning a display kind
  (`pie/expr/prose/word`) and an answer kind (`single/frac/pair/choice`).
- State persists to `localStorage` under `matika_v2` (XP, streak, hearts, daily
  goal, per-skill crown/lesson progress, settings, and a resumable lesson
  session).

To edit, just open the file and change the markup, CSS, generators, or the
`TOPICS` / `UNITS` tables. There is nothing to rebuild.

> The earlier version was a bundled `dc-runtime` artifact; it remains in git
> history if ever needed.
