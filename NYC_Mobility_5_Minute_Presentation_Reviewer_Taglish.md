# NYC Mobility Group B - Updated 5-Minute Presentation Reviewer

Aligned with the revised 21-slide deck. Slides 18-20 now contain the Data Quality results, retention flow, and dashboard screenshot. Slide 21 provides the closing Data Quality summary.

## Read This First Tonight

### The whole project in one sentence

We collected official NYC taxi, Taxi Zone, and weather data, preserved the raw sources, cleaned and validated them through Bronze, Silver, and Gold, then published business and data-quality dashboards protected by an executable end-to-end quality gate.

### The simplest mental model

Think of the pipeline as a restaurant:

- **Ingestion** is receiving the ingredients and checking the delivery.
- **Bronze** is the stockroom. We preserve what arrived and label where it came from.
- **Silver** is food preparation. We standardize, clean, and flag problems.
- **Gold** is the plated meal. The data is organized for analysis.
- **Analytics** answers the business questions.
- **Data Quality** is the final inspection before serving.

### Numbers worth memorizing

| Item | Number | Meaning |
|---|---:|---|
| Green Taxi Bronze, Silver, and Gold rows | 133,367 | All accepted taxi rows were preserved |
| March rows | 44,208 | First monthly Green Taxi batch |
| April rows | 44,238 | Incremental append |
| May rows | 44,921 | Incremental append |
| Taxi Zones | 265 | Official lookup rows |
| Silver Weather hours | 2,208 | Hourly March-May observations |
| Gold Weather members | 2,209 | 2,208 observed hours plus one Unknown member |
| Analytics trip scope | 133,348 | Gold rows minus 19 out-of-range datetime rows |
| Clean record rate | 96.53% | Rows without the standard dashboard DQ flags |
| Missing weather coverage | 11 rows, about 0.01% | Trips assigned to the Unknown Weather member |
| Critical integrity failures | 0 | No broken governed structural checks |
| End-to-end gate | 13 checks | One consolidated PASS or FAIL release decision |

Do not confuse these two totals:

- **133,367** is the complete Gold fact population.
- **133,348** is the March-May Analytics population after filtering 19 out-of-range datetime records.

---

## Important Slide Corrections Before Presenting

1. Slides 18-20 are now visible and usable. Their correct roles are: **Slide 18: Data Quality Results**, **Slide 19: Data Retention Across Layers**, and **Slide 20: Data Quality Dashboard**. Slide 21 is the closing Data Quality Summary.
2. On slide 7, write and say **`failed_checks = 0 -> PASS`** and **`failed_checks > 0 -> FAIL`**. The pipeline always evaluates 13 checks, so `checks = 0` is not the correct decision rule.
3. On slides 14, 18, 19, and 21, describe **4,642 as flag occurrences across the five listed Silver checks**, not as 4,642 distinct affected rows. The exact distinct affected-row count comes from `dq_dashboard_overview.rows_with_any_dq_flag`.
4. On slide 18, rename **“Clean rows”** to **“Clean record rate.”** The displayed value is a percentage.
5. On slide 21, explain that all foreign keys resolve to dimension members while 11 trips use the controlled Unknown Weather member. This is why referential integrity can be 100% while Weather completeness is 99.99%.
6. Do not explain `Unknown` and `N/A` as GPS or shapefile results. The project joins TLC `LocationID` values to the Taxi Zone lookup.

---

## Recommended 5-Minute Flow

