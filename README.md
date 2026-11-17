# JI Industrial Platform

Operating console for the three Jababeka Infrastructure (JI) strategic programmes: vacant asset matching,
one-stop service revenue, and water / waste utility performance. Bilingual English (default) and Indonesian,
purpose-built for internal operating use rather than marketing.

## What this is

A single-page static system. No server, no account, no database. Records live in the browser's
`localStorage` under the key `ji_platform_v2`; the language choice lives under `ji_lang`.
Data can be exported to a JSON snapshot and re-imported on another device.

| View | Purpose | Records |
| --- | --- | --- |
| Operations overview | Three workstream cards, six portfolio KPIs, four charts, a three-phase roadmap, and a data-basis / open-items panel | derived |
| Asset register | Vacant / idle factories, owner terms and matched demand | assets, owners, demands |
| Service revenue | Paid services sold through the one-stop office, and survey leads | deals, leads |
| Water & waste | Two tabs: design capacity register by plant and phase, and monthly readings (loading, energy, compliance, reuse, sludge, unit cost) | plants, metrics |
| Glossary | Regulatory and technical terms with definitions in both languages | 20 entries |
| Settings | Language, import / export, storage status, record ownership, data rules | — |

## Board-review overview

The Overview view is arranged for a board-level read, top to bottom:

- A register banner states that the 203 vacant plots come from the 17 Nov 2026 workbook; commercial, compliance and quality fields are blank pending owner verification, and demand, service leads and utility readings are not yet entered.
- Three workstream cards — vacant asset matching, one-stop service revenue, water & waste efficiency — each show the headline metric, a one-line description and a jump button into the working register.
- Six KPIs: registered assets, vacant / idle share, idle land area, demand in pipeline, service revenue booked, and quoted service pipeline.
- Four unchanged charts: asset status mix, demand by stage, revenue by service line, energy intensity and reuse trend.
- A three-phase roadmap (pilot pool, paid services scale, regional platform with ministry-backed certification) marks the current phase.
- A "data basis and open items" panel records the standing caveats: monthly readings derive from design capacity until metered data is loaded; the Cikarang oxidation-ditch 12,000 m³/day line is listed separately and excluded from the WWTP total pending basis confirmation; the Kendal 16,000 m³/day new build is planned capacity with treatment technology under review.

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

`owners`, `assets`, `demands`, `deals`, `leads`, `plants`, `metrics`. Asset IDs follow
`JI-<estate>-<yy>-<seq>` (`JI-CKR-26-0001`); new asset records inherit the estate code and the next
sequence number. Owner terms are kept separate from the physical asset record so price concessions and
decision authority can change without rewriting the property.

`assets` mirrors the intake workbook (`JI 闲置资产登记模板`, sheet `资产主表`) field for field: asset
ID, estate, block / lot, address, owner ID, owner contact, land area, building area, floors, clear
height, floor load, loading docks, power, water, wastewater connection, gas or steam, permits, land
title, status, owner intent, asking price, condition, retrofit estimate, data quality, last verified,
access tier and notes. All fields are editable in the form, including the ID; renaming an asset
keeps demand links pointing at the renamed record.

The asset detail drawer embeds a Google Maps frame queried from block, address and the Jababeka
Cikarang estate; a link opens the same query in Google Maps. No API key or stored coordinates are
required, and an imprecise address falls back to the estate area.

Commercial fields follow industrial leasing practice: price basis (`Rp/m²/month` for lease,
`Rp/m²` or `Rp total` for sale), service charge (Rp/m²/month), security deposit in months, minimum
lease term, annual escalation, payment schedule, handover condition, fit-out cost (Rp/m²), VAT (PPN)
included, and availability date. These are the parameters buyers and tenants actually compare;
reference ranges observed in Greater Jakarta listings: rent Rp 13,500–81,000 per m² per month,
service charge Rp 1,500–3,000 per m² per month, deposit two to three months, minimum term three years
with 5–10% annual escalation, VAT excluded.

Sorting covers the key metrics: recently verified, rent per m², asking price high to low and low to
high, building area, land area, clear height, connected power, data quality and availability date.
Asking price is normalised to Rp per m² using building area when the record is quoted as a total, so
one ranking holds across both quoting conventions.

`plants` holds design capacity as reported by the operator: estate, plant / unit, phase, treatment
stream (WTP or WWTP), technology, design capacity with its original unit (`Lps` or `m³/day`), status.
The register stores the reported unit and derives the equivalent once, at 1 Lps = 86.4 m³/day, so
mixed units stay comparable without touching the source figure. Units reported separately from a
declared total are flagged and excluded from the operating total until the reporting basis is confirmed.

`metrics` holds one line per plant, indicator and month: flow (actual inflow), energy intensity
(kWh/m³), effluent compliance (%), reuse rate (%), sludge and unit cost (Rp/m³). Loading rate is
derived as actual flow ÷ design capacity; 60–80% is the normal band, above 80% is tight operation and
above 100% is overloading.

Recorded assets: 10 treatment units — Cikarang WTP 1+2 740 Lps; Cikarang WWTP 1+2 15,750 m³/day (IFAS)
and an oxidation-ditch line of 12,000 m³/day reported separately; Kendal Phase 1 WTP 2×70 + 2×70 +
2×100 Lps, WWTP 1A 1,000, 1B 4,000 (IFAS/FCR), 1B extension 16,200 (MBBR), new build 16,000 m³/day
under technology review. Design totals: 1,220 Lps water, 36,950 m³/day wastewater operating, 16,000
m³/day in delivery.

Seed register: 203 vacant plots from the 17 Nov 2026 workbook, matched to 185 unique owner records
(one owner can hold several plots). Land area and owner contact are carried over where the workbook
has them; building size, utility connections, commercial terms, compliance, quality and verification
fields are left blank on purpose and completed during owner follow-up. `demands`, `deals`, `leads`
and `metrics` start empty; `plants` keeps the 10 reported treatment units above.

## Run locally

Open `index.html` in a browser. Publishing through GitHub Pages serves the same file.

## Publish on GitHub Pages

Settings → Pages → Source: *Deploy from a branch* → Branch: `main`, folder `/ (root)` → Save.
The site then appears at `https://pljkt.github.io/ji-industrial-platform/`.

## Operating notes

- Export a JSON snapshot before reloading the register seed, before clearing browser storage, and at the end
  of each reporting month.
- Required fields are validated before a write; a record stays unsaved until the missing field is filled.
- Charts use ECharts 5.5.0 from jsDelivr; there is no analytics and no other external call.
