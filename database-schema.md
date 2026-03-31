# Database Schema Reference

Full column details for all Supabase tables in project `edvmempbcysctlfgtgfa`.

## `ingredients` (188 rows, PK: `id` text)

Core nutritional (per 100g as-is): `protein_per_100g`, `digestibility` (0-1, CHECK), `cost_per_kg`, `fat`, `satfat`, `transfat`, `cholesterol`, `carbs`, `fiber`, `sugar`, `sodium`, `moisture`, `vitamin_d`, `calcium`, `iron`, `potassium`, `fda_name`, `color`.

Amino acids (mg/g protein): `aa_his`, `aa_ile`, `aa_leu`, `aa_lys`, `aa_mc`, `aa_pt`, `aa_thr`, `aa_trp`, `aa_val`.

Extended: `fa_ala`, `fa_la`, `fa_oleic`, `fa_palmitic`, `fa_stearic`, `fiber_insoluble`, `fiber_soluble`, `resistant_starch`, `bcaa_mg_per_g`, `ref_digest_product`, `ref_digest_value`, `ref_digest_source`, `data_sources` (JSONB), `korean_name`.

Metadata: `supplier`, `sub_ingredients`, `notes_spec_ref`, `tags` (text[]), `is_active`, `archived_at`, `archived_by_email`, `created_at`, `updated_at`, `updated_by_email`.

## `formulas` (24 rows, PK: `formula_code` text)

Active: `formula_code`, `display_name`, `piece_weight_g`, `serving_size_g`, `dough_ratio_pct`, `filling_ratio_pct`, `single_component`, `process_template` (JSONB), `source_formula_code` (FK to self), `is_active`, `ratio_mode` (as_mixed/finished, CHECK), `final_moisture_mode` (null/composite/components, CHECK), `final_moisture_whole`, `final_moisture_dough`, `final_moisture_filling`, `formula_type`, `owner_name`, `formula_date`, `notes`.

Deprecated (DO NOT use): `dough_moisture_initial`, `filling_moisture_initial`, `dough_moisture_p1_out`, `filling_moisture_p1_out`, `pp_adjustment_type`, `pp_weight_change_pct`, `pp_moisture_change_pct`, `pp_notes`, `pre_process_snapshot`, `source_batch_id`.

Legacy (keep, don't depend on): `sheet_gid`, `sheet_name`.

## `formula_components` (426 rows, PK: `id` bigint identity)

Active: `formula_code` (FK), `component` (DOUGH/FILLING), `ingredient_id` (FK), `row_type` (ingredient/process_step/header), `display_order`, `grams`, `pct_of_component` (0-1, CHECK), `pct_of_formula`, `supplier`, `lot_code`, `moisture_pct`, `cost_per_kg`, `cost_contribution`, `process_step`, `process_instruction`, `notes`, `notes_processing`.

Batch tracking (all NULL, deferred): `actual_weighed_g`, `actual_pct`, `actual_lot_code`, `actual_supplier`.

## `ingredient_data_points` (6,092 rows, PK: `id` uuid)

Per-field provenance. `ingredient_id`, `variant_id`, `field_name`, `value_numeric`, `value_text`, `value_min`, `value_max`, `confidence` (VAL/QUL/INT/ASS/EST/UNK), `source_type` (tds/coa/lab_report/literature/usda_fdc/korean_db/calculated/user_entered/supplier_email/other), `attachment_id`, `source_document_name`, `source_url`, `source_page`, `source_excerpt`, `is_current`, `superseded_by`, `reported_by_email`, `verified_by_email`, `verified_at`.

## Schema-ready tables (0 rows)

- `ingredient_variants`: supplier-specific variants with full nutritional columns. FK to ingredients.
- `attachments`: document metadata (spec_pdf/coa/image/photo/other). Links to ingredient, variant, formula, or batch. Storage bucket `rd-attachments`.
- `custom_field_definitions` + `custom_field_values`: extensible field system for ingredient/variant/formula/batch entities.
- `documentation_requests`: tracks supplier doc requests (pending/sent/received/processed/cancelled).
- `formula_comments` (7 rows): field-level comments on formulas.
- `import_errors` (0 rows): import error logging.

## Auth/admin tables

- `allowed_emails` (17 rows): signup whitelist.
- `admins` (1 row): admin role.
- `signup_requests` (5 rows): signup approval queue.
- `password_reset_requests` (1 row): PW reset tracking.
- `activity_log` (373 rows): user action log.
