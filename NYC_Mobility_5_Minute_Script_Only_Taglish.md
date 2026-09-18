# NYC Mobility Group B - Updated 5-Minute Script Only

Aligned with the revised 21-slide deck, including the visible Data Quality slides 18-20 and summary slide 21.

## You - Slides 1 and 5

“Good day. Our project is an end-to-end NYC Green Taxi data pipeline in Databricks. We combined Green Taxi trips, the official Taxi Zone lookup, and hourly Open-Meteo weather. We preserved the raw sources, transformed them into a validated star schema, and published business and data-quality dashboards.”

“The pipeline moves through Ingestion, Bronze, Silver, and Gold. Each layer has validation. Gold then feeds business analytics and data-quality views, followed by one consolidated release gate.”

## You - Slide 4

“For ingestion, we use official source URLs and deterministic filenames. We check the HTTP response, reject empty content, calculate SHA-256, and save metadata. Same file and same content returns `IDEMPOTENT_SKIP`. A changed file with the same name is blocked for review instead of silently replacing the raw evidence.”

## You - Slide 10

“Bronze preserves source values and adds lineage. Green Taxi loads monthly: March has 44,208 rows, April adds 44,238, and May adds 44,921, totaling 133,367. A `WHERE NOT EXISTS` source-file guard prevents duplicate Bronze rows. `MERGE` keeps one ingestion-log receipt per batch. Deliberate reruns inserted zero new rows.”

“After Bronze passes, the three Silver branches run. Nella will explain the transformations and Gold model.”

## Nella - Slides 12 and 13

“Silver standardizes data while preserving every row. We convert local timestamps to `TIMESTAMP_NTZ`, calculate duration, map payment codes, and add DQ flags. If a distance is outside our accepted profiling boundary, the value becomes null but the row and flag remain. Bronze Weather stores one raw JSON batch, while Silver expands it into 2,208 hourly rows.”

## Nella - Slide 9

“The Gold grain is one accepted Silver taxi record per fact row. The fact stores count, distance, duration, and trip amounts. Date, Time, Taxi Zone, and Weather Hour provide reusable context. Weather is separate because one hourly observation can describe many trips. Gold retains all 133,367 rows.”

## You - Slide 17

“Gold feeds three Analytics views: demand by time and zone, behavior by Weather, and pickup/drop-off mobility by area. We filter only 19 out-of-range datetime records, leaving 133,348 in-scope trips.”

“Demand is strongest in late afternoon, especially around 17:00, and East Harlem North leads pickup volume. Most observed trips occurred during Cloudy and Clear conditions, but this is association, not causation. The mobility view shows directional imbalance between pickups and drop-offs by zone. Separate Analytics tests validate each grain and reconcile all volumes to Gold.”

“Business insights are useful only when users can see the condition of the data, so Nella will close with the Data Quality Dashboard.”

## Nella - Slides 18 and 19

“We evaluated 133,367 Gold rows and measured a 96.53% clean record rate. The largest issue is zero trip distance, with 4,592 flags. We also recorded 30 extreme-distance flags, one invalid-duration flag, 11 missing-Weather coverage flags, and 19 records outside the analysis window.”

“The 4,642 total shown for the listed Silver checks means flag occurrences, not necessarily 4,642 distinct rows. The clean-rate view combines the standard conditions at row level so each affected row is counted once.”

“Bronze, Silver, and Gold each retain 133,367 taxi rows. Analytics filters only the 19 records outside the March-May scope, leaving 133,348 records for the current business analysis.”

## Nella - Slide 20

“The Data Quality Dashboard shows the clean rate, completeness, uniqueness, referential integrity, violations by rule, and zone-level anomaly rates. Referential integrity remains 100% because missing Weather matches use a controlled Unknown dimension member. Completeness separately reports those 11 missing observations.”

“PASS means our technical rules, relationships, lineage, and reconciliations passed. It does not mean every source value is independently proven.”

## Nella - Slide 7

“The end-to-end notebook consolidates 13 checks and calls `ASSERT_TRUE`. If `failed_checks = 0`, the job passes. Any failed critical check stops the task and blocks an unhealthy release.”

## You - Closing, Slide 21

“Our output is business-ready and auditable: one dashboard explains mobility patterns, while the other explains whether the pipeline data is healthy enough to use. Thank you.”
