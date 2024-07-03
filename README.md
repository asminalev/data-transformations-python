# Data transformations with Python

This is a collection of _Python_ jobs that are supposed to extract, transform and load data.
These jobs are using _PySpark_ to process larger volumes of data and are supposed to run on a _Spark_ cluster (via `spark-submit`).

1. **Get a working environment**  
   Either local ([local](#local-setup), or using [gitpod](#gitpod-setup))

### Gitpod setup

You can setup the environment using

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/data-transformations-python/dataengineer-transformations-python)

There's an initialize script setup that takes around 3 minutes to complete. Once you use paste this repository link in new Workspace, please wait until the packages are installed. After everything is setup, select Poetry's environment by clicking on thumbs up icon and navigate to Testing tab and hit refresh icon to discover tests.

Note that you can use gitpod's web interface or setup [ssh to Gitpod](https://www.gitpod.io/docs/references/ides-and-editors/vscode#connecting-to-vs-code-desktop) so that you can use VS Code from local to remote to Gitpod

### Verify setup

> All of the following commands should be running successfully

#### Run unit tests

```bash
poetry run pytest tests/unit
```

#### Run integration tests

```bash
poetry run pytest tests/integration
```

#### Run style checks

```bash
poetry run mypy --ignore-missing-imports --disallow-untyped-calls --disallow-untyped-defs --disallow-incomplete-defs \
            data_transformations tests

poetry run pylint data_transformations tests
```

### Anything else?

All commands are passing?  
You are good to go!

You can customize your environment (having the test in vscode directly for example): feel free to spend the time making this comfortable for you. 

## Jobs

There are two exercises in this repo: Word Count, and Citibike.

Currently, these exist as skeletons, and have some **initial test cases** which are defined but some are skipped.

The following section provides context over them.

### Code walk

```

/
├─ /data_transformations # Contains the main python library
│ # with the code to the transformations
│
├─ /jobs # Contains the entry points to the jobs
│ # performs argument parsing, and are
│ # passed to `spark-submit`
│
├─ /resources # Contains the raw datasets for the jobs
│
├─ /tests
│ ├─ /units # contains basic unit tests for the code
│ └─ /integration # contains integrations tests for the jobs
│ # and the setup
│
├─ .gitignore
├─ .gitpod\* # required for the gitpod setup
├─ .pylintrc # configuration for pylint
├─ LICENCE
├─ poetry.lock
├─ pyproject.toml
└─ README.md # The current file

```

### Word Count

A NLP model is dependent on a specific input file. This job is supposed to preprocess a given text file to produce this
input file for the NLP model (feature engineering). This job will count the occurrences of a word within the given text
file (corpus).

There is a dump of the datalake for this under `resources/word_count/words.txt` with a text file.

```mermaid
---
title: Citibike Pipeline
---
flowchart LR
  Raw["fa:fa-file words.txt"] -->  J1{{word_count.py}} --> Bronze["fa:fa-file-csv word_count.csv"]
```

#### Input

Simple `*.txt` file containing text.

#### Output

A single `*.csv` file containing data similar to:

```csv
"word","count"
"a","3"
"an","5"
...
```

#### Run the job

```bash
poetry build && poetry run spark-submit \
    --master local \
    --py-files dist/data_transformations-*.whl \
    jobs/word_count.py \
    <INPUT_FILE_PATH> \
    <OUTPUT_PATH>
```

### Citibike

**_This problem uses data made publicly available by [Citibike](https://citibikenyc.com/), a New York based bike share company._**

For analytics purposes, the BI department of a hypothetical bike share company would like to present dashboards, displaying the
distance each bike was driven. There is a `*.csv` file that contains historical data of previous bike rides. This input
file needs to be processed in multiple steps. There is a pipeline running these jobs.

```mermaid
---
title: Citibike Pipeline
---
flowchart TD
  Raw["fa:fa-file-csv citibike.csv"] -->  J1{{citibike_ingest.py}} --> Bronze["fa:fa-table-columns citibike.parquet"] --> J2{{citibike_distance_calculation.py}} --> Silver["fa:fa-table-columns citibike_distance.parquet"]
```

There is a dump of the datalake for this under `resources/citibike/citibike.csv` with historical data.

#### Ingest

Reads a `*.csv` file and transforms it to parquet format. The column names will be sanitized (whitespaces replaced).

##### Input

Historical bike ride `*.csv` file:

```csv
"tripduration","starttime","stoptime","start station id","start station name","start station latitude",...
364,"2017-07-01 00:00:00","2017-07-01 00:06:05",539,"Metropolitan Ave & Bedford Ave",40.71534825,...
...
```

##### Output

`*.parquet` files containing the same content

```csv
"tripduration","starttime","stoptime","start_station_id","start_station_name","start_station_latitude",...
364,"2017-07-01 00:00:00","2017-07-01 00:06:05",539,"Metropolitan Ave & Bedford Ave",40.71534825,...
...
```

##### Run the job

```bash
poetry build && poetry run spark-submit \
    --master local \
    --py-files dist/data_transformations-*.whl \
    jobs/citibike_ingest.py \
    <INPUT_FILE_PATH> \
    <OUTPUT_PATH>
```

#### Distance calculation

This job takes bike trip information and adds the "as the crow flies" distance traveled for each trip.
It reads the previously ingested data parquet files.

Hint:

- For distance calculation, consider using [**Haversine formula**](https://www.movable-type.co.uk/scripts/latlong.html) as an option.

##### Input

Historical bike ride `*.parquet` files

```csv
"tripduration",...
364,...
...
```

##### Outputs

`*.parquet` files containing historical data with distance column containing the calculated distance.

```csv
"tripduration",...,"distance"
364,...,1.34
...
```

##### Run the job

```bash
poetry build && poetry run spark-submit \
    --master local \
    --py-files dist/data_transformations-*.whl \
    jobs/citibike_distance_calculation.py \
    <INPUT_PATH> \
    <OUTPUT_PATH>
```
---

## Reading List

If you are unfamiliar with some of the tools used here, get started from:

- **pytest**: [official](https://docs.pytest.org/en/8.2.x/getting-started.html#get-started)
- **pyspark**: [official](https://spark.apache.org/docs/latest/api/python/index.html) and especially the [DataFrame quickstart](https://spark.apache.org/docs/latest/api/python/getting_started/quickstart_df.html)
