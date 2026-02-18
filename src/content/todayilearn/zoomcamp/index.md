---
title: "Zoomcamp"
description: "Data Engineering Zoomcamp 2026"
date: "January 19 2026"
---

# Homework 1: Docker, SQL and Terraform for Data Engineering Zoomcamp 2026

## Question 1.

Understanding Docker images

Run docker with the python:3.13 image. Use an entrypoint bash to interact with the container.

What's the version of pip in the image?
1. 25.3
2. 24.3.1
3. 24.2.1
4. 23.3.1

### Solution

You can run `python:3.13` with `docker` and `interactive terminal` flag (`-it`)

```{bash}
docker run -it --rm -v $(pwd):/app python:3.13 pip -V
```

Result:
pip 25.3 from /usr/local/lib/python3.13/site-packages/pip (python 3.13)

so the answer is `25.3`

### Answer
>1. 25.3


## Question 2

Given the `docker-compose.yaml`, what hostname and port should pgAdmin use to connect to the Postgres database? (1 point)

1. postgres:5433
2. localhost:5432
3. db:5433
4. postgres:5432
5. db:5432

### Solution

Take a look at the `docker-compose.yaml` file:

```bash
...
image: postgres:18
    environment:
      POSTGRES_USER: "root"
      POSTGRES_PASSWORD: "root"
      POSTGRES_DB: "ny_taxi"
    volumes:
      - "ny_taxi_postgres_data:/var/lib/postgresql"
    ports:
      - "5432:5432"
...
```

You can see that the hostname is `localhost` and the port is `5432`.
So, the answer is `2. localhost:5432`.

### Answer
> 2. localhost:5432

## Question 3.

For the trips in November 2025, how many trips had a trip_distance of less than or equal to 1 mile? (1 point)

1. 7,853
2. 8,007
3. 8,254
4. 8,421

### Solution
First, open Jupyter Notebook or Google Colab to run the code.

Use the following code to download the Parquet dataset file:

```python
!wget https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2025-11.parquet
```

Then, read the `.parquet` file and display it as a DataFrame:

```python
import pandas as pd
df = pd.read_parquet("green_tripdata_2025-11.parquet")
df
```

This is a preview of the data, it consists of 21 columns.

| VendorID | lpep_pickup_datetime | lpep_dropoff_datetime | store_and_fwd_flag | RatecodeID | PULocationID | DOLocationID | passenger_count | trip_distance | fare_amount | extra | mta_tax | tip_amount | tolls_amount | ehail_fee | improvement_surcharge | total_amount | payment_type | trip_type | congestion_surcharge | cbd_congestion_fee |
|---------:|----------------------|-----------------------|--------------------|-----------:|-------------:|-------------:|----------------:|--------------:|------------:|------:|--------:|-----------:|--------------:|----------:|----------------------:|-------------:|-------------:|----------:|----------------------:|-------------------:|
| 2 | 2025-11-01 00:34:48 | 2025-11-01 00:41:39 | N | 1.0 | 74 | 42 | 1.0 | 0.74 | 7.20 | 1.00 | 0.5 | 1.94 | 0.0 | NaN | 1.0 | 11.64 | 1.0 | 1.0 | 0.00 | 0.00 |
| 2 | 2025-11-01 00:18:52 | 2025-11-01 00:24:27 | N | 1.0 | 74 | 42 | 2.0 | 0.95 | 7.20 | 1.00 | 0.5 | 0.00 | 0.0 | NaN | 1.0 | 9.70 | 2.0 | 1.0 | 0.00 | 0.00 |

Before we work with the data, it’s important to convert the column we’re using to the correct data type. In this case, we’ll convert `lpep_pickup_datetime` to a datetime column.

```python
df["lpep_pickup_datetime"] = pd.to_datetime(df["lpep_pickup_datetime"])
```

