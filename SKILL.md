---
name: protein-pal
description: >
  Complete orientation for the Protein Pal ecosystem - a React/TypeScript + Supabase food R&D platform
  built for Samyang Roundsquare's Pulse Lab team. Use this skill for ANY task: bug fixes, features,
  database changes, formula logic, PDCAAS, formula editor, compare tool, nutrition facts, ingredient
  data, Supabase schema, Vercel deployment, Korean localization, WFM compliance, or satellite apps
  (Compliance Companion, Documentation Dude, File Friend, Supplier Sidekick, Lawsuit Larry).
  Trigger on: Protein Pal, F### formulas, PDCAAS, formula editor, ingredient database, compare tool,
  compliance, WFM, SpecVault, data unification, or any stack component in context of this project.
  ALWAYS read this skill before touching any Protein Pal file.
---

# Protein Pal - Core Reference

**For deep dives, read the appropriate file in `references/` alongside this SKILL.md.**

| Reference file | Read when |
|---------------|-----------|
| `references/database-schema.md` | Any DB query, migration, column question, or data entry task |
| `references/data-unification.md` | Korean data, EXIST/KRP/TOOL sources, confidence tiers, supplier doc workflow |
| `references/satellite-apps.md` | Compliance Companion, File Friend, Supplier Sidekick, Lawsuit Larry, Documentation Dude |
| `references/claude-code-guardrails.md` | EVERY Claude Code session (read BEFORE writing any code) |
| `references/korean-branch-status.md` | i18n, TopBar, translation cleanup, merge blockers |

---

## Quick Links

| Resource | Value |
|----------|-------|
| Live app | https://protein-pal-nine.vercel.app |
| Repo | github.com/mckeevermeagan-source/Protein-Pal-security (private) |
| Production branch | `main` |
| Feature branch | `claude/add-korean-language-support-zMufI` (NOT MERGED) |
| Supabase ref | `edvmempbcysctlfgtgfa` |
| Supabase URL | `https://edvmempbcysctlfgtgfa.supabase.co` |
| Admin email | mckeever.meagan@gmail.com |
| Anthropic key env | `ANTHROPIC_API_KEY_TWO` (Vercel, server-side only) |
| AI model | `claude-sonnet-4-20250514` |
| Vercel team | `team_hD2C8I8LWlH0WgjUDdcTRdLm` |
| Vercel project | `prj_WDu2FXlE1BnRdityDMEjfgNUB7d2` |

---

## Stack

React 18.2 (CRA) + TypeScript 4.4, Supabase PostgreSQL (RLS on all tables), AG Grid Community 35.1, Recharts 3.7, Supabase auth + email whitelist, react-i18next (Korean branch), Vercel serverless `/api/` routes (7 endpoints), xlsx/jspdf/pptxgenjs/html2canvas for export. Auto-deploy from `main`. `.npmrc` has `legacy-peer-deps=true` (band-aid).

---

## Repo File Tree (Korean branch)

```
api/
  admin/users.ts                -- BROKEN (500 error)
  compare-charts.js, compare-freefield-charts.js, compare-insights.js
  compare-section.js, nutrition-reference-data.js, report-analysis.js
src/
  App.tsx                       -- PDCAAS calculator/dashboard (~511KB)
  Portal.tsx                    -- Auth + launch screen (~41KB)
  AuthProvider.tsx, activityLog.ts, calculations.ts (~21KB)
  DatabaseModule.tsx, DatabaseHome.tsx
  IngredientsListScreen.tsx, IngredientDetailScreen.tsx
  IngredientCreateScreen.tsx, IngredientEditScreen.tsx, IngredientSearchEditor.tsx
  FormulaSheet.tsx (~68KB), FormulaListScreen.tsx (~40KB)
  FormulaCompare.tsx (~161KB), FormulaReport.tsx (~48KB)
  LanguageToggle.tsx
  supabase/auth.ts, supabase/client.ts, supabase/loader.ts
  helpers/saveFormula.ts, helpers/saveIngredient.ts, helpers/getDataSource.ts
  i18n/i18n.ts, i18n/locales/en.json, i18n/locales/ko.json
```

