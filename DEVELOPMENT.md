# JI Industrial Platform — development notes

Technical companion to `README.md`. The README explains what the console does and what the data
means; this file explains how the single file is built, where each piece lives, the contracts that
must not be broken, and how to test and publish a change.

## 1. Shape of the build

One file, `index.html` (~568 KB), no build step, no package manager, no server, no test runner in CI.
Everything is inline: CSS in one `<style>` block, data in a JS seed, application code in one
`<script>` block wrapped in an IIFE. The only external calls are the chart and map libraries
(ECharts 5.5.0, Leaflet 1.9.4, markercluster 1.5.3) pulled from jsDelivr at runtime.

That shape is deliberate: GitHub Pages serves the file as-is, a copy opened from disk still works,
and nothing secret is involved. The cost is that line numbers drift between edits, all views share
one namespace, and any automated check has to go through a browser. When a change grows past a
few hundred lines, keep it in its own banner section and reuse the primitives below.

| File | Role |
| --- | --- |
| `index.html` | The whole system: shell, styles, dictionaries, seed, views, drawer, storage |
| `README.md` | Product description, data model, field meanings, operating notes |
| `DEVELOPMENT.md` | This file: architecture, contracts, test and release procedure |
| `enrich_diff_report.md` | Evidence trail for the screenshot enrichment pass (per-record anchors and exceptions) |
| `build_seed.py`, `export_register.py`, `audit_*.py`, `verify_*.py` | Offline scripts used to build and audit the seed; not needed at runtime |

## 2. Code map

The `<script>` block is split by banner comments of the form `/* ================= NAME ================= */`.
Navigate by banner, never by line number.

| Banner section | Holds |
| --- | --- |
| `APP` | The three dictionaries: `STR` string table, `ENUM` code groups, `GLOSS` term definitions |
| `STATE & HELPERS` | `LS_KEY`, `COMM_SEED_VERSION`, `DV_SEG_KEY`, `LANG`, `STATE`, `VIEW`, `TAB`, `PAGE`, `FILTER`, `dvSegPref`, `setDvSeg`, and `t`, `E`, `tl`, `gl`, `fmt*` (`fmtInt`, `fmtDec`, `fmtArea`, `fmtDate`, `fmtTime`, `fmtStamp`, `fmtDur`), `uid`, `nextAssetId` |
| `SEED DATA` | `seedData()` — the 203-plot industrial register (`REAL_ASSETS`), the 142-plot commercial register (`REAL_COMMERCIAL`), 185 owners, dummy service samples |
| `STORAGE` | `loadData()`, `save()` — read, migrate, seed-on-first-run |
| `SHARED UI` | `openDrawer`, `closeDrawer`, `toast`, `chipFor`, `paginate`, `pagerHtml`, `filterRows`, `termLine` |
| `METRICS` / `DASHBOARD` | `kpis()` (booked vs quoted service value) and `viewDashboard()` — the whole-system snapshot: eight computed KPIs, a nine-module table, four comparison charts and the attention list |
| `TABLE PRIMITIVE` | `tableHtml`, `emptyRow`, `kpiCard` |
| `ASSETS VIEW` / `COMMERCIAL VIEW` / `SERVICES VIEW` / `UTILITIES VIEW` | `viewAssets`, `viewCommercial` (own collection, own sort list `COMM_SORTS`, ownership chips), `viewServices`, `viewUtilities` (retained, hidden from navigation) |
| `GLOSSARY` / `SETTINGS` | `viewGlossary`, `viewSettings` |
| `FIELD SCHEMA` | `E_TYPES`, `openDetail`, `openForm`, `saveForm`, `deleteRecord`, `initAllMap`, `assetDetailExtra` |
| `EXPORT / IMPORT` | `exportJson`, `importJson` |
| `NAV & RENDER` | `ICONS`, `DRAWER`, `renderNav`, `renderView`, `go` |
| `DATA VALIDATION` | the whole field-verification page: enrolment, scoring, photo pipeline, drawer; `valFields(seg)` picks the asset or commercial template, `dvSegTabs` renders the queue switch, `viewValidationCom` is the commercial twin, and the registry entry buttons use `data-dvopen` |
| `EVENTS` | `bindEvents()` — three delegated listeners (click, change, input), the search debounce and the gate form submit |
| `SIGN IN` | `ACCOUNTS`, `accountFor`, `SESSION_KEY`, `session`, `signedIn`, `isAdmin`, `renderGate`, `showGate`, `submitLogin`, `signOut` |
| `ACCESS LOG` (inside `SIGN IN`) | `ACCESS_KEY`, `ACCESS_MAX`, `readAccessLog`, `logAccess`, `accessHtml`, `ACT_LABEL` — browser-local trail of sign-in / sign-out / resume / failed attempts |
| `INIT` | `setLang`, `init()` — i18n parity check, `?v=` / `?tab=` / `?lang=` / `?seg=` handling, first render, gate state |

