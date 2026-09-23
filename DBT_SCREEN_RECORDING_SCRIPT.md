# OULAD dbt Screen-Recording Guide

This guide provides a simple presentation flow, the screens to show, and an
easy-to-speak script. The recommended recording length is 6 to 7 minutes.

## Main message

The audience only needs to remember this:

> Databricks runs the pipeline. dbt uses SQL models to turn validated Silver
> data into a tested Gold mart. Analytics tables then feed the Databricks
> business and data-quality dashboards.

## Screen-recording sequence

```text
Pipeline diagram
Databricks Workflow
dbt project configuration
Silver source declaration
One dimension model
One fact model
dbt tests
Successful dbt task
Gold schema
Data-quality dashboard
Business dashboard
```

Avoid opening every file. One dimension, one fact, and one test file are enough
to explain how the rest work.

## Before recording

Prepare these tabs before you start:

1. `docs/assets/final-pipeline-with-dq.svg`
2. The Databricks Workflow graph
3. `dbt_project.yml`
4. `models/staging/sources.yml`
5. `models/mart/dim_student.sql`
6. `models/mart/fact_assessments.sql`
7. `models/mart/schema.yml`
8. `tests/dbt/assert_assessment_measures.sql`
9. The successful Databricks `dbt_mart` task logs
10. The `ftw-week-07.03-mart` tables
11. `docs/assets/final-star-schema.svg`
12. The Databricks data-quality dashboard
13. The Databricks business dashboard

Also:

- Increase browser zoom so the code and labels are readable.
- Close unrelated tabs and notifications.
- Collapse sidebars when they cover important content.
- Do not show tokens, passwords, connection strings, or warehouse credentials.
- Use the mouse pointer to guide attention, but do not move it continuously.
- Record only after the Workflow has a successful run that you can show.

# Presentation storyboard and script

## Part 1: Project overview

**Time:** 0:00–0:35  
**Show:** `docs/assets/final-pipeline-with-dq.svg`

### What to point at

Move through the diagram in this order:

```text
CSV → Bronze → Silver → dbt Gold Mart → Analytics → Databricks Dashboards
```

### What to say

> This project transforms the Open University Learning Analytics Dataset into
> tested analytics tables and dashboards. We start with seven CSV files. Bronze
> loads the source data, Silver cleans and validates it, and dbt builds the Gold
> mart. After the mart passes validation, we create the Analytics tables and
> display the results in Databricks dashboards. Data-quality checks run between
> the layers so invalid data does not silently move forward.

### Transition

> I will now show how this process runs inside Databricks.

## Part 2: Databricks Workflow

**Time:** 0:35–1:15  
**Show:** Databricks Workflow graph

### What to point at

Highlight these tasks:

1. Bronze and raw validation
2. Silver and Silver validation
3. `dbt_mart`
4. Post-dbt mart, Analytics, and validation

### What to say

> We used a Databricks Workflow to control the execution order. Bronze must
> succeed before Silver. Silver must succeed before the dbt mart task. The dbt
> task runs the command `dbt build --select path:models/mart`. We did not
> download or install dbt on a personal computer because Databricks already
> provided the connected dbt task and SQL warehouse. If the dbt task succeeds,
> the final notebook validates the mart, builds Analytics, and refreshes the
> dashboard views.

### Important phrase

> Databricks provides the compute and storage. dbt organizes and tests the SQL
> transformations.

## Part 3: dbt project configuration

**Time:** 1:15–1:50  
**Show:** `dbt_project.yml`

### What to highlight

Highlight:

```yaml
model-paths: ["models"]
test-paths: ["tests/dbt"]
macro-paths: ["macros"]
```

Then highlight:

```yaml
+schema: 03-mart
+materialized: table
+file_format: delta
```

### What to say

> This is the main dbt configuration file. It tells dbt where to find the
> models, custom tests, and macros. It also tells dbt to create the mart models
> as physical Delta tables in the `03-mart` schema. Our model files contain
> SELECT queries. dbt generates the required create or replace table commands
> based on this configuration.

### If asked about DDL

> We did not repeat `CREATE OR REPLACE TABLE` in every dbt model. We described
> the desired result with SELECT, and dbt handled the DDL.

## Part 4: Silver sources

**Time:** 1:50–2:15  
**Show:** `models/staging/sources.yml`

### What to highlight

Show:

```yaml
database: ftw-week-07
schema: 02-clean
```

Then scroll through the seven table names.

### What to say

> The source file tells dbt which existing Silver tables it may read. These
> tables already passed Silver validation. A dbt model accesses them using the
> `source` function. This keeps the existing inputs separate from the new tables
> created by dbt.

### Short explanation of `source()`

> `source()` means this table already exists and dbt will read it.

## Part 5: Dimension model

**Time:** 2:15–2:45  
**Show:** `models/mart/dim_student.sql`

### What to highlight

Highlight:

