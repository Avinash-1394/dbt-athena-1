<p align="center">
    <img src="https://raw.githubusercontent.com/dbt-athena/dbt-athena/main/static/images/dbt-athena-long.png" />
    <a href="https://pypi.org/project/dbt-athena-community/"><img src="https://badge.fury.io/py/dbt-athena-community.svg" /></a>
    <a href="https://pycqa.github.io/isort/"><img src="https://img.shields.io/badge/%20imports-isort-%231674b1?style=flat&labelColor=ef8336" /></a>
    <a href="https://github.com/psf/black"><img src="https://img.shields.io/badge/code%20style-black-000000.svg" /></a>
    <a href="https://pepy.tech/project/dbt-athena-community"><img src="https://pepy.tech/badge/dbt-athena-community/month" /></a>
</p>
<p align="center">
    <img src="https://raw.githubusercontent.com/dbt-athena/dbt-athena/main/static/images/dbt-athena-long.png" />
    <a href="https://pypi.org/project/dbt-athena-community/"><img src="https://badge.fury.io/py/dbt-athena-community.svg" /></a>
    <a href="https://pycqa.github.io/isort/"><img src="https://img.shields.io/badge/%20imports-isort-%231674b1?style=flat&labelColor=ef8336" /></a>
    <a href="https://github.com/psf/black"><img src="https://img.shields.io/badge/code%20style-black-000000.svg" /></a>
    <a href="https://pepy.tech/project/dbt-athena-community"><img src="https://pepy.tech/badge/dbt-athena-community/month" /></a>
</p>

## Features

* Supports dbt version `1.4.*`
* Supports [Seeds][seeds]
* Correctly detects views and their columns
* Supports [table materialization][table]
  * [Iceberg tables][athena-iceberg] is supported **only with Athena Engine v3** and **a unique table location**
  (see table location section below)
  * Hive tables is supported by both Athena engines.
* Supports [incremental models][incremental]
  * On iceberg tables :
    * Support the use of `unique_key` only with the `merge` strategy
    * Support the `append` strategy
  * On Hive tables :
    * Support two incremental update strategies: `insert_overwrite` and `append`
    * Does **not** support the use of `unique_key`
* Supports [snapshots][snapshots]
* Supports [Python models][python-models]
* Does not support [persist docs][persist-docs] for views

[seeds]: https://docs.getdbt.com/docs/building-a-dbt-project/seeds
[incremental]: https://docs.getdbt.com/docs/build/incremental-models
[table]: https://docs.getdbt.com/docs/build/materializations#table
[python-models]: https://docs.getdbt.com/docs/build/python-models#configuring-python-models
[athena-iceberg]: https://docs.aws.amazon.com/athena/latest/ug/querying-iceberg.html
[snapshots]: https://docs.getdbt.com/docs/build/snapshots
[persist-docs]: https://docs.getdbt.com/reference/resource-configs/persist_docs


## Quick Start

### Installation

* `pip install dbt-athena-community`
* Or `pip install git+https://github.com/dbt-athena/dbt-athena.git`

### Prerequisites

To start, you will need an S3 bucket, for instance `my-bucket` and an Athena database:

```sql
CREATE DATABASE IF NOT EXISTS analytics_dev
COMMENT 'Analytics models generated by dbt (development)'
LOCATION 's3://my-bucket/'
WITH DBPROPERTIES ('creator'='Foo Bar', 'email'='foo@bar.com');
```

