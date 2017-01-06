Outdoor Water Efficiency Survey:
Household Sample Creation
================

Generation of a sample of households for the Outdoor Water Efficiency Survey. The sample is generated using data stored primarily in the CaDC SCUBA data warehouse.

First connect to the database.

``` r
library(DBI)
library(RPostgreSQL) 
source("../db_config.R")
db <- dbConnect(RPostgreSQL::PostgreSQL(), user=db_user, password=db_password, dbname=db_name, host=db_host, port=db_port)
# knitr::opts_chunk$set(connection = "db")
```

We want the ability to sample our tables, and the easiest wat to do that in Postgres is using the `TABLESAMPLE` command. `TABLESAMPLE` can only be used with actual tables and materialized views, so we will have to forgo our favorite [common table expressions](https://www.postgresql.org/docs/9.1/static/queries-with.html) (aka the `with` clause). Instead we create a new materialized view, or refresh it in this case as it has already been created.

``` sql
CREATE MATERIALIZED VIEW sandbox.census_blockgroups_in_sawpa AS (
    SELECT c.*
    FROM census_blockgroup_polygons_2016 c, sandbox.sawpa_boundary_polygon s
    WHERE ST_Intersects(c.geom_3310, s.geom_3310)
)

--REFRESH MATERIALIZED VIEW sandbox.census_blocks_in_sawpa
```

``` sql
SELECT count(*) FROM sandbox.census_blockgroups_in_sawpa
```

| count |
|:------|
| 3157  |

Now we have a view of all the 2016 census blockgroups within the SAWPA service area. We can now sample from these to get the first level of our sample. A sample fraction of 1.5% gets us to approximately the number of block groups that we want.

Next, the sampled block groups are joined with parcel polygons to get a list of all parcels within the sampled block groups. Additionally, these parcels are filtered to only parcels with an area less than 10,000 sqare meters (~2.5 acres). This does a decent, but not perfect job of filtering out large commercial and industrial parcels.

``` sql
CREATE TABLE sandbox.parcels_in_blockgroup_sample AS 

WITH sawpa_blockgroup_sample AS (
  SELECT * 
  FROM sandbox.census_blockgroups_in_sawpa TABLESAMPLE BERNOULLI (1.5) REPEATABLE (1234)
),

parcels_in_blockgroup_sample AS (
  SELECT a.*, b.geoid as blockgroup_id
  FROM assessor_polygons a, sawpa_blockgroup_sample b
  WHERE ST_Intersects(a.geom, b.geom_3310)
  AND a.shape_area < 10000
)

SELECT * from parcels_in_blockgroup_sample
```

``` sql
SELECT count(*) 
FROM sandbox.census_blockgroups_in_sawpa TABLESAMPLE BERNOULLI (1.5) REPEATABLE (1234)
```

| count |
|:------|
| 35    |

``` sql
SELECT count(*) FROM sandbox.parcels_in_blockgroup_sample
```

| count |
|:------|
| 17780 |

<!-- ```{sql, connection=db, tab.cap = "Number of sampled census blockgroups"} -->
<!-- SELECT blockgroup_id, count(*) as parcel_count  from sandbox.parcels_in_blockgroup_sample -->
<!-- GROUP BY blockgroup_id -->
<!-- ORDER BY parcel_count -->
<!-- ``` -->
Finally, we sample from the parcels within the block groups to get the final household survey sample.

``` sql
CREATE TABLE sandbox.hh_survey_sample AS
(SELECT * FROM sandbox.parcels_in_blockgroup_sample TABLESAMPLE BERNOULLI (10) REPEATABLE (1234))
```

``` sql
SELECT count(*) FROM sandbox.hh_survey_sample
```

| count |
|:------|
| 1822  |
