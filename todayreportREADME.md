# NYC Mobility Group B — Final 5-Minute Presentation Guide

This guide follows the final 35-slide deck and the agreed speaker assignment:

- **Nella:** Slides 1–11
- **Nadine:** Slides 12–16, 19–21, and 23–25
- **No narration / optional evidence:** Slides 17–18 and 22
- **Nella:** Slides 26–35

## One story to remember

> **Last week, we built a working NYC Mobility pipeline. This week, we made it safer for other people to depend on by adding automated deployment controls, rerun safety, governance, monitoring, and a Great Expectations quality gate.**

## Important presentation rule

Several pages are **animation builds**, not separate topics. Do not restart the explanation on every click.

- Slides 8–11 = one GX explanation
- Slides 12–14 = one failure-flow explanation
- Slides 19–21 = one controlled-failure explanation
- Slides 23–25 = one “why we created the demo” explanation
- Slides 26–31 = one verdict explanation

Slides 17–18 and 22 are not assigned to a speaker. Skip them if time is short. If the video on Slide 22 is reliable, play only 10–15 seconds without narration.

## Five-minute running order

| Time | Slides | Speaker | Action |
|---|---:|---|---|
| 0:00–0:15 | 1–2 | Nella | Introduce project and sources |
| 0:15–0:40 | 3–4 | Nella | Pipeline and governed quality gate |
| 0:40–1:00 | 5–7 | Nella | Day 9 improvements and chosen tool |
| 1:00–1:30 | 8–11 | Nella | Explain how GX works; click through builds |
| 1:30–2:00 | 12–14 | Nadine | Explain PASS/FAIL behavior |
| 2:00–2:30 | 15–16 | Nadine | Explain CI/CD and governance |
| — | 17–18 | — | Skip; keep as Q&A evidence |
| 2:30–3:00 | 19–21 | Nadine | Explain safe failure simulation |
| 3:00–3:15 | 22 | — | Optional short failure clip |
| 3:15–3:45 | 23–25 | Nadine | Explain why the demo mattered |
| 3:45–4:25 | 26–31 | Nella | Verdict and trade-off |
| 4:25–5:00 | 32–35 | Nella | Team contribution and closing |

---

# Nella — Slides 1–11

## Slides 1–2 — Project and source scope

### Nella says

> “Good day. We are Group B, and our project analyzes NYC Green Taxi mobility using Green Taxi trips, Taxi Zones, and hourly weather data. Traffic Advisory was explored but excluded from the production pipeline because of source gaps.”

### What it means

The production pipeline uses only sources the team could load and validate consistently. Excluding an unreliable source is a production decision, not a missing feature.

## Slides 3–4 — Architecture and quality gate

### Nella says

> “Our pipeline moves data from ingestion to Bronze, Silver, Gold, Analytics and Data Quality views. The final governed gate combines 14 quality checks into one PASS or FAIL decision before dashboard delivery.”

### What it means

- Bronze preserves raw records and audit information.
- Silver cleans and standardizes each source.
- Gold creates the fact and dimension model.
- Analytics and DQ create reporting and monitoring views.
- The final gate decides whether the dashboards may refresh.

## Slides 5–7 — What changed this week

### Nella says

> “Last week, we focused on building the pipeline. This week, we productionized it through CI/CD and governance, reliability controls, idempotent processing, monitoring, and tested recovery. Our assigned open-source tool was Great Expectations, which we applied to the final quality decision.”

### What it means

The team did not rebuild the whole pipeline with GX. GX was added at one meaningful control point where it can stop unhealthy outputs from reaching users.

## Slides 8–11 — How Great Expectations works

### Nella says

> “Spark and SQL first calculate the actual business and data-quality results. Great Expectations then checks whether the 14-result summary is complete, unique, properly documented, and entirely marked PASS. This produces one auditable result for the job.”

### What it means

Think of GX as checking a report card:

1. Spark and SQL calculate the grades.
2. GX checks that the report card is complete and valid.
3. If every required result passes, delivery continues.
4. If one required result fails, delivery stops.

Only the small 14-row summary is converted to Pandas. The full taxi dataset stays in Spark, so the design does not collect roughly 1.2 million records into driver memory.

### Handoff

> “Nadine will now explain what happens when that final gate fails.”

---

# Nadine — Slides 12–16

