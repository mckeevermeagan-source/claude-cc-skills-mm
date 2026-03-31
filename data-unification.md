# Data Unification: Three Source Problem

The ingredient database draws from three distinct data populations that need to be unified in Supabase with full provenance.

## Source populations (from Master Data Tracker v5)

| Source | Prefix | Count | Origin | State |
|--------|--------|-------|--------|-------|
| **EXIST** | Standard (WHEAT_FLOUR, etc.) | 174 | NL R&D, USDA FDC, supplier specs | Live in Supabase. Best AA coverage (94%+). Weak on fatty acids, fiber fractions |
| **KRP** | KR_ (KR_SALT, etc.) | 131 | Korean Research Platform (Seoul Pulse Lab, Eunji) | Partially in Supabase. Korean nutritional analysis, some AA gaps |
| **TOOL** | Toolkit-specific | 38 | NL toolkit, sensory/functional characterization | Not yet in Supabase. Unique functional/sensory data |

## Confidence tier system

Every field value carries a tier (in `ingredient_data_points.confidence`):

| Tier | Meaning | Source |
|------|---------|--------|
| VAL | Validated by direct analysis | Lab reports, COA |
| QUL | From qualified/official source | Supplier TDS, USDA FDC |
| INT | Internal company data | Korean DB, NL records |
| ASS | Assumed from related data | USDA proxy, literature |
| EST | Rough estimate | Back-calculation, extrapolation |

Goal: move data UP from ASS/EST toward QUL/VAL through supplier document collection.

## Korean ingredient data specifics

Eunji Oh (Seoul) is primary contact.

- **HMMA** (변성고단백): #70, 71, 76 in Korean DB. Macro from direct analysis. AA estimated from CSP blending ratios. Direct AA analysis planned.
- **Wheat gluten**: #61 (진주물산-글루텐), from supplier COA.
- **Seasoning blend, 5'-sodium ribonucleotide, modified tapioca starch**: No data from Korea yet. Discussion pending.
- **Enriched wheat flour** (#77, 영양강화 밀가루): Only AA data. Full analysis planned.
- **`KR_SAMYANG_HMMA_M_CL`**: Missing ALL amino acids in Supabase despite being in F218/F219/F221. Most critical gap.
- **`KR_NL2_MOD_TAPIOCA_STARCH`**: protein=4.74 coincides with cost_per_kg=4.74. Possible entry error.
- **`KR_CASHEW_PASTE`**: fiber=23.79% in Korean data is implausible (USDA=1.0). Kept at USDA.

## Data unification plan (approved, Data Unification Brief v2)

**Phase 1 (now)**: Supplier document collection for all current ingredients. Team divides suppliers. Standardized request package: TDS, ingredient statement, allergen statement, BE disclosure, COA, commercial terms. Host in Protein Pal via File Friend.

**Phase 2**: AI/OCR extraction from hosted documents. Link every field to exact source document/version. Review only exceptions (missing, conflicting, outdated, unclear).

**Phase 3**: NL formula linking. KR evidence linking. TOOL population integration.

**Target**: Launch-ready Hummus Bites dataset by late summer 2026.

Core rule: signed supplier-issued specs are the source of truth for supplier-declared data. AI is intake/transposition, not replacement.

## Referenced documents (in Claude Chat project)

- Data Unification Brief v2 (PPTX)
- try_2_master_data_tracker_FINAL_v6.xlsx
- F218_report.pdf, F219_report.pdf, F221_report.pdf
- formula_compare_20260324_1.xlsx

## PP3DB Excel (legacy reference only)

`PP3DB_protein_pal_master_sheet_8.xlsx` is from the Google Sheets era. Contains F001-F011 tabs with process instructions (F010 has 22 steps). `importer/importer.js` was used for initial migration. **Supabase is the source of truth.** PP3DB is retained only as historical reference for process instruction backfill if needed.