Notes:
- Take note of your AWS region code (e.g. `us-west-2` or `eu-west-2`, etc.).
- You can also use [AWS Glue](https://docs.aws.amazon.com/athena/latest/ug/glue-athena.html) to create and manage Athena databases.

### Credentials

Credentials can be passed directly to the adapter, or they can be [determined automatically](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/credentials.html) based on `aws cli`/`boto3` conventions.
You can either:
- configure `aws_access_key_id` and `aws_secret_access_key`
- configure `aws_profile_name` to match a profile defined in your AWS credentials file
Checkout DBT profile configuration below for details.

### Configuring your profile

A dbt profile can be configured to run against AWS Athena using the following configuration:

| Option                | Description                                                                    | Required? | Example                                    |
|-----------------------|--------------------------------------------------------------------------------|-----------|--------------------------------------------|
| s3_staging_dir        | S3 location to store Athena query results and metadata                         | Required  | `s3://bucket/dbt/`                         |
| s3_data_dir           | Prefix for storing tables, if different from the connection's `s3_staging_dir` | Optional  | `s3://bucket2/dbt/`                        |
| s3_data_naming        | How to generate table paths in `s3_data_dir`                                   | Optional  | `schema_table_unique`                      |
| region_name           | AWS region of your Athena instance                                             | Required  | `eu-west-1`                                |
| schema                | Specify the schema (Athena database) to build models into (lowercase **only**) | Required  | `dbt`                                      |
| database              | Specify the database (Data catalog) to build models into (lowercase **only**)  | Required  | `awsdatacatalog`                           |
| poll_interval         | Interval in seconds to use for polling the status of query results in Athena   | Optional  | `5`                                        |
| aws_access_key_id     | Access key ID of the user performing requests.                                 | Optional  | `AKIAIOSFODNN7EXAMPLE`                     |
| aws_secret_access_key | Secret access key of the user performing requests                              | Optional  | `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY` |
| aws_profile_name      | Profile to use from your AWS shared credentials file.                          | Optional  | `my-profile`                               |
| work_group            | Identifier of Athena workgroup                                                 | Optional  | `my-custom-workgroup`                      |
| num_retries           | Number of times to retry a failing query                                       | Optional  | `3`                                        |
| lf_tags               | Default lf tags to apply to any database created by dbt                        | Optional  | `{"origin": "dbt", "team": "analytics"}`   |
| spark_work_group      | Identifier of athena spark workgroup                                           | Optional  | `my-spark-workgroup`                       |

**Example profiles.yml entry:**
```yaml
athena:
  target: dev
  outputs:
    dev:
      type: athena
      s3_staging_dir: s3://athena-query-results/dbt/
      s3_data_dir: s3://your_s3_bucket/dbt/
      s3_data_naming: schema_table
      region_name: eu-west-1
      schema: dbt
      database: awsdatacatalog
      aws_profile_name: my-profile
      work_group: my-workgroup
      spark_work_group: my-spark-workgroup
      lf_tags:
        origin: dbt
        team: analytics
```

_Additional information_
* `threads` is supported
* `database` and `catalog` can be used interchangeably


## Models

### Table Configuration

* `external_location` (`default=none`)
  * If set, the full S3 path in which the table will be saved. (Does not work with Iceberg table).
* `partitioned_by` (`default=none`)
  * An array list of columns by which the table will be partitioned
  * Limited to creation of 100 partitions (_currently_)
* `bucketed_by` (`default=none`)
  * An array list of columns to bucket data, ignored if using Iceberg
* `bucket_count` (`default=none`)
  * The number of buckets for bucketing your data, ignored if using Iceberg
* `table_type` (`default='hive'`)
  * The type of table
  * Supports `hive` or `iceberg`
* `format` (`default='parquet'`)
  * The data format for the table
  * Supports `ORC`, `PARQUET`, `AVRO`, `JSON`, `TEXTFILE`
* `write_compression` (`default=none`)
  * The compression type to use for any storage format that allows compression to be specified. To see which options are available, check out [CREATE TABLE AS][create-table-as]
* `field_delimiter` (`default=none`)
  * Custom field delimiter, for when format is set to `TEXTFILE`
* `table_properties`: table properties to add to the table, valid for Iceberg only
* `lf_tags` (`default=none`)
  * lf tags to associate with the table
  * format: `{"tag1": "value1", "tag2": "value2"}`
* `lf_tags_columns` (`default=none`)
  * lf tags to associate with the table columns
  * format: `{"tag1": {"value1": ["column1": "column2"]}}`

### Table location

The location in which a table is saved is determined by:

1. If `external_location` is defined, that value is used.
2. If `s3_data_dir` is defined, the path is determined by that and `s3_data_naming`
3. If `s3_data_dir` is not defined data is stored under `s3_staging_dir/tables/`

Here all the options available for `s3_data_naming`:
* `uuid`: `{s3_data_dir}/{uuid4()}/`
* `table_table`: `{s3_data_dir}/{table}/`
* `table_unique`: `{s3_data_dir}/{table}/{uuid4()}/`
* `schema_table`: `{s3_data_dir}/{schema}/{table}/`
* `s3_data_naming=schema_table_unique`: `{s3_data_dir}/{schema}/{table}/{uuid4()}/`

It's possible to set the `s3_data_naming` globally in the target profile, or overwrite the value in the table config,
or setting up the value for groups of model in dbt_project.yml.

> Note: when using a work group with a default output location configured, `s3_data_naming` and any configured buckets are ignored and the location configured in the work group is used.

### Incremental models

Support for [incremental models](https://docs.getdbt.com/docs/build/incremental-models).

These strategies are supported:

* `insert_overwrite` (default): The insert overwrite strategy deletes the overlapping partitions from the destination
table, and then inserts the new records from the source. This strategy depends on the `partitioned_by` keyword! If no
partitions are defined, dbt will fall back to the `append` strategy.
* `append`: Insert new records without updating, deleting or overwriting any existing data. There might be duplicate
data (e.g. great for log or historical data).
* `merge`: Conditionally updates, deletes, or inserts rows into an Iceberg table. Used in combination with `unique_key`.
Only available when using Iceberg.

### On schema change

`on_schema_change` is an option to reflect changes of schema in incremental models.
The following options are supported:
* `ignore` (default)
* `fail`
* `append_new_columns`
* `sync_all_columns`

In detail, please refer to [dbt docs](https://docs.getdbt.com/docs/build/incremental-models#what-if-the-columns-of-my-incremental-model-change).

### Iceberg

The adapter supports table materialization for Iceberg.

To get started just add this as your model:
```
{{ config(
    materialized='table',
    table_type='iceberg',
    format='parquet',
    partitioned_by=['bucket(user_id, 5)'],
    table_properties={
    	'optimize_rewrite_delete_file_threshold': '2'
    	}
) }}

SELECT
	'A' AS user_id,
	'pi' AS name,
	'active' AS status,
	17.89 AS cost,
	1 AS quantity,
	100000000 AS quantity_big,
	current_date AS my_date
```

Iceberg supports bucketing as hidden partitions, therefore use the `partitioned_by` config to add specific bucketing conditions.

Iceberg supports several table formats for data : `PARQUET`, `AVRO` and `ORC`.

It is possible to use iceberg in an incremental fashion, specifically 2 strategies are supported:
* `append`: new records are appended to the table, this can lead to duplicates
* `merge`: must be used in combination with `unique_key` and it's only available with Engine version 3.
   It performs an upsert, new record are added, and record already existing are updated. If
   `delete_condition` is provided in the model config, it can also delete records based on the
   provided condition (SQL condition). You can use any column of the incremental table (`src`) or
   the final table (`target`). You must prefix the column by the name of the table to prevent
   `Column is ambiguous` error.

```sql
{{ config(
    materialized='incremental',
    table_type='iceberg',
    incremental_strategy='merge',
    unique_key='user_id',
    delete_condition="src.status != 'active' and target.my_date < now() - interval '2' year"
    format='parquet',
) }}

SELECT
	'A' AS user_id,
	'pi' AS name,
	'active' AS status,
	17.89 AS cost,
	1 AS quantity,
	100000000 AS quantity_big,
	current_date AS my_date
```

### High available table materialization
The current implementation of the table materialization can lead to downtime, as target table is dropped and re-created.
To have the less destructive behavior it's possible to use `table='table_hive_ha'` materialization.
**table_hive_ha** leverage the table versions feature of glue catalog, creating a tmp table and swapping
the target table to the location of the tmp table.
This materialization is only available for `table_type=hive` and requires using unique locations.

```
{{ config(
    materialized='table_hive_ha',
    format='parquet',
    partition_by=['status'],
    s3_data_naming='table_unique'
) }}

select
  'a' as user_id,
  'pi' as user_name,
  'active' as status
union all
select
  'b' as user_id,
  'sh' as user_name,
  'disabled' as status
```

By default, the materialization keeps the last 4 table versions, you can change it that setting `versions_to_keep`.

#### Known issues
* When swapping from a table with partitions to a table without (and the other way around), there could be a little downtime.
  In case high performances are needed consider bucketing instead of partitions
* By default, Glue "duplicate" the versions internally, so the last 2 versions of a table point to the same location
* It's recommended to have versions_to_keep>= 4, as this will avoid to have the older location removed
* The macro athena__end_of_time needs to be overwritten by the user if using Athena v3 since it requires a precision parameter for timestamps


## Snapshots

The adapter supports snapshot materialization. It supports both timestamp and check strategy. To create a snapshot create a snapshot file in the snapshots directory. If directory does not exist create one.

### Timestamp strategy

To use the timestamp strategy refer to the [dbt docs](https://docs.getdbt.com/docs/build/snapshots#timestamp-strategy-recommended)

### Check strategy

To use the check strategy refer to the [dbt docs](https://docs.getdbt.com/docs/build/snapshots#check-strategy)

### Hard-deletes

The materialization also supports invalidating hard deletes. Check the [docs](https://docs.getdbt.com/docs/build/snapshots#hard-deletes-opt-in) to understand usage.

### Working example

seed file - employent_indicators_november_2022_csv_tables.csv
```
Series_reference,Period,Data_value,Suppressed
MEIM.S1WA,1999.04,80267,
MEIM.S1WA,1999.05,70803,
MEIM.S1WA,1999.06,65792,
MEIM.S1WA,1999.07,66194,
MEIM.S1WA,1999.08,67259,
MEIM.S1WA,1999.09,69691,
MEIM.S1WA,1999.1,72475,
MEIM.S1WA,1999.11,79263,
MEIM.S1WA,1999.12,86540,
MEIM.S1WA,2000.01,82552,
MEIM.S1WA,2000.02,81709,
MEIM.S1WA,2000.03,84126,
MEIM.S1WA,2000.04,77089,
MEIM.S1WA,2000.05,73811,
MEIM.S1WA,2000.06,70070,
MEIM.S1WA,2000.07,69873,
MEIM.S1WA,2000.08,71468,
MEIM.S1WA,2000.09,72462,
MEIM.S1WA,2000.1,74897,
```

model.sql
```
{{ config(
    materialized='table'
) }}

SELECT
    ROW_NUMBER() OVER () AS id
    , *
    , cast(from_unixtime(to_unixtime(now())) as timestamp(6)) AS refresh_timestamp
FROM {{ ref('employment_indicators_november_2022_csv_tables') }}
```

timestamp strategy - model_snapshot_1

```
{% snapshot model_snapshot_1 %}

{{
    config(
      strategy='timestamp',
      updated_at='refresh_timestamp',
      unique_key='id'
    )
}}

SELECT *

from {{ ref('model') }}

{% endsnapshot %}
```

invalidate hard deletes - model_snapshot_2
```
{% snapshot model_snapshot_2 %}

{{
    config
    (
        unique_key='id',
        strategy='timestamp',
        updated_at='refresh_timestamp',
        invalidate_hard_deletes=True,
    )
}}
SELECT * from {{ ref('model') }}

{% endsnapshot %}
```

check strategy - model_snapshot_3
```
{% snapshot model_snapshot_3 %}

{{
    config
    (
        unique_key='id',
        strategy='check',
        check_cols=['series_reference','data_value']
    )
}}
SELECT * from {{ ref('model') }}

{% endsnapshot %}
```

### Known issues

* Incremental Iceberg models - Sync all columns on schema change can't remove columns used as partitioning.
The only way, from a dbt perspective, is to do a full-refresh of the incremental model.

* Tables, schemas and database should only be lowercase

* In order to avoid potential conflicts, make sure [`dbt-athena-adapter`](https://github.com/Tomme/dbt-athena) is not installed in the target environment.
  See https://github.com/dbt-athena/dbt-athena/issues/103 for more details.

* Snapshot does not support dropping columns from the source table. If you drop a column make sure to drop the column from the snapshot as well. Another workaround is to NULL the column in the snapshot definition to preserve history

### Python Models

The adapter supports python models using [`spark`](https://docs.aws.amazon.com/athena/latest/ug/notebooks-spark.html).

#### Prerequisites

* A spark enabled work group created in athena
* Spark execution role granted access to Athena, Glue and S3
* The spark work group is added to the ~/.dbt/profiles.yml file and the profile is referenced in dbt_project.yml

#### Example model

```python
import pandas as pd


def model(dbt, session):
    dbt.config(materialized="table")

    model_df = pd.DataFrame({"A": [1, 2, 3, 4]})

    return model_df
```

#### Known issues in python models

* Incremental models do not fully utilize spark capabilities. They depend on existing sql based logic.
* Snapshots materializations are not supported.


## Contributing

See [CONTRIBUTING](CONTRIBUTING.md) for more information on how to contribute to this project.


## Contributors ✨

Thanks goes to these wonderful people ([emoji key](https://allcontributors.org/docs/en/emoji-key)):

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->
<table>
  <tbody>
    <tr>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/nicor88"><img src="https://avatars.githubusercontent.com/u/6278547?v=4?s=100" width="100px;" alt="nicor88"/><br /><sub><b>nicor88</b></sub></a><br /><a href="https://github.com/dbt-athena/dbt-athena/commits?author=nicor88" title="Code">💻</a> <a href="#maintenance-nicor88" title="Maintenance">🚧</a> <a href="https://github.com/dbt-athena/dbt-athena/issues?q=author%3Anicor88" title="Bug reports">🐛</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://jessedobbelae.re"><img src="https://avatars.githubusercontent.com/u/1352979?v=4?s=100" width="100px;" alt="Jesse Dobbelaere"/><br /><sub><b>Jesse Dobbelaere</b></sub></a><br /><a href="https://github.com/dbt-athena/dbt-athena/issues?q=author%3Ajessedobbelaere" title="Bug reports">🐛</a> <a href="#maintenance-jessedobbelaere" title="Maintenance">🚧</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/lemiffe"><img src="https://avatars.githubusercontent.com/u/7487772?v=4?s=100" width="100px;" alt="Lemiffe"/><br /><sub><b>Lemiffe</b></sub></a><br /><a href="#design-lemiffe" title="Design">🎨</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/Jrmyy"><img src="https://avatars.githubusercontent.com/u/9251353?v=4?s=100" width="100px;" alt="Jérémy Guiselin"/><br /><sub><b>Jérémy Guiselin</b></sub></a><br /><a href="#maintenance-Jrmyy" title="Maintenance">🚧</a> <a href="https://github.com/dbt-athena/dbt-athena/commits?author=Jrmyy" title="Code">💻</a> <a href="https://github.com/dbt-athena/dbt-athena/issues?q=author%3AJrmyy" title="Bug reports">🐛</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/Tomme"><img src="https://avatars.githubusercontent.com/u/932895?v=4?s=100" width="100px;" alt="Tom"/><br /><sub><b>Tom</b></sub></a><br /><a href="#maintenance-Tomme" title="Maintenance">🚧</a> <a href="https://github.com/dbt-athena/dbt-athena/commits?author=Tomme" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/mattiamatrix"><img src="https://avatars.githubusercontent.com/u/5013654?v=4?s=100" width="100px;" alt="Mattia"/><br /><sub><b>Mattia</b></sub></a><br /><a href="#maintenance-mattiamatrix" title="Maintenance">🚧</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/Gatsby-Lee"><img src="https://avatars.githubusercontent.com/u/22950880?v=4?s=100" width="100px;" alt="Gatsby Lee"/><br /><sub><b>Gatsby Lee</b></sub></a><br /><a href="https://github.com/dbt-athena/dbt-athena/issues?q=author%3AGatsby-Lee" title="Bug reports">🐛</a></td>
    </tr>
    <tr>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/BrechtDeVlieger"><img src="https://avatars.githubusercontent.com/u/12074972?v=4?s=100" width="100px;" alt="BrechtDeVlieger"/><br /><sub><b>BrechtDeVlieger</b></sub></a><br /><a href="https://github.com/dbt-athena/dbt-athena/issues?q=author%3ABrechtDeVlieger" title="Bug reports">🐛</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/aartaria"><img src="https://avatars.githubusercontent.com/u/10273710?v=4?s=100" width="100px;" alt="Andrea Artaria"/><br /><sub><b>Andrea Artaria</b></sub></a><br /><a href="https://github.com/dbt-athena/dbt-athena/issues?q=author%3Aaartaria" title="Bug reports">🐛</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/maiarareinaldo"><img src="https://avatars.githubusercontent.com/u/72740386?v=4?s=100" width="100px;" alt="Maiara Reinaldo"/><br /><sub><b>Maiara Reinaldo</b></sub></a><br /><a href="https://github.com/dbt-athena/dbt-athena/issues?q=author%3Amaiarareinaldo" title="Bug reports">🐛</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/henriblancke"><img src="https://avatars.githubusercontent.com/u/1708162?v=4?s=100" width="100px;" alt="Henri Blancke"/><br /><sub><b>Henri Blancke</b></sub></a><br /><a href="https://github.com/dbt-athena/dbt-athena/commits?author=henriblancke" title="Code">💻</a> <a href="https://github.com/dbt-athena/dbt-athena/issues?q=author%3Ahenriblancke" title="Bug reports">🐛</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/svdimchenko"><img src="https://avatars.githubusercontent.com/u/39801237?v=4?s=100" width="100px;" alt="Serhii Dimchenko"/><br /><sub><b>Serhii Dimchenko</b></sub></a><br /><a href="https://github.com/dbt-athena/dbt-athena/commits?author=svdimchenko" title="Code">💻</a> <a href="https://github.com/dbt-athena/dbt-athena/issues?q=author%3Asvdimchenko" title="Bug reports">🐛</a></td>
    </tr>
  </tbody>
</table>

<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->

<!-- ALL-CONTRIBUTORS-LIST:END -->

This project follows the [all-contributors](https://github.com/all-contributors/all-contributors) specification. Contributions of any kind welcome!