## Slides 12–14 — What happens when GX fails?

### Nadine says

> “GX acts like a final security guard. If all expectations pass, the pipeline may continue to the dashboards. If even one governed result fails, the consolidated gate fails, both dashboard refreshes are skipped, and the configured failure notification is sent. The demo changes no source or business table.”

### What it means

This is **fail-closed behavior**: kapag hindi mapatunayan na healthy ang output, hihinto ang delivery instead of showing possibly unreliable dashboard data.

## Slide 15 — What changed?

### Nadine says

> “Every pull request now runs CI checks, including 15 automated unit and alignment tests. Relevant pull requests also deploy a development preview. After review and merge, the production workflow deploys the Databricks bundle and may manually run the Source-to-Gold job, including the dashboards.”

### What it means

```text
Change → Pull request → CI tests → Development preview
       → Review → Merge to main → Production deployment
```

The checks have different jobs:

- **CI tests** protect code, files, and configuration.
- **Databricks runtime validation** protects processed data.
- **GX** enforces the final governed release decision.

Production deployment is automated after an approved merge, but running the full Source-to-Gold job remains an intentional manual option in the workflow.

## Slide 16 — Repository ownership and deployment controls

### Nadine says

> “Production changes have named Code Owners, required review, CI validation, and a protected deployment path. Development previews use environment-scoped credentials, while approved changes merged to main can proceed through the production environment.”

### What it means

- `CODEOWNERS` identifies responsible reviewers.
- Branch protection requires a pull request and passing checks.
- GitHub environments hold separate variables and secrets.
- The production environment can require an approver before deployment.

### Honest limitation

GitHub-side governance is implemented. A dedicated Databricks service principal, strict least-privilege permissions, and fully isolated development and production data remain future improvements.

---

# Slides 17–18 — Optional rerun evidence

**No narration. Skip these slides during the five-minute talk.**

Use them only if someone asks whether rerunning duplicates data.

### Q&A answer

> “The pipeline is designed to be idempotent. It checks accepted source files, uses stable keys and MERGE or guarded inserts, and records ingestion batches. In our recorded before-and-after rerun, the result stayed at 1,245,892 rows and 12 batches, with zero duplicate-key issues.”

### Layman explanation

Parang pag-click ng save twice: hindi ito dapat gumawa ng dalawang kopya ng parehong trip. The second run recognizes data already processed and updates or skips it safely.

---

# Nadine — Slides 19–21

## How did we simulate a failure?

### Nadine says

> “For the demo, an isolated branch temporarily changes only the in-memory ‘Analytics validation passes’ result from PASS to FAIL. It does not insert bad source data or modify business tables. GX detects that forced result and stops the job. The demo code remains in unmerged PR number 83, so main and production do not contain the forced-failure flag.”

### What it means

```text
Normal temporary result: PASS
Demo-only override: FAIL
GX decision: fail the job
Dashboard result: both refreshes skipped
```

This tests the safety control without contaminating real data.

### Accuracy line to remember

Do not say the demo flag is committed as `False` in PR #83. Say:

> “The forced-failure code exists only in unmerged PR #83; it is absent from main.”

---

# Slide 22 — Optional failure video

**No speaker is assigned.** Play only 10–15 seconds if the video is ready; otherwise skip it.

Show only:

1. Upstream tasks are green.
2. `consolidated_quality_gate` is red.
3. Both dashboard refreshes are skipped.

Evidence: failure run ID `82696912080900`.

---

# Nadine — Slides 23–25

## Why we created the demo

### Nadine says

> “We created the demo for three reasons. First, to prove the end-to-end architecture and rerun safety. Second, to test whether a controlled failure stops downstream delivery and creates visible evidence. Third, to confirm that CI/CD can deploy an isolated development preview without changing production.”

### What it means

1. **Validate architecture:** task dependencies and the final gate behave correctly.
2. **Test resilience:** the pipeline detects, reports, and recovers from failure.
3. **Streamline CI/CD:** pull requests are tested and relevant changes deploy to development automatically.

### Evidence behind the slide

- Recorded rerun: 1,245,892 rows, 12 batches, zero duplicate-key issues.
- Forced GX failure: job stopped and dashboard refreshes were skipped.
- Fresh recovery run: GX passed and both dashboards refreshed.

### Handoff

> “Nella will close with our verdict on whether Great Expectations is worth using.”