```sql
select distinct
```

Then highlight:

```sql
sha2(cast(id_student as string), 256) as student_key
```

Finally highlight:

```sql
from {{ source('oulad_clean', 'student_info_clean') }}
```

### What to say

> This model creates the student dimension. It selects each student once and
> creates a consistent hashed student key. The model is SQL with Jinja. The SQL
> performs the transformation, while the Jinja `source` function resolves the
> correct Silver table. The other dimensions follow the same pattern, with one
> clearly defined row grain and key.

## Part 6: Fact model

**Time:** 2:45–3:25  
**Show:** `models/mart/fact_assessments.sql`

### What to highlight

Show the generated keys first. Then show the joins between:

- `student_assessment_clean`
- `assessments_clean`
- `student_info_clean`

Finally highlight the measures:

```sql
assessment_weight
is_banked
score
```

### What to say

> This fact model creates one row per student assessment submission. It joins
> the submission to its assessment definition and matching student enrollment.
> It creates keys that connect the fact to the shared dimensions, then keeps the
> assessment measures needed for analysis. Missing scores remain null because a
> blank score is unknown and must not be changed to zero.

### Optional comparison

> Dimensions describe the student, course, date, or demographic profile. Facts
> record measurable events such as an assessment submission or daily VLE
> activity.

## Part 7: dbt tests

**Time:** 3:25–4:05  
**Show:** `models/mart/schema.yml`, then
`tests/dbt/assert_assessment_measures.sql`

### What to highlight in `schema.yml`

Highlight examples of:

```text
not_null
unique
relationships
accepted_values
```

### What to highlight in the custom test

Show:

```sql
where score not between 0 and 100
   or assessment_weight not between 0 and 100
```

### What to say

> The YAML file documents the models and defines standard tests. `not_null`
> checks required fields, `unique` protects the table grain, `relationships`
> checks foreign keys, and `accepted_values` checks approved categories. We
> also added custom SQL tests for project-specific rules. This test returns
> assessment rows with invalid scores or weights. A custom dbt test passes when
> the query returns zero rows.

### Transition

> These model and test files run together through one dbt build command.

## Part 8: Successful dbt execution

**Time:** 4:05–4:35  
**Show:** Databricks `dbt_mart` task logs

### What to highlight

Show:

- `dbt build --select path:models/mart`
- Successful model statuses
- Passing test statuses
- Final successful task status

### What to say

> Here is the actual dbt task inside Databricks. The same command builds the
> models and runs their tests in a controlled process. The task must finish
> successfully before Analytics can continue. If a model or test fails, the
> Workflow stops at this point instead of refreshing the dashboards with
> untrusted data.

### Recording advice

Do not read every log line. Pause briefly on the command, model summary, test
summary, and successful task status.

## Part 9: Gold mart visualization

**Time:** 4:35–5:10  
**Show:** `ftw-week-07.03-mart` table list, then
`docs/assets/final-star-schema.svg`

### What to point at

Point to the five dimensions:

- `dim_student`
- `dim_course`
- `dim_module_presentation`
- `dim_date`
- `dim_demographics`

Then point to the two facts:

- `fact_assessments`
- `fact_vle_interactions`

### What to say

> The successful dbt task creates five dimensions and two fact tables in
> `03-mart`. The facts share the dimensions, so the final design is a galaxy
> schema. `fact_assessments` supports performance analysis, while
> `fact_vle_interactions` supports engagement analysis. Shared dimensions let
> us compare both types of activity using the same students, courses,
> presentations, dates, and demographic profiles.

## Part 10: Data-quality dashboard

**Time:** 5:10–5:45  
**Show:** Databricks data-quality dashboard

### Recommended visuals

Show the dashboard elements that display:

- Latest status by layer
- Passed, warning, and failed check counts
- Failure percentage against threshold
- Recent validation runs

If the dashboard shows the monitored missing values, point out:

```text
1,111 missing IMD bands: 3.409%, limit 5%
45 missing registration dates: 0.138%, limit 1%
173 missing assessment scores: 0.099%, limit 1%
```

### What to say

> The data-quality dashboard shows whether each layer can be trusted. These
> three missing-value checks appear as warnings because the values are genuinely
> missing but remain within our documented limits. We preserve the rows instead
> of guessing values. Only a failed Critical check stops the pipeline. This lets
> us separate a monitored source limitation from a structural problem such as a
> duplicate key or broken relationship.

Do not claim the counts shown above if the current dashboard displays a
different dataset version. Read the current values from the screen instead.

## Part 11: Business dashboard

**Time:** 5:45–6:25  
**Show:** Databricks business dashboard

### Recommended visuals

Choose two or three existing visuals that clearly answer the assignment
questions. Good choices are:

1. Student outcomes or withdrawal rate by module presentation
2. Engagement or VLE clicks compared with performance
3. Assessment scores or pass rate
4. At-risk student count or risk band

