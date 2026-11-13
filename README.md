# JI Industrial Platform

Operating console for the three Jababeka Infrastructure (JI) strategic programmes: vacant asset matching,
one-stop service revenue, and water / waste utility performance. Bilingual English (default) and Indonesian,
purpose-built for internal operating use rather than marketing.

## What this is

A single-page static system. No server, no account, no database. Records live in the browser's
`localStorage` under the key `ji_platform_v1`; the language choice lives under `ji_lang`.
Data can be exported to a JSON snapshot and re-imported on another device.

| View | Purpose | Records |
| --- | --- | --- |
| Operations overview | Portfolio, pipeline, revenue and utility KPIs with four charts | derived |
| Asset register | Vacant / idle factories, owner terms and matched demand | assets, owners, demands |
| Service revenue | Paid services sold through the one-stop office, and survey leads | deals, leads |
| Water & waste | Monthly plant readings: energy, compliance, reuse, sludge, unit cost | metrics |
| Glossary | Regulatory and technical terms with definitions in both languages | 14 entries |
| Settings | Language, import / export, storage status, record ownership, data rules | — |

## Language rules

- English is the default; Indonesian is the second language. Every label, table header, status,
  chart axis, tooltip and message switches together.
- Strings are stored as `[English, Indonesian]` pairs, so a missing translation cannot silently
  stay in the other language.
- Enumerations are stored as codes (`vacant`, `lease`, `delivered` …) and translated at render time;
  changing language never rewrites the data.
- Company names, plant names and personal names are not translated.
- Numbers, dates and currency follow `en-US` / `id-ID` conventions (`Rp 1.16 bn` / `Rp 1,16 miliar`).
- Professional terms carry a hover definition on first appearance in each view.

## Data model

`owners`, `assets`, `demands`, `deals`, `leads`, `metrics`. Asset IDs follow
`JI-<estate>-<yy>-<seq>` (`JI-CKR-26-0001`); new asset records inherit the estate code and the next
sequence number. Owner terms are kept separate from the physical asset record so price concessions and
decision authority can change without rewriting the property.

Demo dataset sizes: 32 assets, 8 owners, 20 demands, 36 service contracts, 30 survey leads, 100 plant
readings (January–October 2026). The banner marks these records as illustrative; replace them with
live records before operational use.

## Run locally

Open `index.html` in a browser. Publishing through GitHub Pages serves the same file.

## Publish on GitHub Pages

Settings → Pages → Source: *Deploy from a branch* → Branch: `main`, folder `/ (root)` → Save.
The site then appears at `https://pljkt.github.io/ji-industrial-platform/`.

## Operating notes

- Export a JSON snapshot before restoring demo data, before clearing browser storage, and at the end
  of each reporting month.
- Required fields are validated before a write; a record stays unsaved until the missing field is filled.
- Charts use ECharts 5.5.0 from jsDelivr; there is no analytics and no other external call.