---

# Nella — Slides 26–35

## Slides 26–31 — Verdict: useful, but not always necessary

### Nella says

> “Our verdict is that Great Expectations is useful, but not always necessary. Databricks already provides native data-quality controls with less setup. GX adds a structured expectation vocabulary, one centralized quality gate, and rules that can be reused outside Databricks. For this project it improved governance and auditability, but it also added another dependency and more code to maintain.”

### What it means

| Databricks native controls | Great Expectations |
|---|---|
| Built into the platform | Additional open-source dependency |
| Simpler for a small Databricks-only pipeline | Stronger when rules must be centralized or portable |
| Can warn, drop, or fail records | Provides reusable expectations and structured validation results |

### Correct architecture order

```text
Ingestion → Bronze → Silver → Gold → Analytics and DQ → GX Gate → Dashboards
```

GX comes **after** Analytics and DQ because it validates their consolidated results. It comes **before** the dashboards because it controls whether users receive refreshed outputs.

## Slides 32–34 — Team contribution

### Nella says

> “Different members owned quality, reliability, governance, CI/CD, dashboards, documentation, and the demo evidence. We used pull requests, reviews, fixes, and integration to turn separate workstreams into one productionized pipeline.”

### What it means

The main achievement is not one notebook or one tool. It is that separately owned components now behave as one controlled system.

## Slide 35 — Closing

### Nella says

> “In summary, last week we built the data pipeline. This week, we added the controls that help people trust and operate it: automated checks, controlled deployment, idempotent reruns, governed quality decisions, dashboards, and tested recovery. Thank you.”

The QR code links to the repository for the source code and documentation.

---

# Quick technical reference for Q&A

## What are the 18 Databricks tasks?

They are the job’s individual work steps:

1. `ingestion`
2. `bronze_setup`
3. `bronze_load`
4. `bronze_validation`
5. `silver_green_taxi`
6. `silver_taxi_zones`
7. `silver_weather`
8. `validate_silver_green_taxi`
9. `validate_silver_taxi_zones`
10. `validate_silver_weather`
11. `gold_creation`
12. `gold_validation`
13. `data_quality_views`
14. `business_analytics`
15. `analytics_validation`
16. `consolidated_quality_gate`
17. `refresh_quality_dashboard`
18. `refresh_analytics_dashboard`

The number 18 refers to orchestration tasks—not 18 separate pipelines.

## What are the 15 automated tests?

- 6 ingestion-behavior tests
- 3 source-to-notebook alignment tests
- 6 repository and governance tests

They test code and repository contracts before deployment. They do not replace runtime data validation.

## What exactly does GX check?

GX applies six expectations to the 14-row governed summary:

1. Exactly 14 result rows exist.
2. `check_name` is not null.
3. `check_name` is unique.
4. `quality_attribute` is not null.
5. `status` is not null.
6. Every `status` is `PASS`.

## Why can the pipeline be safely rerun?

- Landing recognizes already accepted source files.
- Bronze uses a stable `source_file` guard.
- The ingestion log uses `MERGE`.
- Silver uses guarded inserts or `MERGE`.
- The Gold fact uses a deterministic `trip_key` with `MERGE`.
- Small dimensions use deterministic `CREATE OR REPLACE` logic.

## What happens after GX fails?

1. The job becomes failed.
2. Downstream dashboard tasks are skipped.
3. The configured failure email is sent.
4. The cause is corrected or the demo override is removed.
5. A fresh rerun reuses the pipeline’s idempotent design.

## Claims to avoid

- Do not say GX validates every raw record directly; Spark and SQL calculate the detailed checks.
- Do not say every deployment automatically runs the production pipeline; the workflow run is an optional manual input.
- Do not say GX is mandatory; the team’s verdict is useful but not always necessary.
- Do not say development and production data are fully isolated; that remains a future improvement.
- Do not say the forced-failure flag is on `main`; it exists only in unmerged PR #83.

## Final checklist

- Practice the handoff from Nella to Nadine after Slide 11.
- Skip Slides 17–18 unless asked about rerun safety.
- Keep Slide 22’s clip to 10–15 seconds or skip it.
- Practice the handoff from Nadine to Nella after Slide 25.
- Do not narrate every animation build as a new topic.
- End by five minutes, even if that means skipping the videos.
