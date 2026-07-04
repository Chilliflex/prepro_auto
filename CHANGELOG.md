# Changelog

## [1.0.0b5] — 2026-06-08
### Fixed (critical)
- **Exported `pipeline.py` is now transform-only and reproducible.** Previously the generated script re-`fit`'d scalers, imputers and encoders on whatever data it was given at run time, which caused train/serve skew and leaked test-set statistics (and target-encoding leaked the target). The engine now **persists the fitted state** (means, medians, scaler stats, category maps, target means, learned fill values) at clean time, and the exported script **only transforms** using those stored values. Verified by `tests/test_reproducibility.py` and an end-to-end run on held-out data.
- **One-hot encoding in the exported pipeline is now deterministic.** The category set and output columns are frozen at clean time, so new data always yields the identical columns in the identical order (missing categories → 0, unseen categories → `__other__`), instead of a column set that depended on the incoming data.
- **Target encoding no longer leaks.** Per-category means and the global mean are computed once on the training data and reused; unseen categories fall back to the stored global mean.

### Added
- **AI privacy controls.** New `app/services/privacy.py` masks PII (emails, phones, long IDs, card-like numbers, and all values from PII-named columns) before any sample is sent to an LLM provider. Two settings: `LLM_MASK_PII` (default on) and `LLM_SEND_VALUES` (set off to send no raw values at all). Covered by `tests/test_privacy.py`.
- **`SECURITY.md`** documenting the expression-evaluator mechanism, its honest limitations, the single-user/local trust boundary, and the AI data-privacy posture.
- Format-aware input loading in the exported pipeline (CSV / Parquet / Excel / JSON auto-detected) and Parquet output support — previously hardcoded to `pd.read_csv`/`to_csv`.
- 15 new tests (242 total).

### Changed
- README: corrected the data-privacy section to state exactly what the optional AI layer sends (it does **not** claim "no upload to any server" once AI is enabled); added the reproducibility guarantee to the Export step; described the expression evaluator honestly and linked `SECURITY.md`.
- Version bumped to `1.0.0b5`; the app version string is now `1.0.0b5`.

## [Unreleased]
### Added
- `prepro_auto.quickclean(data, target=…, apply_all=…)` — headless one-call cleaning with no browser; returns the cleaned DataFrame plus a per-stage report of what was applied vs. left for review
- New `group_aggregate` transform preset — group-by feature engineering (e.g. mean price per location) without leaving the browser; previously required dropping to a notebook
- Contextual decision-card alternatives: per-column options with their own confidence score and a one-line explanation; one-hot is no longer offered for high-cardinality columns
- Live issue-count badges on each Clean stage tab that update after execution (e.g. missing 1223 → 0 ✓)
- README "See it in action" demo-GIF section

### Changed
- Executing a stage now auto-runs the next stage and shows its cards (no manual "Run stage" click)
- Decision-card dropdown shows each option's confidence and reason, updates live on selection, and keeps your choice after Override
- Removed the redundant "keep as-is" dropdown option (the Skip button already covers it)
- Exported `pipeline.py` is now schema-aware: it embeds the original column list and **warns** when a future dataset adds columns (passed through uncleaned) or drops expected ones — instead of silently misleading. Never crashes on schema drift.

## [1.0.0b4] — 2026-05-31
### Fixed
- All README screenshot paths converted to absolute GitHub raw URLs — images now render on PyPI
- PDF documentation link converted to absolute GitHub URL — works on PyPI
- Removed three broken PDF links that returned 404

### Added
- Complete Guide PDF (docs/PrePro_Auto_Complete_Guide.pdf) — 13-page guide with all 11 workflow screenshots
- ROADMAP.md — released versions, in-progress, and planned features
- CONTRIBUTING.md — setup instructions and PR guidelines
- GitHub Actions CI workflow (pytest on Python 3.10/3.11/3.12)
- 11 workflow screenshots in README (all 6 steps + 6 transform sub-sections)

---

## [1.0.0b3] — 2026-05-30
### Added
- Drift detection UI in workbench
- Live RAM panel in sidebar (shows upload limits based on available memory)
- Cache-busting for static assets
- Helpful error messages for common mistakes (CLI vs notebook confusion)

### Changed
- Domain-agnostic engine (removed finance-specific presets)

---

## [1.0.0b2] — 2026-05-30
### Added
- `launch_file(path)` — load any file directly, auto-detects encoding (UTF-8/Latin-1/cp1252/BOM)
- Restructured README with full API reference table
- Explicit note clarifying CLI vs notebook usage

### Changed
- Test count: 196 → 200

---

## [1.0.0b1] — 2026-05-29
### Initial public release
- Notebook bridge: `prepro_auto.launch(df)` opens workbench against an in-memory DataFrame
- Web workbench with 6-step guided flow (Upload → Profile → View → Clean → Transform → Export)
- 5 cleaning stages: missing values (MCAR/MAR/MNAR), outliers (IQR + Hampel + Isolation Forest), scaling (normality-driven), correlation/leakage, encoding (cardinality-routed)
- 17 preset transforms + sandboxed pandas expression evaluator
- AI assistance via 5 providers (Groq, OpenAI, Anthropic, Gemini, Mistral)
- Before/after dashboard with KPI tiles
- Drift detection (PSI + KS)
- Three exports: cleaned data (CSV/Parquet), audit PDF, runnable pipeline script
- 196 tests
