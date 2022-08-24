# Meltano Example Projects: Extract & Load (EL) (Jaffle Shop) Sandbox
This project extends the ```jaffle shop``` sandbox project created by [DbtLabs](https://github.com/dbt-labs/jaffle_shop) for the data built tool ```dbt```. This meltano project sources one CSV file from AWS S3 and puts them into one table inside a Postgres database.

![EL Meltano Run](Meltano_EL.gif)

## What is this repo?
What this repo is:

A self-contained sandbox meltano project. Useful for testing out scripts, yaml configurations and understanding some of the core meltano concepts.

What this repo is not:
This repo is not a tutorial or a walk-through. It contains some bad practices. To make it self-contained, it contains a AWS S3 mock, as well
as a dockerized Postgres database. 

We're focusing on simplicity here!

## What's in this repo?
This repo contains an ```AWS S3``` mock with one CSV file inside. The raw customer data.

The meltano project extracts this CSV using the tap-s3-csv ```extractor```, and loads them into the ```PostgreSQL``` database using
the loader ```target-postgres```.

![EL Meltano Diagram](el_meltano_diagram.jpg)

## How to run this project?
Using this repository is really easy as it all runs inside docker via batect. 

Run  ```./batect --list-tasks ``` to see the list of commands.

1. Launch the mock endpoints in a separate terminal window ```./batect launch_mock````
2. With batect: Launch meltano with batect via ```batect melt````
3. OR install meltano with ```pip install meltano```

Then run ```meltano install``` to install the two plugins, the S3 extractor and the PostgreSQL loader as specified
in the [meltano.yml](new_project/meltano.yml):

```yaml
...
plugins:
  extractors:
  - name: tap-s3-csv
    variant: transferwise
    pip_url: pipelinewise-tap-s3-csv
    config:
      bucket: test
      tables:
        - search_prefix: ""
          search_pattern: "raw_customers.csv"
          table_name: "raw_customers"
          key_properties: ["id"]
          delimiter: ","
      start_date: 2000-01-01T00:00:00Z
      aws_endpoint_url: http://host.docker.internal:5005
      aws_access_key_id: s
      aws_secret_access_key: s
      aws_default_region: us-east-1

  loaders:
  - name: target-postgres
    variant: transferwise
    pip_url: pipelinewise-target-postgres
    config:
      host: host.docker.internal
      port: 5432
      user: admin
      password: password
      dbname: demo
```

Finally, run ```meltano run tap-s3-csv target-postgres``` to execute the extraction and loading. Check inside the local database afterwards to see that your data has arrived.
