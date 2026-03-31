# Korean Branch & Localization Status

Branch: `claude/add-korean-language-support-zMufI` -- NOT MERGED to main.

## What the branch contains

Changes across ~30+ files and 5 API routes. This is NOT translation-only.

1. **i18n infrastructure**: react-i18next configured, en.json/ko.json locale files, LanguageToggle.tsx
2. **`korean_name` column**: Live in Supabase, wired into loader.ts, saveIngredient.ts, ingredient CRUD, FormulaCompare.tsx
3. **`calculations.ts`**: PDCAAS/nutrient logic extracted from App.tsx into shared module. App.tsx on this branch still has inline logic too.
4. **`FormulaReport.tsx`**: New file. PDF/PPTX report generation.
5. **API route lang handling**: All 5 AI routes accept `lang` parameter (duplicated pattern, should be shared helper).
6. **`saveFormula.ts` writes to `formula_versions`**: Version tracking on save/update/save-as.

## Merge blockers

- `korean_name` column exists in production Supabase -- no schema blocker
- Mixed-scope PR (translation + logic + new features in one branch) -- manageable but review carefully

## TopBar plan (NOT YET BUILT)

Phase 1 and Phase 2 Claude Code prompts exist (in Claude Chat project). They define a unified TopBar replacing:
- Menu bar in App.tsx
- Sidebar in DatabaseModule.tsx
- Header controls in FormulaCompare.tsx

48px sticky bar, icons only, bilingual tooltips. See PHASE_1_CLAUDE_CODE.md and PHASE_2_CLAUDE_CODE.md.

## Translation cleanup (post-TopBar)

See `translation_audit_spec.md` and `protein_pal_translation_cleanup_tracker.csv` in Claude Chat project.

### Still needs Korean wording revision
- `claims.*goodSource*` / `*excellentSource*` families
- `compare.totalProteinG100kcal`, `correctedProteinG100kcal`, `satietyDensity`
- `compare.healthyFats`, `nonBeneficial`
- `ingredientDetail.observations`
- `grid.pctComponent`, `grid.pctComposite`
- `portal.unableToVerifyEmail`

### Translation policy
- **Keep English**: NFP, ingredient statement, allergen/contains statement, on-pack claims, FDA label output
- **Translate to Korean**: All app chrome (buttons, headings, drawers, tooltips, validation messages, dashboard labels, editor UI)

### Files needing translation sweep (after TopBar)
1. FormulaSheet.tsx (biggest remaining pocket)
2. App.tsx (post-TopBar residual)
3. AdminPanel.tsx
4. Portal.tsx
5. Ingredient CRUD validation/error messages

### Strings deliberately left English
- FDA regulatory output
- `generateCalcAudit()` (decision needed: internal UI or regulatory output?)

## Referenced documents
- PHASE_1_CLAUDE_CODE.md, PHASE_2_CLAUDE_CODE.md (TopBar build prompts)
- UNIFIED_NAV_SPEC.md
- translation_audit_spec.md
- protein_pal_translation_cleanup_tracker.csv
- protein_pal_korean_localization_audit.docx