## 3. Data and storage

`STATE` is one plain object: `owners`, `assets`, `commercial`, `demands`, `deals`, `leads`, `plants`,
`metrics` and `validations`. `assets` and `commercial` are two separate collections on purpose: the
industrial register is never widened by commercial plots, and every join, KPI and queue keys off
`asset_id`, which is `JI-CKR-26-nnnn` for assets and `JI-CML-26-nnnn` for commercial records. Nothing is derived at rest — views compute joins, KPIs and scores at render time.

The store key is `ji_platform_v13` (`LS_KEY`). **Bump the version digit whenever a stored shape
changes meaning** — a new collection, a renamed field, a new enum code that old records cannot
carry — so every browser reseeds instead of reading a half-migrated record. A bump is a one-line
change plus the note in `README.md`; the current value must be stated there.

`loadData()` reads the key, repairs missing fields on older records (price basis, geo status,
latitude/longitude, survey, zone, building type, industry, data-source flags), and seeds
`commercial` from `seedData()` when a store predates the commercial register — that collection is
reference data owned by the estate layer, so it is restored on load rather than treated as a
user-edited record.

That restore is gated by `COMM_SEED_VERSION` (`2026-11-24.exact`), not by `LS_KEY`. When the stored
`commercialVersion` differs, `loadData()` maps every seeded record onto the stored one by `asset_id`
and copies only the source-derived keys (`asset_id`, `code`, `kawasan`, `sector`, `blok`, `nomor`,
`alamat`, `ownerName`, `ownership`, `btype`, `land`, `build`, `lat`, `lon`, `geo`, `geoArea`, `oid`,
`globalid`, `flags`); price, notes, survey and every other hand-entered value survive. Do not replace
the collection wholesale: an early build did, and it would have wiped manual commercial edits.
Records the seed no longer carries are dropped, new ones are added, and `commercialVersion` is then
written, so the merge runs once per browser and version. The branch ends with `applySurvey(parsed)`,
and `loadData()` calls `syncValidations(parsed)` for every path.

The first-run branch seeds, syncs and writes. Unreadable JSON falls through to the seed rather than
throwing. `save()` stringifies the whole `STATE`; a quota error
raises the `dv.full` toast instead of failing silently.

`exportJson()` writes the entire `STATE` as `ji-platform-YYYY-MM-DD.json`, validations included. The
access log is deliberately **not** part of `STATE` and never enters the export (section 12).
`importJson()` guards on the shape (`assets` + `owners`), re-syncs validations, saves and re-renders.
Validation photos are the one exception: the JSON carries only their metadata, because blobs live
in IndexedDB (section 7).

## 4. Language contract

- `STR` maps a key to exactly `['English', 'Indonesian']`, both non-empty. `t(key, vars)` picks by
  `LANG` and substitutes `{name}` placeholders. `init()` warns `i18n gaps` with the offending keys
  when any record breaks the shape — after editing strings, load the page once with the console open.
- `ENUM` holds data codes (`vacant`, `kij1`, `pending` …) and `E(group, code)` resolves them.
  Codes are stored in records; translated labels never are.
