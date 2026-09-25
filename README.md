# Mimir's Diary — Octad Protocol v4.7

## Concept & Inspiration

The project takes its name from **Mimir**, the Norse figure associated with wisdom and knowledge — here reimagined not as a single sage but as something closer to the *collective consciousness of wisdom itself*: an entity that would, by definition, already contain every book, every writer, and every piece of knowledge ever written or yet to be written.

The retrieval mechanism is inspired by Jorge Luis Borges's short story **"The Library of Babel"** — a library containing every possible combination of letters, and therefore, somewhere within its near-infinite shelves, every true statement, every real discovery, and every coherent work that could ever be written, buried among an overwhelming majority of meaningless noise. Querying Mimir is framed as searching that void: sifting a jumbled, near-infinite space of language for the fragment that turns out to be coherent, credible, and connected to real human knowledge.

In that sense the app is a small **epistemological engine** — a piece of interactive fiction about the act of extracting signal (real historical figures, real documented quotes, real research directions) from noise, rather than a straightforward reference tool. The Gemini-generated content and the "real people, real quotes, interpretive commentary" guardrails (see below) exist to keep that signal-from-noise premise honest: the fiction is that you're pulling a true thread out of the Library; the mechanism ensures there's actually a true thread there to pull.

A single-file, AI-powered interactive "diary" experience built as one self-contained HTML page. Opening it reveals an animated 3D book; clicking into it takes you to a two-page spread where you can select topic "runes" arranged around a circular index. Selecting one generates a fully custom entry — a title, a symbolic "lexical anchor," philosophical interpretation, a real historical quote, a six-member "council" of real historical thinkers offering perspectives, and a closing verdict — narrated aloud via the browser's speech synthesis.

## How it works

- **Single file, no build step.** Everything — HTML, CSS, and JavaScript — lives in `mimirs-diary.html`. Just open it in a browser.
- **Content is generated live** by calling the [Google Gemini API](https://ai.google.dev/) directly from the browser for each topic you select. Nothing is pre-written; every entry is produced on demand from a structured prompt and returned as JSON.
- **Model resolution:** Google has been known to gate entire model families for newer API keys/projects (returning a "no longer available to new users" error) while continuing to serve them to older accounts — a moving target that hardcoded model names can't reliably track. So instead of guessing, the app asks Google's own `ListModels` endpoint what's actually usable *right now* on your key before every generation, tries the model that worked last time first (cached locally for speed), then works through what `ListModels` returned, and only falls back to a couple of hardcoded guesses (`gemini-3.6-flash`, `gemini-flash-latest`, `gemini-2.5-flash`) if the discovery call itself fails outright (e.g. no network).
- **Response validation:** each API response is checked for required fields (title, citation, a full council, etc.) before rendering; malformed responses trigger a retry against the next fallback model rather than crashing.
- **Persistent caching:** generated entries are cached in `localStorage`, so revisiting a rune you've already opened doesn't spend another API call or lose content on refresh.
- **Custom topics:** beyond the fixed runes, you can type any topic into the "…or inscribe your own query" field to generate an entry on the fly.
- **Dynamic council roles:** the six-member council is no longer fixed to the same six roles every time — the model chooses whichever six (from a larger pool: Physician, Historian, Philosopher, Scientist, Economist, Strategic Analyst, Theologian, Artist, Poet, Engineer, Ecologist, Linguist, Sociologist, Anthropologist, Mystic, Astronomer, Mathematician) are most relevant to the topic.
- **Narration** uses the browser's built-in Web Speech API (`speechSynthesis`) — no extra service required.
- **Council sources** are rendered as real, working search links: the app builds a Google search query from the source name the model provides plus the topic, so links always resolve to something real instead of a URL the model invented.
- **Historical accuracy guardrails:** the prompt requires every named figure (in the historical citation and the six-member council) to be a real, deceased historical person, requires the citation to be an authentically documented quote, and frames each council member's commentary as interpretation of their known ideas rather than a fabricated direct quote. Each attribution also carries a self-reported **confidence level** (high/medium/low); medium- and low-confidence attributions are visibly flagged in the UI rather than presented as certain.
- **Visited tracking:** runes you've already opened are marked with a small indicator on the summoning circle so you can track what you've explored (persisted in `localStorage`).
- **Responsive layout:** the book, page spread, and typography adapt at narrower viewports (~768px and below) for phone/tablet use.

## Requirements

- A modern browser (Chrome, Edge, Firefox, Safari).
- A **free Google Gemini API key**. Get one at [aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey) — no credit card required for the free tier.
- No installation, server, or dependencies needed.

## Usage

1. Open `mimirs-diary.html` in your browser (double-click, or drag it into a browser window).
2. Click the animated book to open it.
3. Click into any diary entry / rune. On first use, you'll be prompted to paste your Gemini API key.
4. The key is stored **only in that browser tab's memory for the current session** — it is never written to disk, embedded in the file, or sent anywhere except directly to Google's API.

## Notes on API usage

- Free-tier Gemini limits are roughly 15 requests/minute and up to ~1,500 requests/day (subject to change by Google) — more than enough for personal use of this app, since it makes one API call per entry you open.
- If you see an error mentioning that a model is "no longer available to new users," Google has deprecated that model version — update the model name in the `fetch` call inside `<script>` (search for `generateContent`) to whatever current model Google recommends.
- Because the API key is used client-side, avoid sharing a copy of this file with a real key pasted into it, and avoid publishing it as a hosted page with a key embedded.

## Customizing

- **Topics/runes:** edit the `NODES` array in the script to change the available diary entries.
- **Tone/persona:** edit the prompt template (the `prompt` variable inside `extractMimir()`) to change Mimir's voice, the JSON schema returned, or the council roles.
- **Visual theme:** CSS custom properties are defined at the top of the `<style>` block (`--void`, `--gold`, `--ash`, etc.) for quick palette changes.

## Disclaimer

This is a creative/atmospheric project. AI-generated philosophical content and historical framing should not be treated as authoritative scholarship — verify anything you intend to rely on via the linked research sources, and pay attention to the confidence flags on attributions.

## Repo hygiene

- `LICENSE` (MIT) is included if you want to share this publicly.
- `.gitignore` excludes common local artifacts and guards against accidentally committing a copy of the file with a key pasted into it. There's no server-side `.env` in this project (the key lives only in browser memory per session), but the ignore rules are there as a safety net if that ever changes.
