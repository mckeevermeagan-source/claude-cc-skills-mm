---
name: protein-pal
description: >
  Complete orientation and working knowledge for the Protein Pal ecosystem - a React/TypeScript + Supabase
  food R&D platform built for Samyang Roundsquare's Pulse Lab team. Use this skill for ANY task involving
  Protein Pal: bug fixes, feature development, database changes, formula logic, PDCAAS calculations,
  formula editor, compare tool, nutrition facts, ingredient data entry, Supabase schema work, Vercel
  deployment, Claude Code instructions, Korean localization, WFM compliance, SpecVault, or any planned
  satellite app (Compliance Companion, Documentation Dude, File Friend, Supplier Sidekick, Lawsuit Larry).
  Trigger whenever the user mentions Protein Pal, F### formulas, PDCAAS, formula editor, ingredient
  database, compare tool, compliance, WFM, or any stack component in context of this project.
---

# Protein Pal - Complete Working Reference (v2, March 29 2026)

## Quick Links

| Resource | Value |
|----------|-------|
| Live app | https://protein-pal-nine.vercel.app |
| Repo | github.com/mckeevermeagan-source/Protein-Pal-security (private) |
| Production branch | `main` |
| Active feature branch | `claude/add-korean-language-support-zMufI` (Korean i18n + TopBar -- NOT MERGED) |
| Supabase project ref | `edvmempbcysctlfgtgfa` |
| Supabase URL | `https://edvmempbcysctlfgtgfa.supabase.co` |
| Admin email | mckeever.meagan@gmail.com |
| Anthropic API key env | `ANTHROPIC_API_KEY_TWO` (Vercel, server-side only, no `REACT_APP_` prefix) |
| AI model | `claude-sonnet-4-20250514` |
| Vercel team ID | `team_hD2C8I8LWlH0WgjUDdcTRdLm` |
| Vercel project ID | `prj_WDu2FXlE1BnRdityDMEjfgNUB7d2` |

---

## Stack (as of Korean branch)

| Layer | Technology |
|-------|-----------|
| Frontend | React 18.2 (CRA) + TypeScript 4.4 |
| Database | Supabase hosted PostgreSQL, RLS enabled all tables |
| Grid | AG Grid Community 35.1 (MIT, formula editor) |
| Charts | Recharts 3.7 |
| Auth | Supabase email/password via `@supabase/supabase-js` + email whitelist |
| i18n | `i18next` 25.10 + `react-i18next` 16.6 + browser language detector |
| Serverless | Vercel `/api/` routes (Node.js, 7 endpoints) |
| Export | `xlsx` 0.18, `jspdf` 4.2, `pptxgenjs` 3.12, `html2canvas` 1.4 |
| Deployment | Vercel auto-deploy from `main` |
| `.npmrc` | `legacy-peer-deps=true` (band-aid, review needed) |

---

## Repo File Tree (Korean branch)

```
/
  api/
    admin/users.ts              -- Vercel serverless, BROKEN (500 error)
    compare-charts.js           -- AI chart selection for compare view
    compare-freefield-charts.js -- AI free-field chart selection
    compare-insights.js         -- AI nutritional insights
    compare-section.js          -- AI section analysis
    nutrition-reference-data.js -- Reference data endpoint
    report-analysis.js          -- AI formula report analysis
  importer/
    importer.js                 -- PP3DB Excel -> Supabase import script
    apps-script/                -- Legacy Google Sheets Apps Script (deprecated)
    __tests__/                  -- Jest tests for importer
  src/
    index.tsx                   -- Entry point, imports i18n
    App.tsx                     -- PDCAAS calculator/dashboard (~511KB, ~12K+ lines)
    Portal.tsx                  -- Auth + launch screen (~41KB)
    AuthProvider.tsx             -- Supabase auth context
    AdminPanel.tsx              -- Admin panel (~34KB)
    DatabaseModule.tsx          -- Database nav shell with sidebar
    DatabaseHome.tsx            -- Database landing page
    IngredientsListScreen.tsx   -- Ingredient table with tag filter
    IngredientDetailScreen.tsx  -- Ingredient detail view with tabs
    IngredientCreateScreen.tsx  -- New ingredient form with paste-from-Excel
    IngredientEditScreen.tsx    -- Edit ingredient with concurrency check
    IngredientSearchEditor.tsx  -- AG Grid ingredient cell editor
    FormulaSheet.tsx            -- AG Grid formula editor (~68KB)
    FormulaListScreen.tsx       -- Formula table (~40KB)
    FormulaCompare.tsx          -- Compare tool (~161KB)
    FormulaReport.tsx           -- PDF/PPTX report generation (~48KB)
    LanguageToggle.tsx          -- EN/KO flag toggle component
    calculations.ts             -- Extracted PDCAAS/nutrient calc logic (~21KB)
    activityLog.ts              -- Activity logging helper
    styles.css                  -- Global styles
    supabase/
      auth.ts                   -- Supabase SDK client (createClient)
      client.ts                 -- Raw PostgREST client (selectAll, updateRow)
      loader.ts                 -- Data loader mapping Supabase -> App shapes
      migrations/               -- SQL migration files
    helpers/
      saveFormula.ts            -- Formula save/update/save-as with validation
      saveIngredient.ts         -- Ingredient save with validation + UCRC assignment
      getDataSource.ts          -- Data source helper
    i18n/
      i18n.ts                   -- i18next config
      locales/
        en.json                 -- English locale strings
        ko.json                 -- Korean locale strings
```