- `GLOSS` is `{t:[en,id], d:[en,id]}`; `termLine([...])` renders the hover strip at the top of a view,
  `gl(key, plain)` wraps a term inside running copy. Every new professional term needs an entry and a
  place in the term line of the view where it first appears.
- Counts and plurals get their own keys (`dv.photoOne` / `dv.photoMany`) rather than string
  concatenation, which produces "1 photos".
- Company, plant and personal names are never translated.
- All HTML is built from template strings, so every value that comes from data goes through `esc()`,
  including values placed inside attributes.

## 5. Primitives

`tableHtml(cols, bodyHtml, pager, minWidth)` takes `cols` as `[{w, label, align}]` where `w` is a
percentage string. **The widths must sum to 100 by hand** — nothing normalises them, and a stray
1% shows up as a squeezed header or a wrapped label at the 1140 px minimum. Add a column by taking
space from the widest content column (owner name), then re-check the rendered header row.

`paginate(key, rows, size)` reads/stores `PAGE[key]`; `filterRows(rows, key, fields)` reads
`FILTER[key]`; toolbar controls are declared as `data-sel="<view>" data-f="<field>"` and a search
box as `data-q="<view>"`, so `bindEvents` needs no per-view code. Numeric bounds use the same store
under the reserved keys `min.<field>` / `max.<field>`: `filterRows` skips those keys in the string
pass and then calls `applyBounds(rows, f, get)`, which drops any row outside `>= min` / `<= max`
(either bound alone works, and a blank field fails both). The optional `get(row, field)` lets a view
whose rows wrap another record read the number from the nested object (Data validation passes
`r.a.land`). Markup is `data-num="<view>" data-f="min.land"`, and a clear button carries
`data-rngclear="<view>"`. Filters with custom semantics (completion bands) adjust `rows` inside the
view function before pagination.

`chipFor(group, code)`, `kpiCard(label, value, foot, tone)`, `emptyRow(cols)`, `toast(msg)`,
`openDrawer(title, sub, bodyHtml, footerHtml)` / `closeDrawer()`. Drawer content goes into
`#drawerBody`; any action inside a drawer is a `data-*` attribute handled by the delegated listeners
— there are no inline `onclick` handlers anywhere.

### Dashboard snapshot contract

`viewDashboard()` is a derived view. It reads `STATE.assets`, `commercial`, `owners`, `demands`,
`deals`, `leads`, `plants`, `metrics` and `validations` on every render and never stores totals:
land and building area are summed per register, validation coverage is `surveyed / records`, source
gaps count only non-empty `flags[]` arrays, and the owners row compares the owner collection with the
`owner_id` set on assets. The header count sums the nine stored collections; the module table also
includes Glossary and Settings, which are browser-resident rather than part of `STATE`, so its row
totals need not equal the header count.

KPI totals must reconcile with the table: adding a module means one `snapRow()` row, one
`d.row.*` string pair, and including its collection in `totalRecords` when it lives in `STATE`.
The table columns are 21 / 11 / 56 / 12 = 100; change them by hand and re-check at 1140 px. Chart
containers are `ch-mix`, `ch-own`, `ch-area`, `ch-val`; `chart()` owns their ECharts instances and
`clearCharts()` runs before a view swap. `data-tab-to` on a snapshot button sets the remembered
`TAB[view]` before `go(view)`, which is how the Demand row lands on the Demand tab instead of the
asset list. A successful sign-in calls `renderView()` after `logAccess()`, so the Settings row shows
the entry that was just written.

## 6. Field schema

`E_TYPES.<entity>` is `{coll, idKey, prefix, title, sub, fields[]}`, one field per
`{k, t, g|opts, sec, req, hint}`. `t` is `text`, `number`, `date`, `select` or `textarea`; `sec` is
the section id (`identity`, `physical`, `compliance`, `commercial`, `quality`) and drives the section
headings in both the edit form and the validation evidence block.

Adding an asset field therefore means: one entry in `E_TYPES.asset.fields`, the strings `F.<key>`
and `sec.<section>` (only if the section is new), and a value in the seed or a blank default. Then
confirm the edit form renders it, the asset drawer shows it via `showVal`, and the validation drawer
gains one more input — the completion denominator is computed from the field count, so it moves by
itself.