| Time | Owner | Slide | Message | What to point at |
|---|---|---:|---|---|
| 0:00-0:15 | You | 1-3 | Project objective | Title and team |
| 0:15-0:35 | You | 5 | End-to-end architecture | Source to dashboards flow |
| 0:35-1:15 | You | 4 | Ingestion protection | HTTP check, deterministic path, SHA-256, metadata |
| 1:15-1:45 | You | 10 | Bronze incremental load | Monthly row counts, file guard, ingestion log |
| 1:45-2:20 | Nella | 12-13 | Silver transformations | Standardization, flags, weather expansion |
| 2:20-2:55 | Nella | 9 | Gold star schema | Fact in the center and four dimensions |
| 2:55-3:45 | You | 17 | Business Analytics | Demand, Weather association, mobility patterns |
| 3:45-4:05 | Nella | 18 | Data Quality results | 96.53%, zero-distance flags, missing Weather and integrity |
| 4:05-4:20 | Nella | 19 | Retention strategy | 133,367 preserved and 133,348 used for date-scoped Analytics |
| 4:20-4:35 | Nella | 20 | Data Quality Dashboard | KPI cards, violations table and zone anomaly chart |
| 4:35-4:50 | Nella | 7 | End-to-end gate | 13 checks and `ASSERT_TRUE(failed_checks = 0)` |
| 4:50-5:00 | You | 21 | Closing | Business-ready outputs with visible quality limitations |

If the timer becomes tight, skip slides 2, 3, 6, 8, 11, and 14 during the main talk. Keep them as backup slides for questions.

---

## Complete 5-Minute Script

### You - Opening and architecture, Slides 1 and 5

“Good day. Our project is an end-to-end NYC Green Taxi data pipeline in Databricks. We combined three production sources: Green Taxi trips, the official Taxi Zone lookup, and hourly Open-Meteo weather. Our goal was to preserve the original evidence, transform it into a reliable star schema, and publish both business analytics and data-quality monitoring.”

“This diagram shows the full flow. Data enters through Ingestion, passes Bronze, Silver, and Gold validation, then branches into business analytics and data-quality views. The last task is a consolidated quality gate, so the job can stop when a governed check fails.”

### You - Ingestion, Slide 4

“For ingestion, we used official source URLs and deterministic filenames. Before saving a response, we check the HTTP status and reject an empty response. We also calculate a SHA-256 hash. If the same file and same content already exist, the result is `IDEMPOTENT_SKIP`. If the filename is the same but the content changed, we stop it for review instead of silently overwriting our raw evidence.”

“Each landed file has metadata such as source system, source URL, timestamp, size, status, and hash. In simple terms, may resibo at fingerprint ang bawat raw file.”

### You - Bronze, Slide 10

“Bronze preserves the source values and only adds provenance like `source_file`, `batch_id`, and `ingested_at`. Green Taxi is incremental by month. March loaded 44,208 rows, April appended 44,238, and May appended 44,921, for 133,367 total rows.”

“The Bronze data load uses a file-level `WHERE NOT EXISTS` guard. The `MERGE` is used for the ingestion-log receipt, not for loading the Bronze trip rows. When we reran March or May, zero new rows were inserted. That is our proof of idempotency.”

### Handoff to Nella

“After Bronze validation passes, the three Silver branches run. Nella will explain how we standardized the data and built the Gold model.”

### Nella - Silver, Slides 12 and 13

“In Silver, we transformed the raw values into consistent analytical fields while preserving every row. Taxi timestamps became local `TIMESTAMP_NTZ`, trip duration was calculated, payment codes received readable labels, and invalid distance or duration values received DQ flags.”

“For example, a 2,500-mile trip is outside our profiling boundary. We set the distance value to null to protect averages, but we keep the row and set `dq_extreme_trip_distance = TRUE`. Weather is different: Bronze stores one raw JSON batch, while Silver expands its aligned arrays into 2,208 hourly rows.”

### Nella - Gold and star schema, Slide 9

“We started with the business process and grain: one accepted Silver taxi record per fact row. The fact stores trip measures such as count, distance, duration, and amounts. Four dimensions provide context: Date, Time, Taxi Zone, and Weather Hour.”

“Date, Time, and Taxi Zone play both pickup and drop-off roles. Weather connects at the pickup hour because that is when demand begins. Separating Weather avoids repeating all Weather attributes inside every trip row and lets us reuse the same hourly observation. Gold retains all 133,367 fact rows.”

### You - Analytics and Business Dashboard, Slide 17

“From Gold, we created three business-ready views. Taxi Demand asks when and where demand is highest. Weather Behavior compares trips across observed weather conditions. Area Mobility compares pickup and drop-off activity by zone.”

