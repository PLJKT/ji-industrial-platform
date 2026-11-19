# JI Industrial Platform

Operating console for two Jababeka Infrastructure (JI) strategic programmes: vacant asset matching
and one-stop service revenue. The water / waste utility register and indicator framework are retained
in the data model but hidden from the navigation for this release, so they can be re-enabled later.
Bilingual English (default) and Indonesian, purpose-built for internal operating use rather than marketing.

## What this is

A single-page static system. No server, no account, no database. Records live in the browser's
`localStorage` under the key `ji_platform_v11`; the language choice lives under `ji_lang`.
Data can be exported to a JSON snapshot and re-imported on another device.

| View | Purpose | Records |
| --- | --- | --- |
| Operations overview | Two workstream cards, six portfolio KPIs, three charts, a three-phase roadmap, and an action / data attention panel | derived |
| Asset register | One asset-centric register: each vacant plot joins its owner name, business industry, contact, location identification, survey status, motive, ask price, pressure and KYC, plus matched demand | assets, owners, demands |
| Service revenue | Paid services sold through the one-stop office, and survey leads | deals, leads |
| ~~Water & waste~~ | Hidden from navigation in this release. The plants / metrics register, efficiency indicator framework and readings view are kept in the code and data for future re-enablement | plants, metrics (retained) |
| Glossary | Regulatory and technical terms with definitions in both languages | 31 entries |
| Settings | Language, import / export, storage status, record ownership, data rules | — |

## Board-review overview

The Overview view is arranged for a board-level read, top to bottom:

- A register banner states that the 203 vacant plots come from the 17 Nov 2026 workbook; commercial, compliance and quality fields are blank pending owner verification, while service contracts and leads carry Dummy format samples only.
- Two workstream cards — vacant asset matching and one-stop service revenue — each show the headline metric, a one-line description and a jump button into the working register.
- Six KPIs: registered assets, vacant / idle share, idle land area, demand in pipeline, service revenue booked, and quoted service pipeline.
- Three charts: asset status mix, demand by stage, and revenue by service line.
- A three-phase roadmap (pilot pool, paid services scale, regional platform with ministry-backed certification) marks the current phase.
- A "needs attention" panel lists A-class assets with no matched demand and records not verified in the last 90 days.

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

`owners`, `assets`, `demands`, `deals`, `leads` drive the visible views. `plants` and `metrics` are
retained below for the hidden utility view described above. Asset IDs follow
`JI-<estate>-<yy>-<seq>` (`JI-CKR-26-0001`); new asset records inherit the estate code and the next
sequence number. Owner terms are kept separate from the physical asset record so price concessions and
decision authority can change without rewriting the property; the register view joins the owner record
onto each asset row, and the full owner record opens from the asset drawer.

`assets` mirrors the intake workbook (`JI 闲置资产登记模板`, sheet `资产主表`) field for field: asset
ID, estate, block / lot, address, owner ID, owner contact, land area, building area, floors, clear
height, floor load, loading docks, power, water, wastewater connection, gas or steam, permits, land
title, status, owner intent, asking price, condition, retrofit estimate, data quality, last verified,
access tier and notes. All fields are editable in the form, including the ID; renaming an asset
keeps demand links pointing at the renamed record.

`owners` carries name, business industry, contact, legal form, beneficial owner, ownership
structure, motive, floor / asking price, time pressure, encumbrances, transaction preference,
commission tolerance, decision authority and KYB. The register joins the owner's industry onto
each asset row under the Industry column. Industry is recorded against public company profiles
where one could be verified; the remainder follow strong company-name tokens. Pure personal-name
records are marked `Private individual`, and companies that could not be verified stay blank (`—`)
rather than being guessed. The 18-key bilingual taxonomy covers food & beverage, agriculture &
plantation, pharma & healthcare, textile & apparel, plastic & packaging, metal & machinery,
machinery & automation, electronics & electrical, chemicals, automotive components, wood &
furniture, building materials, logistics & warehousing, trade & distribution, services &
consulting, finance & investment, private individual and other.

Each asset row also carries two bilingual Yes/No statuses. **Location identification** is derived from the resolved road coordinate: Yes when the record has a road-level pin, No when the road is still pending. **Survey** records whether someone has visited the factory site to verify the register's data (area, condition, contact, terms) against the physical premises; it is a manual flag defaulting to No until that site check is done, and it can be edited in the asset record.