Do not show every chart. Choose visuals whose titles and filters are readable.

### What to say

> After the mart passes validation, the Analytics layer prepares the measures
> used by this Databricks dashboard. This view lets us examine student outcomes,
> engagement, assessment performance, and at-risk groups. The dashboard reads
> prepared Analytics tables rather than calculating complex transformations in
> every chart. This keeps the definitions consistent and makes the visuals
> easier to maintain.

When pointing to a specific chart, describe only what the chart actually shows.
Use this pattern:

> This chart compares [measure] across [group]. The current view shows [visible
> observation]. We can use the filter to compare modules or presentations.

Avoid claiming that engagement caused performance. The dashboard shows an
association unless the project includes a causal analysis.

## Part 12: Closing

**Time:** 6:25–6:50  
**Show:** Return to the pipeline diagram or Workflow graph

### What to say

> To summarize, Databricks controls the end-to-end Workflow, while dbt builds
> and tests the Gold mart using SQL models. Silver provides the validated input,
> dbt creates the shared dimensions and facts, and the Analytics layer feeds the
> Databricks dashboards. This design makes the process repeatable and ensures
> that failed Critical checks stop untrusted data before it reaches the final
> visuals.

# What each visual proves

| Visual | What it proves |
|---|---|
| Pipeline diagram | The complete layer order and validation points |
| Databricks Workflow | The steps run in a controlled dependency order |
| `dbt_project.yml` | dbt builds Delta tables in `03-mart` |
| `sources.yml` | dbt reads the validated Silver tables |
| Dimension SQL | dbt uses SQL and Jinja to create dimensions |
| Fact SQL | dbt joins Silver entities and creates measurable facts |
| `schema.yml` | Standard model and relationship tests are defined |
| Custom SQL test | Project-specific rules return invalid rows |
| dbt task logs | Models and tests actually ran successfully |
| Gold table list | The seven expected mart tables exist |
| Galaxy schema | The dimensions shared by the two facts |
| DQ dashboard | Layer health, warnings, failures, and thresholds |
| Business dashboard | The final analytics questions and measures |

# Shorter 3-minute version

If the presentation time is limited, show only:

1. Pipeline diagram
2. Databricks Workflow
3. `dbt_project.yml`
4. `fact_assessments.sql`
5. `schema.yml`
6. Successful dbt task
7. Galaxy schema
8. One DQ visual and one business visual

Use this script:

> Our Databricks Workflow runs Bronze, Silver, dbt Gold, Analytics, and the
> dashboard refresh in dependency order. We did not install dbt locally because
> it was already configured as a Databricks task. The dbt command reads the
> validated Silver sources and builds five dimensions and two facts as Delta
> tables in `03-mart`. The transformation files use SQL with Jinja. YAML defines
> documentation and tests, while custom SQL tests cover project-specific rules.
> The Workflow continues only after the dbt models and tests pass. The Gold mart
> then feeds Analytics, the data-quality dashboard, and the Databricks business
> dashboard.

# Likely questions and short answers

## Did you use Python for dbt?

> No. Our dbt transformations use SQL with Jinja. Databricks ran the connected
> dbt task. The separate PySpark profiling notebook is outside the dbt mart.

## Why use dbt if the transformations are SQL?

> SQL defines the transformation. dbt organizes the models, manages sources and
> dependencies, generates the table DDL, runs the tests, and records the result
> in one repeatable process.

## Is dbt the Gold layer?

> Gold is the data layer. dbt is the tool that builds and tests that layer.

## Why are the dbt models only SELECT statements?

> The project config materializes them as tables. dbt generates the create or
> replace table commands for Databricks.

## What is the difference between `source()` and `ref()`?

> `source()` reads an existing input table, such as a Silver table. `ref()`
> points to a dbt model and creates a dependency. This project uses `ref()`
> mainly in relationship and custom tests.

## What does the macro do?

> It makes dbt use the exact schema name `03-mart` instead of combining it with
> another target schema name.

## How do you know the mart is correct?

> It must pass the generic YAML tests, the custom SQL tests, and the final Gold
> validation before Analytics runs.

## Why are some missing values warnings instead of failures?

> They are real source limitations within the accepted monitoring limits. We
> keep them visible without guessing a replacement. Structural problems such as
> broken keys and relationships are Critical and stop the pipeline.

## Did you use Metabase?

> No. We created and viewed the final dashboards directly in Databricks. The
> Metabase folder in the repository was not part of the executed pipeline.

# Recording delivery tips

- Speak slightly slower than normal conversation.
- Pause for one second after opening each new screen.
- Keep code explanations focused on purpose, not every line.
- Use the exact table and schema names visible on screen.
- If a result differs from this script, describe the current screen instead of
  repeating an old number.
- Record in short sections if your software allows clips to be combined.
- Re-record a section when you expose a credential or say an unsupported claim.