“The demand dashboard shows the strongest activity around late afternoon, especially near 17:00, and East Harlem North is the busiest pickup zone. The Weather view shows most trips occurred during Cloudy and Clear conditions, but we describe this as association, not causation. The mobility page highlights directional imbalance, such as much higher pickups than drop-offs in East Harlem North.”

“Every Analytics view has a declared grain and separate validation. Its trip volumes reconcile to the 133,348 in-scope Gold rows.”

### Handoff to Nella

“Business insights are useful only if users can see the condition of the data, so Nella will close with our Data Quality Dashboard and release gate.”

### Nella - Data Quality Results and Retention, Slides 18 and 19

“Slide 18 summarizes the measured results. We evaluated 133,367 Gold rows and recorded a 96.53% clean record rate. The largest issue is zero trip distance, with 4,592 flagged rows. We also found 30 extreme-distance flags, one invalid-duration flag, 11 missing-Weather coverage flags, and 19 records outside the analysis window.”

“The 4,642 shown across the Silver checks represents flag occurrences, not a guaranteed distinct-row count. For the clean rate, the dashboard combines the standard conditions at row level so each affected row is counted once.”

“Slide 19 shows our retention policy. Bronze, Silver, and Gold each contain 133,367 taxi rows, so the row difference is zero. We preserve flagged records in Gold for lineage. Analytics excludes only the 19 records outside the March-May business window, leaving 133,348 in-scope records.”

### Nella - Data Quality Dashboard, Slide 20

“Slide 20 is the monitoring dashboard. It shows the clean rate, completeness, uniqueness, referential integrity, violations by rule, and zone-level anomaly rates. Referential integrity is 100% because missing Weather matches use a controlled Unknown dimension member, while completeness separately reports the 11 missing observations.”

“A PASS means the implemented rules, relationships, lineage, and reconciliations passed. It does not prove that every source value is real-world truth because we do not have an independent trip ledger.”

### Nella - End-to-end gate, Slide 7

“The final notebook consolidates 13 governed checks across the pipeline. It counts failed checks and calls `ASSERT_TRUE`. If `failed_checks = 0`, the job passes. If any critical check fails, the task stops and the release is blocked.”

### You - Closing, Slide 21

“Our final output is therefore both business-ready and auditable: one dashboard explains mobility patterns, while the other explains whether the pipeline data is healthy enough to use. Thank you.”

---

## Your Personal Script - Short Version

Use this if you want to memorize only your parts.

### Ingestion and Bronze

“We ingested Green Taxi Parquet files, the TLC Taxi Zone CSV, and Open-Meteo JSON from official sources. We checked the HTTP response, rejected empty files, used deterministic filenames, and created a SHA-256 hash plus metadata. Same file and same content means `IDEMPOTENT_SKIP`; changed content under the same name is blocked for review.”

“Bronze preserves raw values and adds lineage. Green Taxi loads monthly: 44,208 for March, 44,238 for April, and 44,921 for May, totaling 133,367. A `WHERE NOT EXISTS` source-file guard prevents duplicate Bronze rows. The ingestion log uses `MERGE` to keep one successful receipt per batch.”

### Analytics and Dashboard

“Gold feeds three Analytics views: demand by time and zone, trip behavior by weather, and pickup/drop-off mobility by area. We filter only the 19 records outside the March-May business window, leaving 133,348 in-scope trips.”

“Demand peaks in the late afternoon, especially around 17:00, and East Harlem North leads pickup volume. Cloudy and Clear contain most observed trips, but we say association, not causation. The mobility view shows pickup/drop-off imbalance and peak hours by zone. Separate Analytics tests validate each grain and reconcile volumes back to Gold.”

---

## One End-to-End Example You Can Use

Suppose one Green Taxi trip starts in East Harlem North at 5:00 PM during cloudy weather.

