# Screenshot register reconciliation — 17 Nov 2026 ledger

## Scope and method

- Source register: `Tabel Status Tenant Gedung Kosong 17 Nov 2026.xlsx`, 203 data rows (columns: no., land area, company, block, address, phone).
- Enrichment source: 11 ledger screenshots (industry category, building type, building area, zone, verification status).
- Anchor: block number + land area; name-prefix match only as fallback. Every matched row was checked for land-area conflict.
- Screenshot values fill building area, building type, zone, survey flag; screenshot category overrides the earlier name-based industry mapping. Rows with a blank category keep the prior industry.

## Match summary

- Register rows: 203 · screenshot rows transcribed: 203
- Matched: 203 / 203 · unmatched screenshot rows: 0 · land-area conflicts: 0
- Land area recorded: 194 rows · blank: 9 rows · land total: 344,717 m² (whole m², half-up)
- Building area recorded: 141 rows with built area, 62 rows explicit 0 (kavling / empty plot); built-area total: 69,817 m²
- Survey: 3 verified (Yes), 200 not surveyed (No)

### Zone census

- KIJ1: 114
- KIJ2: 56
- KIJ3: 6
- KIJ5: 8
- KIJ6: 1
- UKM: 18

### Building type census

- TOB: 72
- SFB: 68
- SIB: 31
- KAVLING: 16
- BIZPARK: 10
- COB: 3
- SMH: 2
- ESPACE: 1

### Industry after screenshot override

- logistics: 92
- other: 30
- service: 15
- trade: 13
- chemical: 8
- metal: 8
- textile: 7
- plastic: 5
- machinery: 5
- food: 4
- electronics: 4
- auto: 4
- pharma: 3
- buildmat: 3
- (blank — prior mapping kept): 2

## Verified rows (survey = Yes)

| No. | Company | Block | Zone | Type | Build m² |
| --- | --- | --- | --- | --- | --- |
| 142 | SRI MULIATI | OO 4 D | kij2 | tob | 456 |
| 150 | SURYA SARANA INTERNASIONAL | NN 1 | kij2 | kavling | 0 |
| 166 | PILIPUS TITUS | C1 B 3 | kij6 | tob | 335 |

## Name discrepancies (identity anchored by block + land)

The screenshot company label differs from the workbook name. The record keeps the workbook name; block and land area agree, so the row identity is anchored on those two fields.

| No. | Block | Land m² | Workbook name | Screenshot label |
| --- | --- | --- | --- | --- |
| 3 | P 10 F | 408 | EKO BASUKI | SAM APPAREL MANUF... |
| 4 | P 10 H | 408 | TAEKYUNG JAYA INDONESIA | IR HENDRICK & SUPIAD... |
| 18 | C 14 P | 864 | TANU WIKODHIONO | IANU WIKODHIONO |
| 39 | C 16 AE | 304 | H. CHOYUM | INDOLAKTO |
| 40 | C 2 D | 2145 | DOO WON PRECISION INDONESIA (Tunda) | GANDA PARADE ENERGI |
| 41 | C 2 E | 1303 | DOO WON PRECISION INDONESIA (Tunda) | GANDA PARADE ENERGI |
| 45 | C 16 AJ | 288 | SANKAY JAYA MANDIRI | SANKAYA JAYA MANDIRI |
| 46 | K 1 K | 1380 | DENPOO MANDIRI INDONESIA | TRI EKA KREASI MODA |
| 86 | V 82 R | 568 | ADI KARYA DINAMIKA | INTAN PRESISI |
| 87 | U 22 J2 | 288 | MITRA INTERNASIONAL MANAJEMEN CONSULTING (Tunda) | SPRING SAN CAHAYA |
| 92 | U 19 I | 2749 | ENDANG PONTAN | PERDAGANGAN TEKN... |
| 105 | K 1 M | 1456 | SARANA GRAHA CIPTA | KARYA MANUNGGAL P... |
| 112 | GG 6 H | 2580 | MULTI COLOR JAKARTA INDONESIA | MUSTAM KARYA MAND... |
| 121 | EE 2 A-B | 4068 | FAJAR SURYA LESTARI | FASTRATA BUANA |
| 136 | OO 1 B | 576 | JULIANNY TEDJA | GAGA KONSTRUKSI EN... |
| 141 | OO 4 B | 576 | BLOCK BUSTER BESTARI | ASTI INDOGRAPH |
| 144 | EE 11 K | 360 | CARE LOGISTINDO | CARG LOGISTINDO |
| 148 | RR 5 J | 1914 | SI TECH INDONESIA | SAM APPAREL MANUF... |
| 153 | MM 14 | 1740 | ANDAL GLOBAL MANUFACTURING | ALPHA INTEGRATED |
| 156 | EE 6 L | 864 | DRUGSCREEN LABORATORIES UTAMA | DERMOZONE PRATAM... |
| 162 | A 2-8 | 1426 | YANTORI MUHYIN | OTO KREASI INDO |
| 167 | EE 7 C | 576 | BHINEKA BAJANAS | TRUST STEEL INDONESIA |
| 170 | MM 11 | 1380 | SAM-PLUS INDOTAMA INDUSTRY | TASMA RITELINDO JAYA |
| 171 | FF 1 I | 511 | INDOCARGOMAS PERSADA | ANTAZEP MEDIKA INTE... |
| 180 | WW5 15 | 1189 | INDOCARGOMAS PERSADA | SANKO LEGOS INDON... |
| 198 | BIZ D 07 | 322 | GRAHA BUANA CIKARANG | EKA PRASETIAWATY |

## Source-internal inconsistencies preserved as recorded

| No. | Company | Block | Note |
| --- | --- | --- | --- |
| 121 | FAJAR SURYA LESTARI | EE 2 A-B | Type recorded as KAVLING (empty plot) but building area column shows 959 m²; source figure preserved, no correction applied. |
| 182 | BIZPARK A-3 | A-3 | Industry category blank in screenshot; prior owner industry kept, no override. |
| 181 | BIZPARK B-5 | B-5 | Industry category blank in screenshot; prior owner industry kept, no override. |

## Format rules applied

- Land and building areas are stored as whole square metres (half-up rounding); no decimal places.
- Building area 0 is an explicit no-building value, distinct from a blank unknown field.
- Missing commercial, compliance and quality fields stay blank; no values are invented.
- All 203 records are `Vacant` (source status: GEDUNG KOSONG).