---

## App Structure & UX Flow

The app has four primary surfaces accessible from a launch portal:

### 1. Launch Portal (Portal.tsx)
Central hub. Auth required. Nav cards to: PDCAAS Calculator ("Launch App"), Formula Compare, Database Module. Admin panel accessible to admin users. Korean branch adds LanguageToggle and (planned) unified TopBar.

### 2. PDCAAS Calculator / Dashboard (App.tsx, ~511KB)
The core feature. User selects ingredients, sets gram weights, and the calculator:
- Computes protein contribution per ingredient
- Calculates amino acid score (AAS) per limiting AA
- Applies PDCAAS = AAS x true digestibility (weighted blend)
- Generates FDA Nutrition Facts Panel with rounded values per 21 CFR 101.9
- Supports post-process moisture adjustment (see Moisture Adjustment section)
- **Save As from dashboard** must write a new `F###` record to Supabase
- Formulation tab and Packaging tab (activeTab state)

### 3. Formula Editor (FormulaSheet.tsx, AG Grid)
Two separate AG Grid instances: one for DOUGH component, one for FILLING component.
- Columns: ingredient selector (IngredientSearchEditor.tsx), grams (editable), computed % of component, computed % of formula, optional nutritional columns (22 total), process notes, supplier, lot code
- **Grams are the source of truth** - percentages are always derived, never entered directly
- Save / Save As buttons - Save As assigns the next sequential `F###` via saveFormula.ts
- CSV export per grid
- Comments system (formula_comments table, 7 rows)

### 4. Formula Compare Tool (FormulaCompare.tsx, ~161KB)
Side-by-side comparison of up to 10 formulas.
- Plain HTML table (not AG Grid)
- 11 sections: identity, macros, micros, fiber fractions, fatty acids, amino acids (protein-weighted mg/g protein), PDCAAS block, ingredient list, per-serving parameters, FDA-rounded NFP values, %DV/claims
- Moisture adjustment post-processing applied before display
- Client-side Excel download
- AI chart selection via `/api/compare-charts` and `/api/compare-freefield-charts`
- AI nutritional insights via `/api/compare-insights`
- AI section analysis via `/api/compare-section`

### 5. Formula Report (FormulaReport.tsx, Korean branch only)
PDF/PPTX report generation from formula data. Uses jspdf and pptxgenjs. AI analysis via `/api/report-analysis`.

### 6. Database Module (DatabaseModule.tsx)
Navigation shell with collapsible sidebar. Routes to: DatabaseHome, IngredientsListScreen, IngredientDetailScreen, IngredientCreateScreen, IngredientEditScreen, FormulaListScreen, FormulaSheet.

---

## Dual Supabase Client Problem (CRITICAL BUG)

The app has TWO Supabase access paths that do not share auth state:

1. **`src/supabase/auth.ts`** -- Uses `@supabase/supabase-js` `createClient()`. This client manages auth (login, signup, signOut). It automatically includes the user's JWT on requests. Used by `saveFormula.ts`, `saveIngredient.ts`, and auth-related operations.

2. **`src/supabase/client.ts`** -- Raw `fetch()` calls to PostgREST REST API using only the anon key. No JWT. Used by `loader.ts` for all data reads (ingredients, formulas, formula_components).

**The result**: All data reads go through `client.ts` with only the anon key. RLS blocks these unless the temporary `anon_read_*` policies exist. Three band-aid policies are currently in place:
- `anon_read_ingredients`
- `anon_read_formulas`
- `anon_read_components`

**The fix**: `client.ts` needs to use the Supabase SDK client from `auth.ts` (or get the session token from it) so that data queries carry the authenticated JWT. Then drop the anon_read policies.

---

## Live Database State (verified March 29 2026)

| Table | Rows | PK | Purpose |
|-------|------|----|---------|
| `ingredients` | 188 | `id` (text) | All raw materials. `korean_name` column live. KR_ prefix for Korean ingredients |
| `formulas` | 24 | `formula_code` (text) | F001-F011 (legacy), F210-F221 (active Korean). Max ~F221 |
| `formula_components` | 426 | `id` (bigint identity) | Links formulas to ingredients. DOUGH/FILLING components |
| `ingredient_data_points` | 6,092 | `id` (uuid) | Per-field provenance with confidence tiers (VAL/QUL/INT/ASS/EST) |
| `activity_log` | 373 | `id` (bigint) | User action tracking |
| `formula_comments` | 7 | `id` (bigint) | Field-level comments on formulas |
| `allowed_emails` | 17 | `id` (bigint) | Signup whitelist |
| `admins` | 1 | `id` (bigint) | Admin roles |
| `signup_requests` | 5 | `id` (bigint) | Signup approval queue |
| `password_reset_requests` | 1 | `id` (uuid) | PW reset tracking |
| `import_errors` | 0 | `id` (bigint) | Import error log |
| `ingredient_variants` | 0 | `id` (uuid) | Supplier-specific variants (schema ready) |
| `attachments` | 0 | `id` (uuid) | Document storage metadata (schema ready, SpecVault will use) |
| `custom_field_definitions` | 0 | `id` (uuid) | Extensible field system (schema ready) |
| `custom_field_values` | 0 | `id` (uuid) | Values for custom fields (schema ready) |
| `documentation_requests` | 0 | `id` (uuid) | Supplier doc request tracking (schema ready) |

