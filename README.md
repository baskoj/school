# Matika-Jirka

A single-file, offline math-practice web app for one kid (Jirka), in Czech,
themed around a bear band ("medvědí kapela"). Open it, pick a topic, answer
10 questions, collect stars.

## Run it

Open `Matika-Jirka.html` in any modern browser. No server, no build step, no
network — everything (code, fonts, runtime) is embedded in the one file, and
progress is saved in the browser's `localStorage`.

## What it covers

Eight topics, unlocked in order (a topic opens once the previous one has at
least one star):

1. **Zaokrouhlování** — rounding to tens … hundred-thousands
2. **Zlomky** — reading a fraction off a pie chart
3. **Písemné dělení** — long division
4. **Dělení s nulami** — dividing round numbers
5. **Slovní úlohy** — word problems
6. **Sčítání a odčítání** — multi-digit addition / subtraction
7. **Jednotky objemu** — litres ↔ hectolitres
8. **Jednotky hmotnosti** — t / kg / g conversions

Each run is **10 randomly generated questions**. Score earns stars: ≥ 90 % → 3,
≥ 70 % → 2, ≥ 50 % → 1. An unfinished test can be resumed from the home screen.

## How it's built

`Matika-Jirka.html` is a **bundled artifact** produced by `dc-runtime`. It is
not hand-edited HTML — the loader at the top of the file unpacks three embedded
payloads:

- `<script type="__bundler/manifest">` — gzip + base64 of the generated
  `dc-runtime` React framework plus the embedded web fonts (Raveo Display / Inter
  substitutes and JetBrains Mono). **Do not edit by hand.**
- `<script type="__bundler/template">` — a JSON string holding the actual app:
  declarative markup (`sc-if` / `sc-for` / `{{ }}` bindings) and the app logic.
- `<script type="__bundler/ext_resources">` — external resource list (empty).

### The app source

The editable source lives **inside the template payload**, as a
`<script type="text/x-dc" data-dc-script>` block defining
`class Component extends DCLogic`. The runtime calls `renderVals()` on every
render to produce the values bound into the markup, plus React-style lifecycle
hooks (`componentDidMount`, `componentDidUpdate`). The app is a small state
machine over four screens: `home → intro → test → result`.

State persisted to `localStorage`:

- `medved_progress` — `{ topicId: stars }`
- `medved_session` — the in-progress test, so it can be resumed

### Editing the app

Because the source is JSON-encoded inside the template payload, edit it by
decoding, changing the `data-dc-script` block, and re-encoding with the
bundler's exact escaping: `json.dumps(..., ensure_ascii=False)`, then escape
every `</` as a `<` + unicode-escaped slash (see the code below) so an embedded
`</script>` can't close the host `<script>` tag.

```python
import re, json

HTML = "Matika-Jirka.html"
src = open(HTML).read()
pat = re.compile(r'(<script type="__bundler/template">\s*)(.*?)(\s*</script>)', re.S)
m = pat.search(src)
tpl = json.loads(m.group(2))                      # decoded template HTML

# ... edit the data-dc-script block inside `tpl` ...

enc = json.dumps(tpl, ensure_ascii=False).replace("</", "<\\u002F")
open(HTML, "w").write(src[:m.start(2)] + enc + src[m.end(2):])
```

The manifest (runtime + fonts) is untouched by this process. To rebuild the
runtime itself you need the original `dc-runtime` project (`bun run build`),
which is not part of this repo.
