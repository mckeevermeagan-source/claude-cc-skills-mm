# Satellite App Roadmap

All apps share Supabase project `edvmempbcysctlfgtgfa`. The shared `ingredients.id` is the linking key.

## Apps

| App | Purpose | Supabase Access | Status |
|-----|---------|-----------------|--------|
| **Compliance Companion** | WFM/retailer ingredient screening, NOVA scoring, additive flagging | **READ-ONLY. Never modifies Supabase.** Reads `ingredients` + external sources (WFM QS, EWG, Open Food Facts) | Spec designed (10-block A-J scoring). E635 double-declaration issue on all 3 Hummus Bites variants |
| **Documentation Dude** | Generate formatted spec sheets, regulatory summaries, label copy, ingredient statements | Read-only for ingredient/formula data; may write to own export tables | Concept only |
| **File Friend** | Document ingestion AI agent. Accepts supplier files (TDS, COA, allergen, BE, quotes). Classifies, extracts via AI/OCR, links fields to exact source document/page/version. This is the "SpecVault" build | Writes to `attachments`, `ingredient_data_points` (provenance). Reads `ingredients` for ID matching | Core infrastructure for data unification. Schema tables exist. Build plan in Data Unification Brief |
| **Supplier Sidekick** | Autonomous supplier document retrieval. Operates its own email address. Monitors gaps, sends standardized requests, receives and routes to File Friend | Reads `ingredients`, `documentation_requests`. Writes to `documentation_requests` (status). Routes files to File Friend | Schema ready. Depends on File Friend first |
| **Lawsuit Larry** | Competitive intelligence: food brand litigation, FTC violations, class actions, labeling violations | External sources (PACER, FDA warnings, state AG filings). Independent of Supabase ingredient data | Concept + `brand-litigation-intel` skill exists |

## Architectural Rules

1. **Compliance Companion never writes to Supabase.** Read-only screening. Issues surfaced to user for manual action.
2. **File Friend is the sole path for document-sourced data into Supabase.** No other satellite app writes nutritional data or provenance records. Prevents provenance chain breaks.
3. **Supplier Sidekick never writes ingredient data.** Manages request/response lifecycle only. Extraction goes through File Friend.
4. Authentication flows through the same Supabase auth for all apps.

## Deferred features (parked, not killed)

| Feature | Status |
|---------|--------|
| Batch tracking | Schema columns exist (actual_weighed_g, etc.), all NULL |
| Process steps UI | Data in formula_components. F010 has 22 steps as reference |
| Component magnet | Pull dough from one formula, filling from another. Designed, not built |
| Formula versioning UI | formula_versions table exists, saveFormula.ts writes to it, no browse UI |
| ProteinPalLM AI agent | RAG over ingredient data. Concept |

## External dependencies

| System | Status | Relationship |
|--------|--------|-------------|
| SciSpot ELN | Parked (Joana) | Lab notebooks. Complementary, not competing. Do not block Protein Pal |
| USDA FDC API | Available | `https://api.nal.usda.gov/fdc/v1/` with `DEMO_KEY`. Two-step search-then-fetch |
| Braun Hagey | Intro call stage | US food claims, FTC/FDA litigation risk. Pablo co-lead |
| Jam MCP | Connected | Visual QA. `listJams orderBy: createdAt` |
| GitHub MCP | Broken | Orphaned connector. Support request drafted (org `1afe57c9-ae21-4352-b292-76a717130273`) |

## Regulatory reference points

- 21 CFR 101.9(d), Appendix B: Nutrition Facts format, font specs
- 81 FR 33742 (2016), 83 FR 65776 (2018): Updated label rules
- FASTER Act: Sesame = Major Nine allergen
- Lupin: Not US major allergen, EU major allergen. Voluntary US disclosure advisable
- "Amount per serving" type size gap: Appendix B says 6pt Helvetica Black, CFR silent
- Korean: 식품등의 표시기준 / MFDS 식품첨가물공전