1. **Ingestion:** The monthly Parquet file is downloaded, hashed, and saved with metadata.
2. **Bronze:** The trip remains as received and gets `source_file`, `batch_id`, and `ingested_at`.
3. **Silver:** Timestamps are standardized, duration is calculated, payment code is labeled, and quality flags are added.
4. **Gold:** The trip becomes one fact row linked to the pickup Date, Time, Taxi Zone, and Weather Hour.
5. **Analytics:** It contributes one trip to the 17:00 East Harlem North demand count and to the Cloudy category.
6. **Data Quality:** If its distance is zero, the trip remains present but appears as a flagged record.
7. **Quality gate:** The pipeline checks row preservation, keys, relationships, lineage, and Analytics reconciliation before accepting the run.

---

# Likely Panel Questions and Strong Answers

## 1. Paano kayo nag-transformation?

**Short answer:**

“Layered ang transformation. Bronze preserves the source. Silver standardizes types, derives fields, and adds DQ flags. Gold organizes the validated data into a fact table and reusable dimensions. Analytics then applies question-specific aggregation.”

**Example:**

“Raw pickup and drop-off timestamps stay visible. Silver converts them to local `TIMESTAMP_NTZ`, creates hourly fields, and calculates duration. Gold creates Date and Time keys. The dashboard then groups trips by day and hour.”

**Show:** Slides 12-13, then slide 9.

## 2. Paano kayo nag-ingest?

**Short answer:**

“Official URL to deterministic Databricks Volume path. We validate the HTTP response, reject empty content, calculate SHA-256, write a metadata sidecar, and preserve the existing file on rerun.”

**Important technical detail:**

The landing helper performs the content hash check. Bronze uses `WHERE NOT EXISTS` on `source_file`. The Bronze ingestion log uses `MERGE` for one receipt per batch.

**Show:** Slide 4 and slide 10.

## 3. Paano niyo masasabi na maganda or secured ang Data Quality Dashboard?

**Best correction first:**

“Mas accurate sabihin na reliable and governed ang dashboard. Security and data quality are related but different.”

**Answer:**

“Reliable siya because the metrics come from read-only governed views, every count is traceable to Gold DQ flags, overlapping flags are counted once in the executive clean rate, and critical relationships are checked by the end-to-end gate. For security, credentials stay in GitHub Secrets, Databricks controls workspace permissions, and the GitHub ruleset protects the main branch with required checks and blocked force pushes.”

**Do not say:** “Secure because 96.53% clean.” A clean-rate metric does not prove access security.

**Show:** Slides 18-20 for dashboard trust, then slide 7 for enforcement.

## 4. Ano ang challenges na na-encounter ninyo as a group?

**Answer:**

“Three major challenges: mixed source formats, safe reruns, and agreeing on what to retain versus reject. Taxi was Parquet, zones were CSV, and Weather was nested JSON. We solved repeatability with hashes and file guards. We solved questionable values by preserving rows and adding explicit flags instead of silently deleting them.”

**Team-process addition:**

“We also had to keep notebooks, modular `src`, tests, documentation, and the Databricks job aligned. Pull-request reviews and GitHub checks helped us catch naming and dependency issues.”

## 5. Bakit hiwalay ang `dim_weather_hour`?

**Answer:**

“Different ang grain. One taxi trip is one fact row, while Weather is one observation per hour. If we copy temperature, rain, and wind into every trip row, repeated ang same Weather values thousands of times. A separate dimension reduces repetition, keeps Weather reusable, and gives us one controlled hourly key.”

**Example:**

“If 500 trips started at 5 PM, all 500 can point to one 5 PM Weather row.”

**Extra point:**

The dimension has 2,208 observed hours plus key `0` as an Unknown member, for 2,209 rows.

**Show:** Slide 9. Point to the red Weather dimension and `pickup_weather_hour_key`.

## 6. Bakit hindi tinuloy ang web scraping?

**Answer:**

“We explored Traffic Advisory data, but the available HTML/PDF sources did not provide complete and consistent March-May coverage. Using partial traffic data could bias the analysis. Since it was optional, we documented the exploration and excluded it from production rather than presenting incomplete data as complete.”

**Strong closing:**

“This was a data-governance decision, not simply a technical limitation.”

**Show:** Slide 3.

## 7. Paano kayo nag-come up sa decisions?

**Answer:**

