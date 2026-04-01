# LLM Code Review & Validation Prompt

Use this prompt with an LLM to conduct a thorough review of this ETL + Prompt Chain homework submission:

---

## Context: Telia Data Engineer Internship - ETL + LLM Prompt Chain Homework

You are reviewing a complete homework submission that combines two tasks:
1. **ETL Pipeline**: Extract European weather data from 2 APIs, transform it, load into SQLite, and create analytical views.
2. **LLM Prompt Chain**: Design a 4-step prompt chain that guides an LLM through a complex data transformation task with intermediate validation.

The submission is a single Jupyter notebook with executed cells, CSV outputs, SQLite database, and JSON artifacts.

---

## Task Overview

### Assignment Requirements:
From the file `telia_data_engineer_internship_ha.txt`:
1. **Extract**: Fetch all European countries from RestCountries API + their capital coordinates. Use coordinates to fetch 30 days of weather from Open-Meteo Archive API.
2. **Transform**: Clean and prepare data (check for missing coordinates, convert sunshine duration, normalize formats, deduplicate).
3. **Load**: Save to local database (SQLite chosen here). Keep raw data separate from cleaned data.
4. **Views**: Build 3 analytical views (capitals ranked by avg temp, countries with most rainfall, full 30-day summary).
5. **Verify**: Confirm data integrity.

### Secondary Task (Prompt Chain):
Design a 4-step LLM prompt chain for complex data transformation:
- **Step 1**: Schema analysis – analyze input data structure, identify quality issues
- **Step 2**: Plan generation – create a numbered transformation plan based on schema
- **Step 3**: Execution – apply transformations with intermediate snapshots
- **Step 4**: Validation – compare original vs transformed, flag issues
- Include error-handling contracts (model must output ERROR: ... if ambiguous)
- Include JSON schema validation between steps
- Include a deterministic test case with expected intermediate outputs

---

## What This Notebook Implements

### Sections 1–5: Core ETL Pipeline
1. **Imports & Setup** – pandas, requests, jsonschema, sqlite3, pathlib
2. **RestCountries Extraction** – fetches 53 European countries with capital coordinates
3. **Open-Meteo Extraction** – fetches last 30 days of weather per capital (1,590+ records)
4. **Data Transformation** – cleans data with readable junior-style loops:
   - Normalizes numeric columns (coerce to float)
   - Converts sunshine from seconds to hours
   - Deduplicates by (country, capital, date)
5. **Analytical Views** – generates 3 views as intermediate CSV outputs

### Sections 6–10: LLM Prompt Chain Design & Orchestration
6. **Prompt Templates** – defines 4 prompts (schema, plan, execution, validation)
7. **JSON Schemas** – strict validation schemas for each step output
8. **Orchestration Script** with mock LLM:
   - `run_prompt_chain()` function chains steps N→N+1
   - Validates output at each step before passing to next
   - Mock LLM returns deterministic, structurally valid JSON
9. **Test Case** – 3-row synthetic dataset with duplicates and format issues
   - Expected: schema analysis identifies issues, plan creates 4 steps, execution deduplicates to 2 rows, validation passes
10. **Real-Data Chain Execution** – runs chain on 200 real weather records, saves 4 JSON artifacts

### Sections 11–12: Database & Schema Dump
11. **SQLite Load** – creates 3 normalized tables and 3 views in database
12. **Schema Dump** – generates SQL schema file for submission

---

## Key Artifacts Generated

### CSV Files (in `outputs/`):
- `raw_countries_europe.csv` – 53 countries with coordinates
- `raw_weather_daily.csv` – 1,590 raw weather records
- `raw_weather_skipped_capitals.csv` – log of skipped entries (empty if all succeeded)
- `clean_weather_daily.csv` – cleaned, deduplicated, with sunshine_duration_hours
- `view_capitals_ranked_by_avg_temp.csv` – avg max temp per capital
- `view_countries_most_rainfall.csv` – total rainfall per capital
- `view_full_30_day_summary.csv` – full 30-day aggregated stats

### Database & Schema:
- `weather_etl.db` – SQLite database with tables and views
- `database_schema.sql` – SQL schema dump (deliverable)

### Prompt Chain Artifacts (JSON):
- `prompt_chain_step1_schema_analysis.json` – identified columns, types, quality issues
- `prompt_chain_step2_plan.json` – 4-step transformation plan with assumptions & risks
- `prompt_chain_step3_execution.json` – execution log with before/after snapshots
- `prompt_chain_step4_validation.json` – validation report with check results

---

## Review Checklist

### ETL Pipeline Correctness
- [ ] **API Extraction**: Both APIs are called successfully with proper error handling
- [ ] **Data Completeness**: 53 countries and ~1,590 weather records (30 days × ~53 capitals)
- [ ] **Schema Consistency**: All numeric fields are floats, dates are ISO format strings
- [ ] **Sunshine Duration**: Converted from seconds to hours (divide by 3600)
- [ ] **Deduplication**: Applied on (country, capital, date) with keep='first'
- [ ] **Views**: All 3 views compute correctly and are sortable/usable
- [ ] **Database**: SQLite tables have proper primary keys and foreign keys
- [ ] **No Data Loss**: Document any skipped records and explain why