## 7. Data validation module

`STATE.validations[]` holds one record per queued register record:
`{asset_id, seg, plannedDate, pic, outcome, checkedAt, findings, evidence{}, photos[], updatedAt}`.
`seg` is `ind` or `com` and decides which template, queue and scoring denominator the record uses.
Outcome codes live in `ENUM.voutcome` (`pending`, `scheduled`, `surveyed`, `blocked`, `mismatch`).

**Enrolment.** `syncValidations(state)` is idempotent and gives every register record a record — all
203 assets and all 142 commercial plots — whatever its register flag says — the flag is derived from here, so record existence cannot depend on it. It
canonicalises the array, adds a blank record for every asset without one, backfills evidence keys /
photo arrays / outcome, then `applySurvey`, and returns the number added. It runs from `loadData`,
the seed branch, the reset path, `importJson`, `saveForm`, `viewValidation`, `openValidation` and
`saveValidation`. Records outliving their queue membership is intentional — the visit history stays
visible after the register row turns to `Survey = Yes`. `deleteRecord` drops the asset's validation
row with the asset.

**Zero baseline.** `resetSurveyOnce(state)` runs once, from `loadData`'s stored-state branch only
(gated by `state.surveyFromZero`, so imports and later saves are untouched): every non-`pending`
outcome returns to `pending` and `checkedAt` is cleared, while evidence, photos, findings and dates
stay. It clears the results the first build had carried over from hand-ticked register rows, so the
whole register starts at No. Bumping `LS_KEY` would lose real edits — use a one-time migration like
this instead.

**Evidence set.** `valFields(seg)` returns the template for the queue: `E_TYPES.asset.fields` minus
`VAL_SKIP` (`asset_id`, `survey`, `verified` — those live in the schedule block), 37 fields today,
and `E_TYPES.commercial.fields` minus the same three (37 of the 40 identity-first template) when
`seg === 'com'`. `dvSegTabs` switches the page between the 203 industrial records and the 142
commercial plots; `emptyEvidence(seg)`, `newValidation(assetId, seg)`, `valScore`, `valEvidenceN`
and the drawer all read `rec.seg` so one page serves both queues.

**Queue selection and entry points.** The active segment lives in `FILTER.validation.seg`; `setDvSeg()`
also writes `localStorage.ji_dv_seg` (key `DV_SEG_KEY`), and `viewValidation()` falls back to that
preference when the view opens without a segment, so a reload reopens the queue the user left. Both
registers carry a `[data-dvopen]` button with their record count, and every asset / commercial drawer
carries `[data-vrec]`; both handlers call `setDvSeg()` before `go('validation')` or `openValidation()`,
so the queue and the drawer always agree on the register. `?v=validation&seg=com` (or `seg=ind`) is
the shareable deep link; `?v=commercial` is in the whitelist for the same reason.

**Labels.** `evidenceHtml` prints `t(f.label || ('F.' + f.k))`. Commercial fields reuse the `c.*`
labels (`c.code`, `c.sector`, `c.ownerName`), so a raw `F.code` on screen means a template entry lost
its `label` — fix the schema, do not add a duplicate `F.*` string. Keys, order and section headings are
inherited from the register, so the drawer mirrors the register row field by field. `regHint(k, asset)`
prints the current register value under each input, formatted the same way the register formats it,
and is omitted entirely when the register has no value.

**Completion.** `valScore(rec)` returns `{ev, evN, evT, coreN, phN, total}` with
`total = round(100 · (0.5·evidence coverage + 0.3·core/3 + 0.2·photo))`, where core is planned date,
person in charge and a non-`pending` outcome, one third each. Changing the weights means changing
`dv.formula` (shown in the drawer), the README paragraph and this section in the same commit; the
three component bars must stay visible, because the total alone is a progress indicator, not a grade.
`refreshValScore(assetId)` rewrites only `#dvBig`, `#dvEvBar`, `#dvEvN`, `#dvCoreBar`, `#dvCoreN`,
`#dvPhBar`, `#dvPhN` — it must never re-render the form, or the caret and the unsaved values are lost.

