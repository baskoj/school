# Matika s medvědy — Duolingo-style redesign

## Goal

Rebuild the single-file math-practice web app so it looks and feels like
Duolingo, for one kid. Keep the existing 13 math topics and their
verified question generators; wrap them in a gamified path/lesson UI with the
existing bear band as mascots.

## Constraints

- **Single self-contained file** `Matika.html` — vanilla HTML/CSS/JS, no
  framework, no build step, no network. Replaces the old `dc-runtime` bundle.
  (Old file is backed up before replacement.)
- Served from GitHub Pages at the **same URL**; offline; progress in
  `localStorage`.
- Czech UI. Written artifacts (comments, commits) in English.

## Content (unchanged math)

All 13 topic generators are ported verbatim from the current app (they are
tested): `zaokr, zlomky, deleni, nuly, slovni, scitani, objem, hmotnost,
nasobeni, zbytek, poradi, zlomkypoc, geometrie`. Each returns
`{ disp, ans, text?, answer|answerNum/answerDen|answer1/answer2, ... }`.

## Structure: units → skills → lessons → crowns

- **4 units** (oddíly), each a colored banner on the path:
  1. **Počítání do milionu** — zaokr, scitani, nasobeni, deleni, nuly, zbytek, poradi
  2. **Zlomky** — zlomky, zlomkypoc
  3. **Jednotky** — objem, hmotnost
  4. **Geometrie** — geometrie
- Each topic is a **skill** node on the winding path (alternating left/right).
- Each skill has **4 lessons × 5 questions**. Finishing all 4 lessons earns a
  **crown**; a skill can be leveled up to **3 crowns** by redoing it.
- Skills unlock **in order**: the next skill opens once the current one has ≥1 crown.
- Node states: `locked` (grey + lock), `current` (orange, progress ring over the
  4 lessons, "ZAČÍT" popover), `done` (green + crown badge; brighter/gold ring at
  higher crown levels).

## Gamification

- **🔥 Série (streak):** +1 the first time a lesson is completed on a new
  calendar day. A day with no completed lesson resets the streak to 0. Stored as
  `{ count, lastDate }` (ISO date, local).
- **⚡ XP + úrovně:** +10 XP per completed lesson (+5 bonus for a flawless
  lesson). Daily goal = 20 XP (shown as a ring/progress). Level = floor(totalXP /
  100) + 1.
- **❤️ Srdíčka (gentle):** start each calendar day with 5. Wrong answer −1. Every
  3 correct answers in a row within a lesson: +1 (max 5). At 0 hearts the lesson
  is **not** blocked — the kid can finish; hearts simply can't go negative and
  refill next day. (No timers, no hard lockout.)
- **🎉 Oslavy + zvuky + animace:**
  - Confetti + bouncing bear + stat cards (XP / streak / accuracy) on lesson complete.
  - Per-answer feedback sheet: green "Výborně!" / red with the correct answer.
  - Sounds via **WebAudio** (generated tones — correct chime, wrong buzz,
    lesson-complete arpeggio, key tap). No audio files → stays offline. A mute
    toggle persists in settings.
  - CSS: button press depth (3D shadow), node pop, progress fill, screen fades.

## Mascots

The existing **bear band SVG** (guitar / accordion / singer) is ported 1:1 from
the old file, preserving every detail. Usage:
- Home path: the three-bear stage scene in the header banner (as today).
- Lesson: a single bear head + speech bubble giving the topic hint.
- Celebrate: the singer bear, bouncing, among confetti.

## Question rendering (in a lesson)

Reuse the existing answer models plus one new one for variety:
- `single` — numeric keypad + answer box (deleni, nuly, scitani, nasobeni,
  poradi, zlomkypoc "část z celku", objem, hmotnost, geometrie).
- `frac` — stacked čitatel/jmenovatel keypad entry (zlomky pie, zlomkypoc addition).
- `pair` — two labelled boxes podíl/zbytek (zbytek).
- `choice` **(new)** — 4 tap tiles, one correct + 3 near-miss distractors.
  Used to add Duolingo flavor on a subset (zaokrouhlování, jednotky). Distractors
  are generated near the correct value and de-duplicated.

Question selection per lesson: 5 questions from the topic generator, **no
duplicates within a lesson**, and different from the topic's previous run where
generator variety allows (carried over from current `buildQuestions` logic).

## Persistence (localStorage, new namespace `matika_v2`)

```
{
  xp: number, streak: { count, lastDate }, hearts: { count, date },
  daily: { date, xp },
  skills: { [topicId]: { crown: 0..3, lesson: 0..3 } },  // lesson = lessons done toward next crown
  settings: { sound: true },
  session: { ...in-progress lesson... } | null
}
```
Fresh start (no migration from old `medved_*` keys). Unfinished lesson resumable.

## Screens / state machine

`path → (tap current skill) → lesson-intro popover → lesson (Q1..Q5 with
feedback) → lesson-complete celebrate → path`. A quit (✕) from a lesson returns
to the path and saves the session for resume.

## Out of scope

Leagues/leaderboards (needs other users), friends, gems/shop, cross-device sync,
geometry drawing, non-numeric topics (time/length units, tables/graphs).

## Testing / verification

Headless-Chrome drive (existing puppeteer harness) over an HTTP server:
- App boots with no console errors; path shows 4 units / 13 skills.
- Complete a lesson in several topics solving correctly → XP/streak/crown update;
  solving wrong → heart decrements and correct answer shown.
- `choice`, `pair`, `frac`, `single` inputs all grade correctly.
- Reload mid-lesson resumes; streak/hearts roll over by date.
- Bears render (guitar visible).
```