“We started from the business questions and grain, then profiled the sources. We chose rules that were explainable in SQL, preserved lineage, and tested each decision through reconciliation. Team reviews were used for changes such as key design, unknown members, analytical filters, and optional-source scope.”

**Simple framework:**

Business question -> grain -> source evidence -> transformation rule -> validation -> documented decision.

## 8. Ano ang new learnings ninyo?

**Answer:**

“The biggest learning is that clean data does not mean deleting unusual data. A better pipeline preserves evidence, flags the problem, and lets each analysis decide what to filter. We also learned that idempotency, grain, lineage, and executable validation are as important as the transformation itself.”

## 9. How did you come up with the star schema?

**Answer:**

“We began with the business process: one accepted Green Taxi record. That became the fact grain. Numeric measures such as count, distance, duration, and amounts belong in the fact. Descriptive context belongs in dimensions: when, where, and under what Weather conditions.”

**Why role-playing dimensions?**

“The same Date, Time, and Taxi Zone tables answer both pickup and drop-off questions through different foreign keys.”

**Show:** Slide 9.

## 10. Bakit walang Traffic Advisory?

**Answer:**

“Optional siya and incomplete ang required date coverage. We preferred three complete, validated sources over a fourth source that could introduce inconsistent scope and misleading comparisons.”

## 11. Bakit hindi ninyo dinelete ang bad records?

**Answer:**

“Deleting full rows can create silent data loss. We retain the record, nullify only an unusable measure when needed, and preserve a DQ flag. That lets auditors trace the source and lets different business questions use appropriate filters.”

**Example:**

“A trip with extreme distance may still have useful fare, time, zone, and lineage data.”

## 12. Ano ang difference ng Bronze validation, Data Quality Dashboard, at end-to-end gate?

**Answer:**

- **Bronze validation** checks source receipt, counts, provenance, and idempotency.
- **Data Quality Dashboard** monitors row-level issues and aggregated health metrics.
- **End-to-end gate** combines critical checks into one executable release decision.

## 13. Ano ang idempotency?

**Answer:**

“Same input plus same operation should not create duplicate results. Parang pinindot mo ulit ang elevator button: hindi dapat dumating ang dalawang elevator records. Our same-file rerun inserts zero new Bronze rows and keeps one log receipt.”

## 14. Why use `INSERT INTO` in Bronze instead of rebuilding the table?

**Answer:**

“Bronze is incremental. New monthly batches are appended while accepted earlier months remain unchanged. Rebuilding would be slower and could unnecessarily change ingestion metadata.”

## 15. Why is `MERGE` not used for the Bronze trip rows?

**Answer:**

“We use a guarded `INSERT INTO` for the data because our approved batch identity is the source file. `MERGE` is used only for the ingestion-log receipt so the source identifier and batch ID appear once.”

## 16. What happens if an upstream file changes but keeps the same filename?

**Answer:**

“The SHA-256 hash changes. The landing helper does not overwrite the preserved file automatically. It raises the case for review. In production, we would assign a versioned source identifier after approval.”

## 17. Is file-level idempotency enough?

**Answer:**

“It is appropriate for the approved immutable monthly files in this project. At larger production scale, I would add a transactional batch manifest, stronger checksum governance, and controlled versioning for corrected upstream files.”

## 18. Why is Weather one Bronze row but 2,208 Silver rows?

**Answer:**

“Bronze preserves one complete JSON payload. Silver uses `POSEXPLODE` on the hourly time array and uses the array position to align temperature, rain, snow, code, and wind. This creates one row per hour without losing the raw original.”

## 19. Why use `TIMESTAMP_NTZ`?

**Answer:**

“Taxi and Weather are analyzed using local NYC clock time. `TIMESTAMP_NTZ` preserves the source-provided local hour and avoids an accidental timezone conversion during joins. We also keep the Weather timezone and UTC offset for traceability.”

## 20. Why join Weather at pickup time only?

**Answer:**

“The business question is about conditions when demand begins, so pickup time is the consistent exposure point. A future model could add drop-off Weather if the question changes.”

## 21. Why do Time and Weather have key `0`?

**Answer:**