**Unsaved-edit guard.** `captureValInputs(rec)` writes the drawer's inputs back into the record
before any action that re-renders the drawer (`addPhotos`, `removePhoto`, `saveValidation`), and
`valDrawer` saves and restores `#drawerBody.scrollTop`. This is not optional: the first version
re-rendered after an upload and silently dropped the typed person-in-charge, moving the score from
55% back to 35%. Any new drawer action follows the same sequence — capture, mutate, re-render.

**Photos.** JPG only, checked by MIME type or extension, unlimited count.
`shrinkToJpeg(file, 720, 0.62)` draws to a canvas on a white matte and re-encodes with
`toBlob('image/jpeg')`; the blob goes to IndexedDB (`ji_validation_photos`, store `p`, keyed by photo
id) and the record keeps `{id, w, h, bytes}`. `PH.get(id)` returns a cached object URL used by the
thumbnail grid; `PH.forget` / `PH.del` revoke it. If IndexedDB is unavailable the module sets
`PH.broken`, counts the upload as rejected and continues — the page never fails hard. A typical
phone photo lands at 720 px on the long side and 5–120 KB.

**One-way derivation.** The register's Survey column is a derived mirror: `surveyOf(assetId, state)`
returns `'yes'` only when a record exists with `outcome === 'surveyed'`, else `'no'`, and
`applySurvey(state)` rewrites `asset.survey` for every asset. `saveValidation` still fills `checkedAt`
(today by default) and `verified` when the outcome is `surveyed`, then calls `applySurvey`, and
`syncValidations` ends with the same call, so load, seed, import, reset and every save agree. Do
not reintroduce a direct `asset.survey = 'yes'` write anywhere else — changing the outcome and letting
`applySurvey` run is the only supported path, and it also works backwards (changing `surveyed` back to
`pending` returns the row to `No`). `E_TYPES.asset.fields` marks `survey` with `derived: 1`;
`openForm` then renders a read-only `.derived` block (chip plus `f.derivedSurvey`) instead of an input,
`saveForm` never sees that key and keeps the stored value, and `openDetail` appends `dv.derivedTag`
after the value so the source of the flag is visible.

**List.** `viewValidation` joins assets and owners, applies zone, building type, outcome,
completion-band, land-area bounds and free-text filters, sorts through `VAL_SORTS` (completion low to
high by default, so untouched rows surface first), shows 12 rows, and prints four KPIs: in queue,
scheduled, result recorded and average completion. The rows wrap the asset (`r.a`), so bounds go
through `applyBounds(rows, FILTER.validation, (r, fl) => fl === 'land' ? r.a.land : r[fl])`; the land
figure is also a column, so a filtered queue is verifiable on screen. The global search box and the
New button are hidden on this view, since the queue is derived and rows are opened, not created.

**Commercial twin.** `viewValidationCom` mirrors the industrial view against `STATE.commercial`: same
scoring (`valScore`), same 12-row paging, same four KPIs, but ownership (Owner / GBC / Belum BAST),
result, completion-band and land bounds instead of zone and building type, and 11 columns keyed to the
commercial register (plot code, sector, name on record, ownership, land, plan, PIC, result, done,
proof). Rows carry `r.a` as the commercial record, so `regHint` and the drawer compare against the
commercial fields; `dv.noteCom` states that visits are stored on the `JI-CML` record and travel with
the JSON export. The industrial queue keeps `dv.note`.

## 8. Adding a view

1. `ICONS.<view>` — one path inside a 24×24 `viewBox`.
2. `renderNav()` — `item('<view>', 'nav.<view>')` in the intended position.
3. `renderView()` — an `else if (VIEW === '<view>') view<Name>();` branch, plus any hide rules for the
   search box and New button.
