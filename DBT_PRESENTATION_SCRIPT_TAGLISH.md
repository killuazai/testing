# OULAD dbt Presentation Script

Approximate speaking time: 4 to 5 minutes

## Before recording

Open these screens first:

1. Pipeline diagram
2. Databricks job or run page
3. `dbt_project.yml`
4. `models/staging/sources.yml`
5. `models/mart/dim_student.sql`
6. `models/mart/fact_assessments.sql`
7. `models/mart/schema.yml`
8. Successful dbt task result
9. Gold tables or galaxy schema
10. Databricks DQ dashboard
11. Databricks business dashboard

If you do not have a Workflow graph, show the Databricks task list or run page.
That is enough.

# Script

## 1. Introduction

**Show:** Pipeline diagram

> Hi, I will explain the dbt part of our OULAD data pipeline.
>
> Our complete process starts from the CSV files. We load them into Bronze,
> clean and validate them in Silver, then use dbt to build our Gold Mart.
> After that, we build the Analytics tables and use them for our Databricks
> dashboards.

## 2. Pipeline order

**Show:** Databricks job, task list, or run page

> Ito yung order ng pipeline namin sa Databricks.
>
> First, tatakbo ang Bronze. After passing the Bronze validation, saka tatakbo
> ang Silver. Kapag pasado rin ang Silver validation, saka tatakbo yung dbt
> task.
>
> The dbt task uses this command:

```bash
dbt build --select path:models/mart
```

> Hindi na kami nag-download or nag-install ng dbt locally. Connected na siya
> sa Databricks job namin. Databricks provides the compute and connection, while
> dbt manages the SQL models and tests.

## 3. dbt project configuration

**Show:** `dbt_project.yml`

> Ito yung main configuration file ng dbt project.
>
> Dito nakalagay kung nasaan yung models, tests, and macros. Dito rin namin
> sinabi na ang output ay physical Delta tables inside the `03-mart` schema.
>
> Yung SQL models namin contain SELECT statements. Si dbt na yung gumagawa ng
> required create or replace table command based on this configuration.

## 4. Silver input tables

**Show:** `models/staging/sources.yml`

> Ito naman yung `sources.yml`, located under `models`, then `staging`.
>
> Naka-list dito yung seven clean Silver tables na babasahin ng dbt. Existing
> tables na sila inside `ftw-week-07.02-clean`.
>
> Kapag gumamit kami ng `source()` sa SQL model, dito kinukuha ni dbt yung
> correct catalog, schema, and table name.

## 5. Dimension model

**Show:** `models/mart/dim_student.sql`

> This is one example of a dimension model.
>
> The student dimension selects each student once, then creates a hashed
> student key using SHA-256.
>
> SQL yung ginagamit for the transformation. Yung double curly braces are
> Jinja syntax used by dbt to locate the correct Silver source.

## 6. Fact model

**Show:** `models/mart/fact_assessments.sql`

> This is one example of a fact model.
>
> The assessment fact contains one row per student assessment submission. It
> joins the submission, assessment details, and student enrollment.
>
> It also contains the keys connected to our dimensions, plus the assessment
> type, weight, banked indicator, and score.
>
> Kapag blank ang score, it stays null. Hindi namin siya ginawang zero because
> zero is an actual grade, while null means unknown or not recorded.

## 7. dbt tests

**Show:** `models/mart/schema.yml`

> Sa `schema.yml`, nandito yung documentation and standard tests ng models.
>
> `not_null` checks required values. `unique` checks duplicate keys.
> `relationships` checks if the foreign keys exist in the dimensions.
> `accepted_values` checks if the category is allowed.

**Show:** One file inside `tests/dbt/`

> May custom SQL tests din kami for specific business rules.
>
> For example, this test checks if assessment scores and weights are within
> zero to one hundred. A custom dbt test passes when the query returns zero
> invalid rows.

## 8. Successful dbt run

**Show:** Successful dbt task result or logs in Databricks

> Ito yung actual dbt task result inside Databricks.
>
> The same `dbt build` command creates the models and runs the tests. Kapag may
> model or dbt test na nag-fail, titigil yung pipeline and hindi muna
> magpo-proceed sa Analytics.
>
> In this run, the models and tests completed successfully, so the pipeline was
> allowed to continue.