### `ingredients` key columns

Core nutritional (per 100g as-is): `protein_per_100g`, `digestibility` (0-1, CHECK constraint), `cost_per_kg`, `fat`, `satfat`, `transfat`, `cholesterol`, `carbs`, `fiber`, `sugar`, `sodium`, `moisture`, `vitamin_d`, `calcium`, `iron`, `potassium`, `fda_name`, `color`.

Amino acids (mg/g protein): `aa_his`, `aa_ile`, `aa_leu`, `aa_lys`, `aa_mc`, `aa_pt`, `aa_thr`, `aa_trp`, `aa_val`.

Extended: `fa_ala`, `fa_la`, `fa_oleic`, `fa_palmitic`, `fa_stearic`, `fiber_insoluble`, `fiber_soluble`, `resistant_starch`, `bcaa_mg_per_g`, `ref_digest_product`, `ref_digest_value`, `ref_digest_source`, `data_sources` (JSONB), `korean_name`.

Metadata: `supplier`, `sub_ingredients`, `notes_spec_ref`, `tags` (text[]), `is_active`, `archived_at`, `archived_by_email`, `created_at`, `updated_at`, `updated_by_email`.

### `formulas` key columns

Active: `formula_code`, `display_name`, `piece_weight_g`, `serving_size_g`, `dough_ratio_pct`, `filling_ratio_pct`, `single_component`, `process_template` (JSONB), `source_formula_code` (FK to self), `is_active`, `ratio_mode` (as_mixed/finished, CHECK), `final_moisture_mode` (null/composite/components, CHECK), `final_moisture_whole`, `final_moisture_dough`, `final_moisture_filling`.

Deprecated (exist, DO NOT use in new logic): `dough_moisture_initial`, `filling_moisture_initial`, `dough_moisture_p1_out`, `filling_moisture_p1_out`, `pp_adjustment_type`, `pp_weight_change_pct`, `pp_moisture_change_pct`, `pp_notes`, `pre_process_snapshot`, `source_batch_id`.