4. `init()` — add `'<view>'` to the `?v=` whitelist, otherwise a deep link silently falls back.
5. Strings — `nav.<view>`, `v.<view>.title`, `v.<view>.sub`, and every label the view prints.
6. Toolbar/filters — `data-sel="<view>" data-f="<field>"`, then read `FILTER.<view>` in the view.
7. `README.md` view table row, `DEVELOPMENT.md` code-map row if the section is new.

## 9. Test harness

Browser APIs need an `http` origin; `file://` blocks or degrades `localStorage` and IndexedDB in
some modes. Serve the folder with `python -m http.server 8123` and open
`http://127.0.0.1:8123/index.html`.

| Script | Use |
| --- | --- |
| `extract_js.py` | Extracts the script body to `check.js` |
| `node --check check.js` | Syntax gate after every edit — silent output means pass |
| `make_test.py` | Builds `test_dv.html` from `index.html` with an appended harness (`?t=A` drawer, photo, compression; `?t=B` save, write-back, language switch) |
| `cdp_test.js` | Drives Chrome headless over CDP (Node 22 with global `WebSocket`), polls `document.title` for harness completion, returns the console log and a screenshot |
| `cdp_live.js` | Live smoke on the published page: title, row count, KPIs, drawer opens, evidence inputs, photo input, filter, bilingual switch |
| `cdp_dash.js` | Dashboard regression: KPI labels and values, nine table rows, four chart canvases, absence of the old roadmap / demo strip, console errors, screenshot |
| `cdp_shot.js` / `cdp_drawer.js` | Clean screenshots of a URL / of the drawer top and bottom for review |
| `verify_val.py` | Static check: every `STR` entry has two non-empty strings, validation functions exist |

Two pitfalls worth remembering:

- **Do not use `--virtual-time-budget` for interaction tests.** Under virtual time `Image.onload`
  and blob decoding never fire, so a photo upload fails for reasons that have nothing to do with the
  code. Drive Chrome with a real remote-debugging port and poll for a completion marker instead.
- **PowerShell cannot run multi-line Python inline**, and quoting mangles `python -c`. Write the
  script to a `.py` file, then run it.

## 10. Release checklist

1. Edit `index.html`; touch `README.md` / `DEVELOPMENT.md` when behaviour, schema or rules change.
2. `python extract_js.py`, then `node --check check.js`.
3. Bump `LS_KEY` if the stored shape changed, and update the key in `README.md`.
4. Local smoke on `8123`: the edited view plus the drawer it opens.
5. On Overview, check the eight KPIs and nine module rows against `localStorage.ji_platform_v13`
   counts, confirm all four charts draw, and switch to ID — no literal `d.*` / `nav.*` keys should
   appear and the Demand row must open on its own tab.
6. `test_dv.html` harness `?t=A` and `?t=B` when the validation page is affected.
7. Publish from a temporary clone (the repo has no `.deploy` directory): copy `index.html` in,
   commit as `PLJKT`, push `main`.
8. Verify content, not just the push: fetch
   `https://raw.githubusercontent.com/PLJKT/ji-industrial-platform/<commit-sha>/index.html`,
   strip CR, and compare SHA-256 against the local file. They must match. The `/main/` raw URL lags
   a minute or two behind the push; the commit SHA URL is authoritative.
9. Open `https://pljkt.github.io/ji-industrial-platform/?v=<view>` with a cache-busting query and
   confirm the title, the row count, the drawer and the language switch.

## 11. Sign in gate

`ACCOUNTS` holds the credential pairs and their roles:

| User | Password | Role |
| --- | --- | --- |
| `admin` | `admin@123` | Administrator |
| `coordinator` | `coordinator@1234` | Coordinator |

`accountFor(u, p)` is the only place that compares credentials; never test a password inline.
`sessionStorage.ji_session` holds a JSON session record `{user, role, at, sid}` for this tab only
(`at` is the ISO sign-in time, `sid` an id such as `S-MWUNBO6M-ASV6`). `session()` parses it and
tolerates the legacy bare-username string (mapping it to the Administrator role), so an old tab is
not locked out on deploy; `signedIn()` is `!!session()` and `isAdmin()` is
`session().role === 'Administrator'`.