### Prompt Chain Design
- [ ] **Prompt Templates**: Each of 4 prompts is focused, clear, and defines output format
- [ ] **Error Contract**: Each prompt includes instruction "output ERROR: ..." if ambiguous
- [ ] **JSON Schemas**: Valid JSON Schema objects for steps 1, 2, 3, 4
- [ ] **Validation Between Steps**: `parse_and_validate_json()` enforces schema at each transition
- [ ] **Orchestration**: `run_prompt_chain()` correctly chains steps and passes validated outputs

### Test Case & Determinism
- [ ] **Test Dataset**: Synthetic data includes duplicates, casing issues, format inconsistencies
- [ ] **Expected Behavior**: 
  - Step 1: Identifies schema, quality issues documented
  - Step 2: Plan specifies 4+ transformation steps with clear rationale
  - Step 3: Execution log shows all 4 steps applied
  - Step 4: Validation passes with all checks = PASS
- [ ] **Assertion Checks**: Test uses `assert` statements to verify expected rows/columns after dedup
- [ ] **Reproducibility**: Mock LLM returns same output every run (deterministic)

### Code Quality (Junior-Style but Functional)
- [ ] **Readability**: Uses explicit loops where appropriate, clear variable names
- [ ] **Comments**: Each cell has purpose documented in markdown headers
- [ ] **Error Handling**: API calls include try-catch, timeout handling
- [ ] **No Silent Failures**: Skipped records logged, validation results reported
- [ ] **Execution Order**: Cells can be run top-to-bottom without errors
- [ ] **State Management**: No circular dependencies, each section depends only on prior outputs

### Deliverables & Documentation
- [ ] **Database Dump**: `database_schema.sql` is valid SQL and includes all tables/views
- [ ] **CSV Exports**: All 7 CSV files are present and properly formatted
- [ ] **JSON Artifacts**: All 4 prompt-chain step outputs are valid JSON
- [ ] **Notebook Execution**: All cells have been run; execution count is visible
- [ ] **Output Consistency**: CSV data row counts match database table row counts

---

## Specific Questions to Validate

### ETL Pipeline
1. **Row Count Tracking**: What was the row count before and after deduplication? Why?
2. **Missing Data**: Were any countries skipped? If yes, which ones and why?
3. **Data Quality**: Are all numeric columns properly typed? Any NaN values? Are they expected?
4. **View Accuracy**: Do the top 3 capitals in view_capitals_ranked_by_avg_temp make geographic sense?

### Prompt Chain
1. **Step 1 Quality**: Does the schema analysis identify potential issues (nulls, mixed types, etc.)?
2. **Step 2 Plan**: Are the transformation steps ordered logically? Do they build on each other?
3. **Step 3 Execution**: Does the transformation log show actual before/after row counts?
4. **Step 4 Validation**: Does the validation check deduplication correctness? Any failures?
5. **Error Handling**: If a prompt step received an ERROR from the mock LLM, how was it handled?

### Integration
1. **Test Case**: Did the test case pass (assertion check for 2 rows after dedup)?
2. **Real-Data Chain**: Did the real-data chain on 200 records also validate as PASS?
3. **Prompts to Orchestration**: Can a real LLM API (OpenAI, Claude) be swapped in for mock_llm_call without changing chain logic?

---

## What Success Looks Like

✅ **Successful Submission** has:
- Notebook executes top-to-bottom without errors
- 53 countries and ~1,590 weather rows fully extracted and cleaned
- SQLite database created with normalized schema
- `database_schema.sql` contains all table and view SQL
- 4-step prompt chain runs successfully on test case (assertions pass)
- 4-step prompt chain runs on real data (validation status = PASS)
- All 7 CSV files present in outputs/
- All 4 JSON artifacts present in outputs/
- Code is readable and uses junior-friendly patterns (explicit loops, clear comments)
- No semantic errors in prompts or validation logic

❌ **Red Flags** to audit:
- Missing API data or incomplete extracts
- Silent data loss without logging
- Duplicate function definitions or circular logic
- Prompt templates with ambiguous output format expectations
- JSON schema validation that doesn't actually block invalid outputs
- Test case that doesn't verify expected dedup behavior
- CSV row counts that don't match database tables
- Database schema dump is empty or malformed SQL

---

## Final Validation Instructions

1. **Run the notebook top-to-bottom**: Confirm no errors appear during execution
2. **Check outputs/ folder**: Verify all 11 expected files are present and non-empty
3. **Spot-check CSV data**: Open one view and verify data makes sense (sorted, aggregated correctly)
4. **Inspect database_schema.sql**: Confirm it's valid SQL and includes all tables/views
5. **Review test assertions**: Confirm they validate dedup behavior, row count reduction, schema checks
6. **Read prompt templates**: Confirm they are focused, specify output format clearly, include error contract
7. **Trace chain execution**: Follow how mock_llm_call output flows from step N to N+1
8. **Validate JSON artifacts**: Open step1 JSON and confirm it has required keys (dataset_summary, columns, global_quality_risks)
9. **Spot-check mock LLM logic**: For step 3 (execution), verify it actually applies transformations and tracks row count changes
10. **Confirm dedup behavior**: Verify that 3-row test becomes 2-row after dedup (2 unique keys after normalization)