Legacy (keep but don't depend on): `sheet_gid`, `sheet_name`.

### `formula_components` key columns

Active: `formula_code` (FK), `component` (DOUGH/FILLING), `ingredient_id` (FK), `row_type` (ingredient/process_step/header), `display_order`, `grams`, `pct_of_component` (0-1 range, CHECK), `pct_of_formula`, `supplier`, `lot_code`, `moisture_pct`, `cost_per_kg`, `cost_contribution`, `process_step`, `process_instruction`, `notes`, `notes_processing`.

Batch tracking (all NULL, deferred): `actual_weighed_g`, `actual_pct`, `actual_lot_code`, `actual_supplier`.

---

## PDCAAS Calculation Logic

**This is extracted to `src/calculations.ts` on the Korean branch.** App.tsx still has inline calculation logic on main. The extracted version is the canonical one going forward.

Reference pattern: WHO/FAO 2-5y (mg/g protein): His=19, Ile=28, Leu=66, Lys=58, M+C=25, P+T=63, Thr=34, Trp=11, Val=35.

Key functions in `calculations.ts`: `computeCompositeNutrients`, `computeAminoAcids`, `computeWeightedDigestibility`, `computePdcaas`, `buildTotalFormula`, `applyMoistureConcentration`, `computeFinishedGood`, `getProteinClaim`, `getFiberClaim`, `scaleToServing`, `fdaRound.*`.

Moisture adjustment modes: null (no adjustment), "composite" (single target for whole formula), "components" (separate dough/filling targets with ratio re-weighting).

FDA label: display UNCORRECTED protein (rounded). %DV uses PDCAAS-CORRECTED protein. Critical regulatory distinction.

---

## Moisture Adjustment Logic

Applied post-PDCAAS calculation when `final_moisture_mode` is not null. Adjusts all nutrients from as-mixed state to finished-good state. On the Korean branch this logic is in `calculations.ts` (`applyMoistureConcentration`, `computeFinishedGood`).

**Formula:**
```
nutrient_finished = nutrient_as_mixed * (100 - moisture_target) / (100 - moisture_initial)
```

Where `moisture_initial` = weighted average moisture of all ingredients as formulated.

**Modes:**
- `null` - no adjustment, display as-mixed
- `"composite"` - single moisture target for whole formula (`final_moisture_whole`)
- `"components"` - separate targets for DOUGH (`final_moisture_dough`) and FILLING (`final_moisture_filling`), then weighted by dough_ratio_pct/filling_ratio_pct. `ratio_mode` controls whether the dough_ratio_pct is as-mixed or finished.

When a formula is saved from the PDCAAS dashboard with a post-process adjustment active, the `final_moisture_mode` and corresponding target value(s) must be written to the `formulas` row.

---

## Data Entry Flow: Ingredients

1. User navigates to ingredient management (IngredientCreateScreen.tsx)
2. Fills form: id (unique slug), common_name, korean_name (Korean branch), all macro fields, all 9 AA fields (mg/g protein), digestibility
3. Optional: fatty acid fractions, fiber fractions, BCAA sum, data_sources JSONB, supplier, tags
4. Paste-from-Excel supported (textarea parses tab-separated values into fields, but PASTE_ORDER does not include korean_name yet)
5. Source data comes from: USDA FoodData Central, supplier COA/spec sheets (Inveja, WOA, AGT), peer-reviewed literature, Korean Research Platform
6. On save: `saveIngredient.ts` validates, auto-assigns UCRC code (queries max, increments, zero-pads to 4 digits), INSERTs to `ingredients`, writes to `data_change_log`, sets `updated_by_email` from auth session, `is_active = true`
7. Edit flow (IngredientEditScreen.tsx): same form, read-only id/UCRC, changed fields highlighted yellow, optimistic concurrency check on `updated_at`

## Data Entry Flow: Formulas

1. User opens Formula Editor (FormulaSheet.tsx) or PDCAAS Calculator (App.tsx)
2. Selects ingredients, enters gram weights per ingredient
3. Sets piece_weight_g, serving_size_g, dough_ratio_pct (for dual-component)
4. Optionally configures moisture adjustment
5. Clicks **Save As** - `saveFormula.ts` handles this:
   - Queries `MAX(formula_code)` from `formulas` where code matches `^F\d+$`
   - Parses integer, increments, formats as `F` + zero-padded number
   - Validates: collision check, ingredient existence, % sum per component (must be ~1.0), concurrency check on update
   - INSERTs new row to `formulas` with `source_formula_code` = current formula
   - INSERTs all component rows to `formula_components`
   - Writes version record to `formula_versions`
   - Sets `updated_by_email` from auth session
6. Update mode: concurrency check against `updated_at`, computes field diff, writes changes to `data_change_log`

---

## Formula Numbering

- Codes are sequential: F001-F011 (legacy), F210-F221 (active Korean variants)
- Gap F012-F209 is intentional (legacy numbering jump)
- Save As always queries `MAX(formula_code)` from formulas where code matches `^F\d+$`, parses int, increments, zero-pads to 3+ digits
- Numbers are never recycled. `source_formula_code` records lineage
- `saveFormula.ts` validates: collision check, ingredient existence check, % sum check per component, concurrency check on update

---

## Vercel API Routes (7 endpoints)

| Route | Purpose |
|-------|---------|
| `/api/compare-charts` | AI selects charts for compare view |
| `/api/compare-freefield-charts` | AI free-field chart selection |
| `/api/compare-insights` | AI nutritional insights |
| `/api/compare-section` | AI section analysis |
| `/api/report-analysis` | AI formula report analysis |
| `/api/nutrition-reference-data` | Reference data endpoint |
| `/api/admin/users` | Admin user management (BROKEN - 500 error) |

All AI routes use `ANTHROPIC_API_KEY_TWO`. Korean branch adds `lang` parameter handling to each route (duplicated pattern, should be shared helper).

---

## Known Bugs & Issues

### Critical

| Bug | Location | Impact |
|-----|----------|--------|
| Dual Supabase client - reads use anon key only | `client.ts` vs `auth.ts` | All data reads bypass auth; band-aid RLS policies required |
| Admin Users tab 500 error | `api/admin/users.ts` | Admin cannot manage users |
| `KR_SAMYANG_HMMA_M_CL` missing all amino acids | Supabase `ingredients` | PDCAAS calculation for formulas using HMMA is wrong |
| Korean branch not merged to main | Git | All Korean i18n, TopBar, extracted calculations.ts, FormulaReport.tsx live only on feature branch |

### Moderate

| Bug | Location | Impact |
|-----|----------|--------|
| `client.ts` `selectAll` fetches entire table (no filter) | `loader.ts` `loadFormulaComponents` | Loads ALL 426 formula_components for every formula, filters client-side |
| `.npmrc` has `legacy-peer-deps=true` | Root | Hides dependency conflicts |
| 5 API routes duplicate lang-handling system prompt logic | `api/*.js` | Maintenance burden, inconsistency risk |
| `IngredientCreateScreen` paste-from-Excel does not include `korean_name` | `PASTE_ORDER` constant | Inconsistent Korean data entry |
| `FormulaSheet.tsx` has untranslated hardcoded English | UI strings | Korean mode shows English in formula editor |

### Data Quality

| Issue | Impact |
|-------|--------|
| Several protein sources missing digestibility values | PDCAAS underestimates for those ingredients |
| `bcaa_mg_per_g` mostly 0 (not backfilled) | BCAA comparison inaccurate |
| Fatty acid fractions all 0 for most ingredients | Fatty acid analysis non-functional |
| KR_CASHEW_PASTE fiber = 23.79% in Korean data (implausible, kept at 1.0 USDA) | Needs Eunji verification |
| KR_NL2_MOD_TAPIOCA_STARCH protein=4.74 coincides with cost_per_kg=4.74 | Possible data entry error |

---

## Unmerged Korean Branch Work

The `claude/add-korean-language-support-zMufI` branch contains changes across ~30+ files and 5 API routes. It is NOT translation-only. Key additions:

1. **i18n infrastructure**: `react-i18next` configured, `en.json`/`ko.json` locale files, `LanguageToggle.tsx` component
2. **`korean_name` column**: Added to ingredients table (live in Supabase), wired into loader.ts, saveIngredient.ts, ingredient CRUD screens, FormulaCompare.tsx
3. **`calculations.ts`**: PDCAAS and nutrient logic extracted from App.tsx into shared module. Used by FormulaCompare.tsx. App.tsx on this branch still has inline logic too (both exist).
4. **`FormulaReport.tsx`**: New file. PDF/PPTX report generation from formula data.
5. **API route lang handling**: All 5 AI routes accept `lang` parameter, adjust system prompt.
6. **`saveFormula.ts` writes to `formula_versions`**: Version tracking on save/update/save-as.

### Merge blockers (resolved or manageable)
- `korean_name` column exists in production Supabase -- no schema blocker
- Mixed-scope PR -- translation + logic changes + new features in one branch

### TopBar plan (NOT YET BUILT)
Phase 1 and Phase 2 Claude Code prompts exist (uploaded documents in this project). They define a unified TopBar component replacing the menu bar in App.tsx, sidebar in DatabaseModule.tsx, and header controls in FormulaCompare.tsx. These have not been executed yet.

---

## Planned Features & Specs (with document references)

### Immediate (before or alongside Korean merge)

| Feature | Spec Location | Risk | Effort |
|---------|--------------|------|--------|
| TopBar unified navigation | Phase 1 + Phase 2 Claude Code prompts (project files) | Medium - touches 5 files | 2 Claude Code sessions |
| Korean translation cleanup | `translation_audit_spec.md` (project file) + `protein_pal_translation_cleanup_tracker.csv` | Low - text changes only | 1-2 sessions after TopBar |
| Fix dual Supabase client | `client.ts` needs to use SDK client | Low - small change, big impact | 1 session |
| Drop anon_read band-aid policies | After client fix | Low | SQL in Supabase dashboard |

### Near-term (next 2-4 weeks)

| Feature | Description | Risk | Dependencies |
|---------|-------------|------|-------------|
| Save As from PDCAAS dashboard | Dashboard "Save As Formula" button writes new F### to Supabase with post-process adjustments reflected | Medium | saveFormula.ts works, needs UI wiring in App.tsx |
| Formula editor simplification | Remove batch logic/forks from FormulaSheet.tsx, enable only Save As | Low | Already mostly done |
| `loader.ts` filter optimization | `loadFormulaComponents` should filter server-side not client-side | Low | Change fetch URL to include ?formula_code=eq.X |

### Medium-term (1-3 months)

| Feature | Description | Risk | Dependencies |
|---------|-------------|------|-------------|
| File Friend (document ingestion) | Supplier spec/COA/TDS upload, AI/OCR extraction, provenance linking. Uses `attachments` + `ingredient_data_points` tables (schema ready). Ingests documents and writes extracted fields with source links. This is the SpecVault build | High - new subsystem | Supabase Storage bucket, Anthropic Files API, Vercel serverless |
| Supplier Sidekick (autonomous retrieval) | Operates its own email address. Monitors `documentation_requests`, sends standardized requests to supplier contacts, receives responses, routes to File Friend for ingestion. Tracks completeness per ingredient | High - needs email integration | File Friend must exist first. Gmail API or dedicated mailbox |
| WFM Compliance Companion | Multi-layer ingredient screening against WFM Quality Standards, 10-block scoring (A-J). READ-ONLY: never modifies Supabase data, only reads from `ingredients` table and external sources | High - new app | 14 data sources identified, E635 double-declaration issue on Hummus Bites |
| Korean UI localization deployment | Merge Korean branch, Sungbae/Ella review of ko.json for food science terminology | Medium - needs human review | TopBar merge first |
| Claude Code database access controls | Read-only Postgres role, changelog trigger, service role key revocation | Medium - Supabase admin work | Pal runs SQL manually |

---

## Satellite App Roadmap

These are future apps sharing the Supabase backend, each deployed as separate Vercel projects or routes:

| App Name | Purpose | Shared Resources | Supabase Access | Status |
|----------|---------|-----------------|-----------------|--------|
| **Compliance Companion** | WFM/retailer ingredient screening, NOVA scoring, additive flagging | Reads `ingredients` table + external sources (WFM QS, EWG, Open Food Facts) | **READ-ONLY. Never modifies Supabase.** | Spec designed (10-block A-J scoring). E635 double-declaration issue identified on all 3 Hummus Bites variants |
| **Documentation Dude** | Generate formatted spec sheets, regulatory summaries, label copy, ingredient statements from formula data | Reads `formulas`, `ingredients`, `formula_components`, Anthropic API | Read-only for data; may write to its own export/output tables | Concept only |
| **File Friend** | Document ingestion AI agent. Accepts uploaded supplier files (TDS, COA, allergen statements, BE disclosures, quotes), classifies type, extracts candidate values via AI/OCR, links each extracted field to exact source document/page/version. This is the "SpecVault" build | Writes to `attachments`, `ingredient_data_points` (with source provenance). Reads `ingredients` for ID matching | Write access to document and provenance tables only | Core infrastructure for data unification. Schema tables exist. Build plan in Data Unification Brief |
| **Supplier Sidekick** | Autonomous supplier document retrieval agent. Operates its own email address. Monitors documentation gaps, sends standardized request packages to supplier contacts, receives and routes responses to File Friend | Reads `ingredients`, `documentation_requests`. Writes to `documentation_requests` (status updates). Routes received files to File Friend | Write access to request tracking only | Schema ready. Email pipeline designed. Depends on File Friend existing first |
| **Lawsuit Larry** | Competitive intelligence on food brand litigation, FTC violations, class actions, labeling violations | External data sources (PACER, FDA warning letters, state AG filings). Does not depend on Supabase ingredient data | Read-only or independent | Concept + `brand-litigation-intel` skill exists |

### Architectural principles

1. All satellite apps share Supabase project `edvmempbcysctlfgtgfa`. The shared `ingredients.id` is the linking key across all systems.
2. **Compliance Companion never writes to Supabase.** It is a read-only screening tool. Issues are surfaced to the user for manual action.
3. **File Friend is the only path for document-sourced data into Supabase.** No other satellite app writes nutritional data or provenance records. Prevents provenance chain breaks.
4. **Supplier Sidekick never writes ingredient data.** It manages request/response lifecycle only. Actual extraction goes through File Friend.
5. Authentication flows through the same Supabase auth for all apps.

---

## Data Unification: Three Source Problem

The ingredient database draws from three distinct data populations that need to be unified in Supabase with full provenance:

### Source populations (from Master Data Tracker v5)

| Source | Prefix / Marker | Count | Origin | Current State |
|--------|----------------|-------|--------|---------------|
| **EXIST** | Standard IDs (WHEAT_FLOUR, CHICKPEA_COOKED, etc.) | 174 rows | Original Protein Pal NL R&D ingredients, USDA FDC, supplier specs | Live in Supabase. Best amino acid coverage (94%+). Weak on fatty acids, fiber fractions |
| **KRP** | KR_ prefix (KR_SALT, KR_WHEAT_GLUTEN_BLEND, etc.) | 131 rows | Korean Research Platform (Seoul Pulse Lab). Eunji's team | Partially in Supabase (some KR_ rows exist). Korean nutritional analysis data, some AA gaps |
| **TOOL** | Toolkit-specific IDs | 38 rows | NL toolkit formulation ingredients, sensory/functional characterization | Not yet in Supabase. Contains functional/sensory data unavailable elsewhere |

### Data quality tier system

Every field value carries a confidence tier (used in `ingredient_data_points.confidence`):

| Tier | Meaning | Source |
|------|---------|--------|
| VAL | Validated by direct analysis | Lab reports, COA with analytical results |
| QUL | From qualified/official source | Supplier TDS, USDA FDC official entries |
| INT | Internal company data | Korean DB, NL formulation records |
| ASS | Assumed from related data | USDA proxy, literature values |
| EST | Rough estimate or calculated | Back-calculation, mass balance, extrapolation |

Goal: move data UP from ASS/EST toward QUL/VAL through supplier document collection (File Friend) and verification.

### Korean ingredient data specifics

Eunji Oh (Seoul) is the primary contact for Korean ingredient data:

- HMMA (변성고단백): Ingredients #70, 71, 76 in Korean DB. Macro data from direct analysis. AA values estimated from CSP blending ratios. Direct AA analysis planned.
- Wheat gluten: #61 (진주물산-글루텐), data from supplier COA.
- Seasoning blend, 5'-sodium ribonucleotide, modified tapioca starch: No nutritional data from Korea yet. Discussion pending on whether flavoring ingredient AA data is needed.
- Enriched wheat flour (#77, 영양강화 밀가루): Only AA data available. Full analysis planned.
- `KR_SAMYANG_HMMA_M_CL` missing all AA values in Supabase despite being in active trial formulas (F218, F219, F221). Most critical data gap.
- `KR_NL2_MOD_TAPIOCA_STARCH` protein=4.74 coincides with cost_per_kg=4.74 -- possible data entry error.
- `KR_CASHEW_PASTE` fiber=23.79% in Korean data is implausible (USDA=1.0). Kept at USDA.

### Data unification plan (approved, Data Unification Brief v2)

Phase 1 (now): Supplier document collection for all current ingredients. Divide suppliers across team. Standardized request package (TDS, ingredient statement, allergen statement, BE disclosure, COA, commercial terms). Host in Protein Pal via File Friend.
Phase 2: AI/OCR extraction. Link every field to exact source document/version. Review only exceptions.
Phase 3: NL formula linking, KR evidence linking, TOOL population integration. Target: launch-ready Hummus Bites dataset by late summer 2026.

### Referenced documents (in this project)
- Data Unification Brief v2 (PPTX)
- try_2_master_data_tracker_FINAL_v6.xlsx (Master Data Tracker)
- formula_compare_20260324_1.xlsx
- F218_report.pdf, F219_report.pdf, F221_report.pdf (Korean control formula reports)
- protein_pal_korean_localization_audit.docx
- translation_audit_spec.md, protein_pal_translation_cleanup_tracker.csv
- PHASE_1_CLAUDE_CODE.md, PHASE_2_CLAUDE_CODE.md (TopBar build prompts)
- UNIFIED_NAV_SPEC.md
- wfm_feature_plan.md

---

## PP3DB Excel (Legacy Reference Only)

The file `PP3DB_protein_pal_master_sheet_8.xlsx` is from the Google Sheets era. It contains original F001-F011 formula tabs with process instructions, batch weights, and metadata. The `importer/importer.js` was used for initial migration.

**This file is not the active data source.** Supabase is the source of truth. PP3DB is retained as historical reference for process instructions (F010 has 22 detailed steps). If process instruction data needs backfilling into `formula_components.process_step`/`process_instruction`, the PP3DB tabs are the reference for a one-time targeted backfill, not an ongoing dependency.

---

## Non-Negotiable Coding Rules

1. **Grams are the source of truth.** Percentages computed from grams, never stored as input. Never reverse-calculate grams from percentage.

2. **No `toLocaleString()`, no `Intl.NumberFormat`.** Dot decimal separator only, everywhere. NL/EU locale would produce comma decimals, breaking Excel copy-paste.

3. **No extraction/refactoring of PDCAAS logic from App.tsx without explicit instruction.** On Korean branch, `calculations.ts` exists as the extracted version. On main, logic is still inline in App.tsx. Do not touch App.tsx calculation logic on main without authorization.

4. **Branch discipline.** Before any edit: `git branch --show-current`. Never assume file contents -- read the file before editing. The branch `claude/check-wavy-background-NEZR1` is dangerous (missing critical files) -- never merge it.

5. **RLS silent failure awareness.** Supabase REST API PATCH returns 200/204 but silently drops writes when RLS blocks. After any `ALTER TABLE`: `NOTIFY pgrst, 'reload schema';`.

6. **No `<form>` tags in React components.** Use onClick/onChange handlers.

7. **Immutable user copy.** Written copy provided by Pal is inserted verbatim, nothing changed.

8. **Formula numbering is append-only.** Save As is the only create mechanism. No overwrite, no delete, no fork, no batch promotion.

9. **`common_name` field for queries.** Use `common_name` (not `name`) when querying ingredients. Filter active with `WHERE is_active = true`.

10. **Vercel "Ready" does not mean functional.** Always verify deployed behavior.

---

## Claude Code Guardrails (EXPANDED)

### Authorization Boundary

Claude Code is in **read-only mode by default** for ALL Supabase and git operations. Without explicit per-action written approval from Pal in the current session:

- **PROHIBITED**: ALTER TABLE, DROP, TRUNCATE, DELETE on any table
- **PROHIBITED**: INSERT or UPDATE to any production Supabase table
- **PROHIBITED**: git push to main
- **PROHIBITED**: Any Vercel production deploy triggered manually
- **PROHIBITED**: Using the service role key for any autonomous database operation
- **PROHIBITED**: Direct SQL changes that bypass app-layer logging

For any SQL that modifies data or schema: write the query, show it to Pal, stop. Pal runs it in Supabase SQL Editor.
For any git operation targeting main: open a PR, stop. Pal merges via GitHub UI.

### Scope Creep Prevention

- **One instruction at a time.** Claude Code receives only structured prompts from Claude Chat. Never direct instructions from Pal.
- **Prompt format**: TASK / FILES TO TOUCH / FILES TO NOT TOUCH / STOP CONDITION / DO NOT / REPORT BACK
- **No multi-feature PRs.** Each Claude Code session produces one focused change.
- **Read before edit.** Every file must be read before modification. Never assume contents.
- **No branch switching.** Include explicit "do not switch branches" in every prompt.
- **Show complete output.** All grep/cat commands use "show complete output, no summary".

### Forensic Markers for Unauthorized Changes

- `KRP-` prefix (vs standard `KR_`) on ingredient IDs = Claude Code originated
- `null` in `updated_by_email` on rows that should have it = direct SQL bypass (not through app UI)
- Pre/post session ingredient exports for diffing are recommended but not yet implemented

### Post-Session Checklist

After every Claude Code session, verify:
1. Correct branch confirmed
2. No unintended files modified (check git diff)
3. Build succeeds (`npm run build`)
4. No secrets committed (check for API keys, connection strings)
5. PR opened (not pushed to main)

---

---

## People

| Name | Role | Context |
|------|------|---------|
| Pal (Meagan McKeever) | App owner, sole builder | Admin, mmckeever@roundsquare.ai (work), mckeever.meagan@gmail.com (personal) |
| Eunji Oh | Pulse Lab R&D, Seoul | Korean ingredient data, formula verification, amino acid analysis planning |
| Sungbae Byun | Team Lead, Pulse Lab Korea | Korean UI reviewer for ko.json terminology |
| Ella Kim | Korea HQ | Primary Korean-language end user |
| Fredoen | Senior manager/boss | Expects PhD-level rigor. Budget approver |
| Pablo Melgarejo | Team Lead, R&D Strategy, NL | US regulatory work, Braun Hagey counsel introduction |
| Joana | SciSpot pilot workstream owner | Positioned as complementary to Protein Pal, not competing |
| Teun | Pal's brother-in-law, software engineer | Built auth system |
| Gabriela | Colleague | PT website help |
| Phillip Baek, Jang Jae Ho | HQ/Korea contacts | |

---

## External Dependencies & Integrations

| System | Status | Relationship to Protein Pal |
|--------|--------|---------------------------|
| SciSpot ELN | Parked (Joana's workstream) | Lab notebooks/experiment records. Complementary, not competing. Do not block Protein Pal work on SciSpot decisions |
| USDA FDC API | Available | `https://api.nal.usda.gov/fdc/v1/` with `DEMO_KEY`. Two-step search-then-fetch. Nutrient name matching requires exact USDA canonical strings |
| Braun Hagey (counsel) | Intro call stage | US food claims substantiation, FTC/FDA litigation risk. Pablo co-lead |
| Jam MCP | Connected | Visual QA. `listJams` with `orderBy: createdAt`. `analyzeVideo` with full URL |
| GitHub MCP | Broken | Orphaned connector record blocking re-addition. Support request drafted to Anthropic (org ID `1afe57c9-ae21-4352-b292-76a717130273`) |

---

## Regulatory Reference Points

- 21 CFR 101.9(d): Nutrition Facts format
- Appendix B to Part 101: Graphic enhancements (font sizes, line weights)
- 81 FR 33742 (2016 Final Rule): Updated label requirements
- 83 FR 65776 (2018 Technical Amendments): Corrected Appendix B
- FASTER Act: Sesame = Major Nine allergen, requires "Contains" statement
- Lupin: Not a US major allergen, but EU major allergen + peanut cross-reactivity. Voluntary US disclosure advisable
- "Amount per serving" type size: Known gap in codified rule text (Appendix B specifies 6pt Helvetica Black, CFR silent)
- Korean: 식품등의 표시기준 / MFDS 식품첨가물공전 for ingredient naming standards

---

## For Claude Code Sessions

Start every session:
1. `git branch --show-current` -- confirm correct branch
2. `git status` -- confirm clean working tree
3. Read any file before editing it
4. One instruction at a time; wait for confirmation
5. Use `show complete output, no summary` in grep/cat
6. Specify branch explicitly in any git operation

SQL migrations:
- Test in Supabase SQL Editor first
- `NOTIFY pgrst, 'reload schema';` after ALTER TABLE
- Large data operations: direct SQL, not via API (bypasses RLS)

Auto-deploy conflicts:
- If Vercel auto-deploy overwrites CLI deploy (commit author `noreply@anthropic.com`): `git commit --amend --author="Meagan McKeever <mckeever.meagan@gmail.com>"`

---

## Deferred Features (parked, not killed)

| Feature | Status | Notes |
|---------|--------|-------|
| Batch tracking | Deferred | Schema columns exist in formula_components (actual_weighed_g, actual_pct, actual_lot_code, actual_supplier), all NULL |
| Process steps UI | Deferred | Data stored in formula_components (process_step, process_instruction). F010 has 22 steps as reference |
| Component magnet | Designed, not built | Pull dough from one formula, filling from another |
| Formula versioning UI | Deferred | formula_versions table exists, saveFormula.ts writes to it, but no UI to browse versions |
| ProteinPalLM AI agent | Concept | RAG over ingredient data. Interesting but not now |

---

## References

The repo may eventually contain a `references/` directory for:
- `db-schema-full.md` - extended column-level notes and migration history
- `pdcaas-logic.md` - full calculation spec with edge cases
- `formula-compare-spec.md` - complete spec for the compare tool

The `calculations.ts` file on the Korean branch IS the canonical PDCAAS/nutrient logic reference. It contains all functions, constants, types, and FDA rounding rules in one 21KB file.