`init()` calls `renderGate()` and `showGate(!signedIn())`, which also toggles `html.locked` so the
page behind does not scroll. The card is `<div class="gate">` placed before `.app`, z-index 100 —
above the drawer (70) and toast (95) — and its form submits through `submitLogin()`: a correct pair
stores the session, writes a `ji_resume_sid` marker, appends an `in` entry to the access log, sets
`#acctUser`, hides the gate, re-renders the current view so the Settings access count includes the
entry just written, and toasts `lg.welcome` with the account name; a wrong pair clears the
password, prints `lg.err`, keeps focus and appends a `fail` entry. The header carries the signed-in
name plus a log-out icon (`signOut()` appends an `out` entry with the session duration, clears both
session keys and re-locks). The card has its own EN / ID switch (`#gateLang`) using the same
`data-lang` delegation as the header, and `setLang` re-renders the gate because otherwise the locked
page would keep the old language.

Both accounts can open both registers and both validation queues; the only role difference today is
that `isAdmin()` gates the *Clear log* action. Do not build a data permission on `role` without first
moving the check to a backend — section 13's caveat applies.

This is deliberately a front gate, not authentication: it is a static file, the credentials are in
plain sight in this source, and anyone can read `localStorage` or open the JSON export. Keep the
wording in the UI honest ("internal console") rather than implying access control. Real protection
requires serving the data from a backend that checks a password or an SSO session first. Role checks
such as `isAdmin()` are therefore UI conventions against accidental misuse, **not** a security
boundary — the data is already readable by anyone who opens the file.

## 12. Access log

`localStorage.ji_access_log` holds a flat array, newest first, capped at `ACCESS_MAX = 300` entries.
It sits beside `ji_platform_v13` and is never merged into `STATE`, so `save()`, the JSON export and
the import path leave it alone; clearing the register seed does not clear it either.

One entry is `{at, tz, ev, user, role, sid, dur?}`:

| Field | Meaning |
| --- | --- |
| `at` | ISO timestamp of the event; `tz` is the fixed label `WIB` |
| `ev` | `in` (credentials accepted), `out` (log-out), `resume` (a tab reopened an existing session), `fail` (wrong user or password) |
| `user`, `role` | Account that acted; a failed attempt stores the typed username and `role: '—'` |
| `sid` | Session id from the session record, or `—` for a failure |
| `dur` | Seconds between sign-in and sign-out, written only on `out` |

`logAccess(ev)` stamps `at` and `tz` when missing, unshifts, and swallows quota errors. `init()`
reads the session after `loadData()` and writes exactly one `resume` entry per tab and session id —
the marker is `sessionStorage.ji_resume_sid`, which `submitLogin()` sets to the new `sid`, so a plain
reload does not duplicate the entry and a fresh sign-in gets its own.

`accessHtml()` renders the last eight entries as a small table (date and time on two lines, account
with role underneath, event, session id, duration); `viewSettings` places it in the Access log card
with a *Clear log* button. **Only the Administrator sees that button**, and the click handler in
`bindEvents` re-checks `isAdmin()` and refuses with `st.adminOnly` even if a button is forged into the
DOM — both the render and the handler must keep the guard. `t('st.act.*')` translates the event codes;
adding an event means a new code there and a row in `ACT_LABEL`.

## 13. Known limits

- Photos live in the browser profile. They survive reloads and JSON export/import on the same
  machine, but a different browser, a cleared profile or a JSON moved to another device arrives with
  metadata and no thumbnails. Bundling photos into the export would need a zip path that does not
  exist yet.
- The layout is desktop-first: below roughly 1100 px the tables scroll horizontally. That is
  site-wide behaviour, not a regression in any single view.
- No automated test suite runs on push. Coverage is the harness plus the live smoke, both manual.
- The access log (section 12) is per browser, not per estate: two devices produce two unrelated logs,
  there is no central trail, and clearing browser data erases it.
- The validation queue's segment preference (`ji_dv_seg`) and the validation records themselves are
  browser-local like the register. Export/import carries the records, but not the UI preference or
  the photos.