To filter the date range, we can use the `between()` method with two parameters: the start date and the end date (see [pandas.Series.between](https://pandas.pydata.org/docs/reference/api/pandas.Series.between.html)). Save the filtered result into a new variable called `df_result`.

```python
df_result = df[df["lpep_pickup_datetime"].between("2025-11-01","2025-12-01")]
```

Because the question asks for trips with a `trip_distance` of less than or equal to 1 mile, we can filter like this:

```python
df_result[df_result["trip_distance"] <= 1].reset_index()
```
This filters rows where `trip_distance` is less than or equal to 1.

We can count the total number of matching trips by using `len()`:

```python
len(df_result[df_result["trip_distance"] <= 1].reset_index())
```

### Answer
> 8007 Trips

## Question 4.
Which pickup day had the longest total trip distance? Only consider trips with a trip_distance less than 100 miles. (1 point)

1. 2025-11-14
2. 2025-11-20
3. 2025-11-23
4. 2025-11-25

### Solution
First, open Jupyter Notebook or Google Colab to run the code.

Use the following code to download the Parquet dataset file:

```python
!wget https://d37ci6vzurychx.cloudfront.net/trip-data/green_tripdata_2025-11.parquet
```

Next, read the `.parquet` file and display it as a DataFrame:

```python
import pandas as pd
df = pd.read_parquet("green_tripdata_2025-11.parquet")
```

There are several ways to complete this task, but I prefer to extract the month, day, and year from the pickup datetime column and store them in separate variables.

```python
df["month"] = df["lpep_pickup_datetime"].dt.month
df["day"] = df["lpep_pickup_datetime"].dt.day
df["year"] = df["lpep_pickup_datetime"].dt.year
```
Let's aggregate the data by month and day, summing the trip_distance column, so we get the total trip distance for each date.

```python
df[df["trip_distance"] < 100].groupby(["month", "day"]).agg(total=("trip_distance", "sum")).sort_values("total", ascending=False)
```

Here is the resulting table of total trip distances (less than 100 miles), grouped by month and day:

| Month | Day | Total Trip Distance |
|-------|-----|--------------------|
| 11    | 20  | 6377.03            |
|       | 19  | 6031.56            |
|       | 18  | 5976.12            |
|       | 6   | 5973.34            |
|       | 25  | 5954.80            |
|       | 5   | 5841.01            |

The day with the longest total trip distance is **20 November 2025**.

### Answer
> 2. 2025-11-20

## Question 5.

Which was the pickup zone with the largest total_amount (sum of all trips) on November 18th, 2025? (1 point)

1. East Harlem North
2. East Harlem South
3. Morningside Heights
4. Forest Hills

### Solution

We need one more file for this task. Please download it using the code below:

```
!wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv
```

Example of the data:

| LocationID | Borough | Zone                      | service_zone |
|------------|---------|---------------------------|--------------|
| 1          | EWR     | Newark Airport            | EWR          |
| 2          | Queens  | Jamaica Bay               | Boro Zone    |
| 3          | Bronx   | Allerton/Pelham Gardens   | Boro Zone    |

Let’s import both files:

```python
import pandas as pd
df = pd.read_parquet("./green_tripdata_2025-11.parquet")
df_zone = pd.read_csv("./taxi_zone_lookup.csv")
df_zone = df_zone.dropna()
```

Next, merge the two tables using `PULocationID` and `LocationID` as the join keys:

```python
df_joined = pd.merge(df,df_zone,left_on="PULocationID",right_on="LocationID")
df_joined
```

This will create one combined table (the main `df` and `df_zone`) and store it in the `df_joined` variable.

Now, let’s filter trips for **November 18th, 2025**, then group by zone and aggregate the sum of `total_amount`:

```python
df_joined[
    (df_joined["day"] == 18) & (df_joined["months"] == 11) & (df_joined["year"] == 2025)
].groupby("Zone").agg(total=("total_amount", "sum")).sort_values("total", ascending=False)
```

Lastly, we can sort from the highest to the lowest `total_amount`.

| Zone                     | total    |
|--------------------------|----------|
| East Harlem North        | 9281.92  |
| East Harlem South        | 6696.13  |
| Central Park             | 2378.79  |
| Washington Heights South | 2139.05  |

We can see that East Harlem North is the pickup zone that have the highest total amount that user paying.

### Answer
> 1. East Harlem North

## Question 7.
Which of the following sequences describes the Terraform workflow for:
- Downloading plugins and setting up backend
- Generating and executing changes
- Removing all resources? (1 point)

1. terraform import, terraform apply -y, terraform destroy
2. teraform init, terraform plan -auto-apply, terraform rm
3. terraform init, terraform run -auto-approve, terraform destroy
4. terraform init, terraform apply -auto-approve, terraform destroy
5. terraform import, terraform apply -y, terraform rm


### Solution
Check the Terraform module in this GitHub repo:
https://github.com/DataTalksClub/data-engineering-zoomcamp/blob/main/01-docker-terraform/terraform/1_terraform_overview.md#execution-steps

From this guideline, we can see that:
1. `terraform init` is used to initialize the backend, install plugins, and check the configuration.
2. `terraform apply` is used to apply the plan in the cloud, with the `-auto-approve` flag.
3. Lastly, we can use `terraform destroy` to remove all resources and stacks from the cloud.

So the answer is **4**.

### Answer

> 4. terraform init, terraform apply -auto-approve, terraform destroy