“Key `0` is a controlled Unknown member. It preserves the fact row and keeps the foreign-key relationship valid when context is unavailable. For Time, actual clock hours use keys 1-24, while the displayed `hour_24` stays 0-23.”

## 22. Why are Taxi Zone IDs 264 and 265 retained?

**Answer:**

“They are members of the official TLC Taxi Zone lookup. We test mapping through the actual dimension join. We do not hardcode those IDs as automatically invalid.”

## 23. Is `trip_key` a real-world trip ID?

**Answer:**

“No. TLC did not provide a guaranteed unique trip identifier. Our `trip_key` is a deterministic technical fingerprint for one accepted fact row. We validate that it is non-null and unique in the current dataset, but we do not claim it proves real-world identity.”

## 24. Why did you not deduplicate using vendor, pickup, drop-off, and zones?

**Answer:**

“Different legitimate records can share those fields but have different passenger, distance, or financial values. Removing them would be based on an unproven identity rule. We profiled the groups and preserved the records.”

## 25. Why do Silver and Gold use full refresh while Bronze is incremental?

**Answer:**

“Bronze protects the ingestion history, so it appends new batches. At the current course scale, deterministic full-refresh Silver and Gold builds are simpler and reproducible. Their validation signatures confirm that unchanged inputs produce unchanged business rows.”

## 26. Why is Gold still 133,367 when Analytics is 133,348?

**Answer:**

“The 19 records are outside the requested March-May analytical window, but they remain valid source evidence. Gold preserves them; date-sensitive Analytics filters them using `dq_out_of_range_datetime = FALSE`.”

## 27. Why is the clean rate only 96.53% if referential integrity is 100%?

**Answer:**

“They measure different things. Referential integrity asks whether keys connect correctly. Clean rate also considers record-level validity and completeness flags, especially zero distance. A row can join perfectly to every dimension and still contain a questionable measure.”

## 28. Does PASS mean the source data is accurate?

**Answer:**

“No. PASS means our technical rules, lineage, relationships, and reconciliation checks passed. We have no independent ground-truth trip ledger, so real-world accuracy is explicitly limited.”

## 29. What makes the quality gate enforceable?

**Answer:**

“It is part of the Databricks task dependency graph and ends with `ASSERT_TRUE(failed_checks = 0)`. A failure raises an error, stops the task, and prevents the run from being treated as healthy.”

## 30. Why 13 checks?

**Answer:**

“The 13 checks cover row preservation, retained measures, dimension completeness, key uniqueness, foreign keys and join cardinality, lineage, populated DQ flags, Weather continuity, Analytics validation, and documentation of all seven quality attributes.”

## 31. How did you validate the Analytics views?

**Answer:**

“We test that every view contains rows, each declared grain is unique, hours remain 0-23, and all pickup, drop-off, demand, and Weather volumes reconcile with the in-scope Gold fact population.”

## 32. Why use `COUNT(*)` as demand?

**Answer:**

“The fact grain is one accepted trip row, and `trip_count` is 1 per row. Therefore, counting rows is the direct trip-volume measure for demand.”

## 33. Can you say bad weather caused lower demand?

**Answer:**

“No. We can say the observed trip volume was lower during Rain or Snow conditions in this dataset. The number of hours per condition, seasonality, schedules, and other factors may differ. The result is descriptive association, not causal proof.”

## 34. What does trip-weighted Weather mean?

**Answer:**

“One hourly Weather row repeats for every trip in that hour. A busy hour therefore has more influence on the average than a quiet hour. The metric describes the Weather experienced by trips, not an equal-weight average of hours.”

## 35. What is the strongest business finding?

**Answer:**

“Demand concentrates in late afternoon and East Harlem North has the highest pickup activity. The zone also shows a large pickup-to-drop-off imbalance, which may support fleet-positioning analysis. We call it an opportunity for investigation, not guaranteed profit.”

## 36. Is total amount the same as profit or revenue?

**Answer:**

“It is the reported total trip amount from the source. It includes fare and applicable additions such as tips, tolls, and surcharges. It is not net profit because we do not have operating costs.”