Do not say that the run succeeded unless the screen actually shows a successful
result.

## 9. Gold data model

**Show:** `03-mart` tables or final galaxy schema

> After the dbt build, ito yung final Gold Mart namin.
>
> We have five dimensions: student, course, module presentation, date, and
> demographics.
>
> We also have two fact tables: assessment submissions and VLE interactions.
>
> Since the two facts share the same dimensions, this is called a galaxy
> schema. The assessment fact supports performance analysis, while the VLE fact
> supports engagement analysis.

## 10. Data-quality dashboard

**Show:** Databricks DQ dashboard

> This Databricks dashboard shows the data-quality results per layer.
>
> It shows which checks passed, which checks have warnings, and if there are
> failed validations.
>
> For example, we found missing IMD bands, registration dates, and assessment
> scores. We kept those records because the missing values came from the source.
> We did not invent replacement values.
>
> Warning means may issue na mino-monitor, pero within the allowed limit.
> Critical failure means may structural problem, such as duplicate keys or
> broken relationships, and that can stop the pipeline.

If the dashboard displays the current counts, you can say:

> We found 1,111 missing IMD bands, 45 missing registration dates, and 173
> missing assessment scores. Their percentages remain within our documented
> warning limits.

Skip the numbers if the current dashboard shows different values.

## 11. Business dashboard

**Show:** Databricks business dashboard

> After all validations passed, the Analytics tables became the source of our
> Databricks business dashboard.
>
> The dashboard helps us review student outcomes, withdrawal patterns,
> assessment performance, VLE engagement, and at-risk students.
>
> The dashboard reads prepared Analytics tables, so pare-pareho yung
> definitions and calculations used by the charts.

When showing a chart, use this simple format:

> This chart compares **[measure]** across **[group]**. Based on the current
> view, we can see **[visible observation]**. We can also use the filter to
> compare different modules or presentations.

Only describe what you can actually see. Do not say that engagement caused a
specific outcome. You can say they are related or associated.

## 12. Closing

**Show:** Pipeline diagram again

> To summarize, Databricks controls the pipeline, while dbt builds and tests
> our Gold Mart using SQL models.
>
> Silver provides the clean input. dbt creates the dimensions and facts.
> Analytics prepares the final measures, then the Databricks dashboards show
> the business and data-quality results.
>
> This process makes the pipeline repeatable and prevents failed critical checks
> from reaching the final dashboards. Thank you.

# Very short version

Use this if you only have around two minutes:

> Our OULAD pipeline starts with Bronze for raw data and Silver for cleaning and
> validation. After Silver passes, Databricks runs our connected dbt task using
> `dbt build --select path:models/mart`.
>
> We did not install dbt locally. Databricks already provides the task and SQL
> warehouse connection. Our dbt transformations use SQL with Jinja. The
> `sources.yml` file identifies the Silver inputs, the SQL models build five
> dimensions and two facts, and `schema.yml` contains the standard tests. We
> also added custom SQL tests for project-specific rules.
>
> When the models and tests pass, dbt creates the Gold tables inside `03-mart`.
> The Analytics layer then prepares the measures used by our Databricks business
> and data-quality dashboards. Critical failures stop the pipeline, while known
> source limitations within the allowed threshold appear as warnings.

# Quick answers

## Did we use SQL or Python?

> We used SQL with Jinja for the dbt transformations. We did not use Python dbt
> models.

## Did we install dbt locally?

> No. dbt was already configured as a task inside our Databricks pipeline.

## Did we use Metabase?

> No. We used Databricks dashboards.

## Why did we use dbt?

> dbt organizes the SQL models, creates the tables, and runs the tests in one
> repeatable process.

## What is `sources.yml`?

> It tells dbt where the existing Silver input tables are located.

## What is `schema.yml`?

> It documents the models and defines tests for keys, required values, allowed
> values, and relationships.

## What is a macro?

> A macro is reusable Jinja logic. Our macro makes dbt use the exact `03-mart`
> schema name.

## What is the Gold Mart?

> It is the final set of dimensions and facts prepared for Analytics.

## What is the difference between a warning and a failure?

> A warning is a monitored issue within the allowed limit. A critical failure
> stops the pipeline.