---

## App Structure

### 1. Portal (Portal.tsx) -- Auth + launch cards to App, Compare, Database, Admin
### 2. PDCAAS Calculator (App.tsx) -- Core feature. Ingredient selection, gram weights, PDCAAS calc, FDA NFP, moisture adjustment, Save As to Supabase. Formulation + Packaging tabs.
### 3. Formula Editor (FormulaSheet.tsx) -- AG Grid for DOUGH/FILLING. Grams = source of truth. Save/Save As via saveFormula.ts. Comments system.
### 4. Formula Compare (FormulaCompare.tsx) -- Up to 10 formulas, 11 sections, AI charts/insights, Excel download.
### 5. Formula Report (FormulaReport.tsx, Korean branch) -- PDF/PPTX generation via AI analysis.
### 6. Database Module (DatabaseModule.tsx) -- Sidebar nav to Home, Ingredients CRUD, Formula list/editor.

---

## CRITICAL BUG: Dual Supabase Client

`src/supabase/auth.ts` uses SDK `createClient()` (carries JWT). `src/supabase/client.ts` uses raw `fetch()` with anon key only (no JWT). All data reads go through `client.ts`. RLS blocks these without three band-aid `anon_read_*` policies. **Fix**: make `client.ts` use the SDK client or get the session token from `auth.ts`. Then drop the band-aid policies.

---

## Live DB State (March 29 2026)

| Table | Rows | Notes |
|-------|------|-------|
| `ingredients` | 188 | `korean_name` live. KR_ prefix for Korean |
| `formulas` | 24 | F001-F011 legacy, F210-F221 active |
| `formula_components` | 426 | DOUGH/FILLING links |
| `ingredient_data_points` | 6,092 | Per-field provenance |
| `activity_log` | 373 | |
| Empty schema-ready tables | 0 | `attachments`, `ingredient_variants`, `custom_field_definitions`, `custom_field_values`, `documentation_requests` |

Full column details: see `references/database-schema.md`

---

## PDCAAS Calculation Logic

Extracted to `src/calculations.ts` (Korean branch). App.tsx on main still has inline logic.

Reference pattern WHO/FAO 2-5y (mg/g protein): His=19, Ile=28, Leu=66, Lys=58, M+C=25, P+T=63, Thr=34, Trp=11, Val=35.

Key functions: `computeCompositeNutrients`, `computeAminoAcids`, `computeWeightedDigestibility`, `computePdcaas`, `buildTotalFormula`, `applyMoistureConcentration`, `computeFinishedGood`, `getProteinClaim`, `getFiberClaim`, `scaleToServing`, `fdaRound.*`.

FDA label rule: display UNCORRECTED protein (rounded). %DV uses PDCAAS-CORRECTED protein.

---

## Moisture Adjustment

Applied when `final_moisture_mode` is not null. `nutrient_finished = nutrient_as_mixed * (100 - target) / (100 - initial)`. Modes: null (as-mixed), "composite" (single target), "components" (separate dough/filling targets, ratio re-weighting). `ratio_mode` controls whether dough_ratio_pct is as-mixed or finished.

---

## Data Entry Flows

**Ingredients**: IngredientCreateScreen.tsx. Form with paste-from-Excel. saveIngredient.ts validates, auto-assigns UCRC, writes to `ingredients` + `data_change_log`. Edit: IngredientEditScreen.tsx with concurrency check on `updated_at`.

**Formulas**: FormulaSheet.tsx or App.tsx. Save As via saveFormula.ts: queries MAX(formula_code) matching `^F\d+$`, increments, validates (collision, ingredient existence, % sum ~1.0), inserts formula + components + version record. Update: concurrency check, diff, change log.