## 37. Why use a `FULL OUTER JOIN` in area mobility?

**Answer:**

“A zone may have pickup activity, drop-off activity, or both. `FULL OUTER JOIN` keeps pickup-only and drop-off-only zones. The final key uses `COALESCE`, and null-safe equality prevents separate duplicate null groups.”

## 38. What does 0.60 pickup-versus-drop-off correlation mean?

**Answer:**

“Zones with more pickups generally also have more drop-offs. It is a moderate positive relationship, not a guarantee for every zone and not proof of causation.”

## 39. Why did you use WMO Weather codes?

**Answer:**

“The source provides coded hourly conditions. We mapped the documented codes into business-friendly groups such as Clear, Cloudy, Drizzle, Rain, Snow, and Thunderstorm. The exact code remains in the view for traceability.”

## 40. Why did you separate Gold creation, Analytics, and validation?

**Answer:**

“They change for different reasons. Gold models reusable business entities. Analytics answers specific questions. Validation checks both without changing them. This separation prevents a dashboard request from rewriting the dimensional model.”

## 41. Why did you not use dbt?

**Answer:**

“The approved implementation uses Databricks notebooks, Delta tables, SQL views, Asset Bundles, and separate validation notebooks. This met the project scope without adding another deployment layer. The modular `src` SQL still keeps one transformation per table or view.”

## 42. How does CI/CD help the data pipeline?

**Answer:**

“CI checks repository and bundle structure before changes are accepted. CD validates and deploys the Databricks Asset Bundle, then runs the dependency-controlled job. Credentials remain in GitHub Secrets rather than source files.”

## 43. What happens if one Silver validation fails?

**Answer:**

“Gold depends on all three Silver validation tasks. If one fails, Gold does not start. This prevents partially validated sources from entering the star schema.”

## 44. What would you improve next?

**Answer:**

“I would standardize Weather ingestion to one production contract, either monthly batches or one requested range, instead of keeping both the approved combined baseline and a monthly demonstration. I would also add stronger batch versioning, monitoring alerts, and a verified external source if accuracy needs to be measured.”

---

## Questions You Should Ask Back When a Panel Question Is Ambiguous

- “Do you mean source accuracy or pipeline consistency?”
- “Are you asking about landing idempotency or Bronze idempotency?”
- “Do you mean full Gold population or the March-May Analytics scope?”
- “Are you asking about dashboard reliability or access security?”

This lets you answer the correct technical question instead of guessing.

---

## Safe Phrases During Defense

Use:

- “Based on our implemented SQL...”
- “The validated condition is...”
- “The dashboard shows an association...”
- “Gold preserves the full population, while Analytics applies the business filter.”
- “We scoped accuracy honestly because no independent ground truth is available.”
- “That is a known extension for production.”

Avoid:

- “The data is 100% accurate.”
- “Weather caused demand to decrease.”
- “All flagged rows are bad trips.”
- “Unknown zones came from GPS coordinates.”
- “`trip_key` is the official TLC trip ID.”
- “`MERGE` loads the Bronze trip rows.”
- “133,367 trips are all in the March-May Analytics scope.”

---

## Final Night-Before Checklist

- Confirm slides 18-20 render correctly in presentation mode.
- Change every “4,642 flagged records” label to “4,642 flag occurrences” unless you replace it with the exact distinct count from `rows_with_any_dq_flag`.
- Rename “Clean rows” on slide 18 to “Clean record rate.”
- Correct slide 7 to `failed_checks = 0` for PASS.
- On slide 21, describe the zone view as pickup-zone anomaly monitoring, not Weather conditions across zones.
- Confirm the Business Dashboard subtitle distinguishes 133,367 Gold rows from 133,348 date-scoped Analytics rows.
- Rename the Weather dashboard borough table to “Trip Volume and Fare by Borough.”
- Confirm the final Databricks pipeline run is green.
- Open the Business and Data Quality dashboards in separate tabs as live-demo backup.
- Keep slides 3, 7, 9, 10, 14, and 15 ready for questions.
- Practice once with a strict five-minute timer.
- During the presentation, explain the “why,” not every SQL line.