The asset detail drawer embeds a Google Maps frame only when the record's location is
identified: the frame pins that single asset by its stored coordinates, and a link opens the
same query in Google Maps. Records whose road is still pending show no map card at all. No API
key is required.

Below the asset register table, a wide Leaflet map (OpenStreetMap tiles, no API key) plots every
asset whose road was resolved offline once during the build; markers cluster on the map and a popup
opens the asset record. Records without a resolved road are counted as pending and are not pinned,
because parcel coordinates are not in the source register and are not invented. The pending list
needs parcel coordinates from the estate master plan to be located. A header link still opens the
whole estate in Google Maps.

Phone contacts are stored as text. Cikarang / Bekasi fixed lines carry the `021` area code; numbers
already in `08xx` mobile format are kept as mobiles. Multi-number cells, `HO` / `Jababeka` labels
and range suffixes (`-58`) are preserved. One 10-digit entry (`5349268001`) is kept as reported and
still needs owner confirmation.

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

`plants` holds design capacity as reported by the operator (this collection feeds only the hidden utility view): estate, plant / unit, phase, treatment
stream (WTP, WWTP or solid waste), technology, design capacity with its original unit (`Lps` or `m³/day`),
status. The register stores the reported unit and derives the equivalent once, at 1 Lps = 86.4 m³/day, so
mixed units stay comparable without touching the source figure. Units reported separately from a
declared total are flagged and excluded from the operating total until the reporting basis is confirmed.

`metrics` holds one line per plant, indicator and month (this collection feeds only the hidden utility view): treated flow (m³/day), energy intensity
(kWh/m³), chemical dose (kg/m³), effluent compliance (%), pollutant removal (%), sludge output
(kg DS/m³), reuse rate (%), unit cost (Rp/m³), and solid-waste coverage, material recovery, landfill
diversion and cost per tonne. Loading rate is derived as actual flow ÷ design capacity; the reference
band is 80–90% utilisation, below 70% indicates under-use and above 100% indicates overload. The
readings tab also carries an indicator framework: each indicator lists its stream, unit, public
benchmark range and what it measures. Benchmarks are KPI reference ranges from public WWTP / WTP and
solid-waste practice sources, not JI measured results; plant values are entered only from metered
readings.

Retained utility seed data: 10 treatment units — Cikarang WTP 1+2 740 Lps; Cikarang WWTP 1+2 15,750 m³/day (IFAS)
and an oxidation-ditch line of 12,000 m³/day reported separately; Kendal Phase 1 WTP 2×70 + 2×70 +
2×100 Lps, WWTP 1A 1,000, 1B 4,000 (IFAS/FCR), 1B extension 16,200 (MBBR), new build 16,000 m³/day
under technology review. Design totals: 1,220 Lps water, 36,950 m³/day wastewater operating, 16,000
m³/day in delivery. No solid-waste plant capacity record has been reported yet; the solid-waste
indicators exist in the framework and reading form, with capacity rows to be added when the waste plant
register is supplied.

Seed register: 203 vacant plots from the 17 Nov 2026 workbook, matched to 185 unique owner records
(one owner can hold several plots). Land area and owner contact are carried over where the workbook
has them; 194 plots carry a recorded land area (344,717 m² total, whole square metres rounded half up) and 9 plots leave the area blank
because the workbook has none. Building size, utility connections, commercial terms, compliance,
quality and verification fields are left blank on purpose and completed during owner follow-up.
`demands` starts empty; `plants` keeps the 10 reported treatment units above, and `metrics` is seeded
with 21 Dummy format readings for 2026-10 (flow, energy, compliance, removal, reuse, sludge, chemical
dose, unit cost) so the hidden utility table, filters and KPI aggregations can be tested before real
metered data is loaded.

`deals`, `leads` and `metrics` carry a **Data source** flag: `Real` (`Nyata`) for actual
registered records and `Dummy` for a format sample. The 10 seeded contracts, 10 seeded survey leads and
21 seeded utility readings are all `Dummy` — their plant names, values and scores exist only to
demonstrate the table, filter and field format and are not real; signed agreements, real
respondents and metered readings are entered with `Real`. KPI cards and dashboard aggregates count
every visible record in the register regardless of its source flag; the flag marks provenance, not inclusion.

`deals` service-line taxonomy follows estate-operator charges cross-checked across IEAT
estate tariffs, the Jababeka CODB schedule and shared-utility SEZ sources: permits and
compliance, energy and power, clean water supply, environmental treatment, solid and
hazardous waste, estate management, people, supply chain, and finance and insurance.

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