---

## Formula Numbering

F001-F011 (legacy), F210-F221 (active). Gap F012-F209 intentional. Save As only. Numbers never recycled. `source_formula_code` tracks lineage.

---

## API Routes

| Route | Purpose |
|-------|---------|
| `/api/compare-charts` | AI chart selection |
| `/api/compare-freefield-charts` | AI free-field charts |
| `/api/compare-insights` | AI nutritional insights |
| `/api/compare-section` | AI section analysis |
| `/api/report-analysis` | AI formula report |
| `/api/nutrition-reference-data` | Reference data |
| `/api/admin/users` | Admin users (BROKEN) |

All AI routes use `ANTHROPIC_API_KEY_TWO`. Korean branch adds duplicated `lang` parameter handling.

---

## Non-Negotiable Coding Rules

1. **Grams = source of truth.** Never store or reverse-calculate from percentages.
2. **No `toLocaleString()`, no `Intl.NumberFormat`.** Dot decimal only, everywhere.
3. **No PDCAAS refactoring from App.tsx without explicit instruction.** Copy first, confirm, then remove original.
4. **Branch discipline.** `git branch --show-current` before any edit. Read files before editing. Never merge `claude/check-wavy-background-NEZR1`.
5. **Auto-deploy fix.** If commit author is `noreply@anthropic.com`: `git commit --amend --author="Meagan McKeever <mckeever.meagan@gmail.com>"`
6. **RLS silent failures.** PATCH returns 200 but drops writes silently. `NOTIFY pgrst, 'reload schema';` after ALTER TABLE.
7. **Anon read policies are a band-aid.** Do not drop until dual-client bug is fixed.
8. **No `<form>` tags.** Use onClick/onChange.
9. **Immutable user copy.** Insert verbatim, change nothing.
10. **Formula numbering is append-only.** Save As only. No overwrite, delete, fork, or batch promotion.
11. **Use `common_name` not `name`** when querying ingredients. Filter `WHERE is_active = true`.

---

## Known Bugs

**Critical**: Dual Supabase client (anon key reads), Admin Users 500, `KR_SAMYANG_HMMA_M_CL` missing all AAs, Korean branch not merged.

**Moderate**: `loader.ts` fetches all 426 components (no server-side filter), `.npmrc` legacy-peer-deps, 5 API routes duplicate lang logic, paste-from-Excel missing korean_name, FormulaSheet hardcoded English.

**Data**: Several proteins missing digestibility, bcaa_mg_per_g mostly 0, fatty acid fractions mostly 0, KR_CASHEW_PASTE fiber implausible, KR_NL2_MOD_TAPIOCA_STARCH protein/cost collision.

---

## People

| Name | Role |
|------|------|
| Pal (Meagan McKeever) | Owner, sole builder. mmckeever@roundsquare.ai / mckeever.meagan@gmail.com |
| Eunji Oh | Pulse Lab R&D Seoul. Korean ingredient data, AA analysis planning |
| Sungbae Byun | Team Lead Pulse Lab Korea. ko.json reviewer |
| Ella Kim | Korea HQ. Primary Korean end user |
| Fredoen | Boss. PhD rigor expected. Budget approver |
| Pablo Melgarejo | R&D Strategy NL. US regulatory, Braun Hagey counsel |
| Joana | SciSpot pilot owner. Complementary to Protein Pal |
| Teun | Brother-in-law, engineer. Built auth |

---

## Claude Code Session Protocol

1. `git branch --show-current`
2. `git status` -- confirm clean
3. Read files before editing
4. One instruction at a time
5. `show complete output, no summary`
6. Specify branch explicitly
7. SQL migrations: test in Supabase SQL Editor first. `NOTIFY pgrst, 'reload schema';` after ALTER TABLE
8. Open PR, never push to main. Pal merges.
9. **Read `references/claude-code-guardrails.md` before writing any code.**
