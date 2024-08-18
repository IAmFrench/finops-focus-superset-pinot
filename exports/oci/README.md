# About the `./exports/oci` directory

This folder holds OCI FOCUS billing export

# How to download OCI FOCUS export?

Let's assume you have created a FOCUS export on a bucket named `finops-exports-1a2b3c4d`.

You can then use the following script [`./scripts/download_focus_export.sh`](./scripts/download_focus_export.sh) to download your exports:

```bash
../scripts/download_focus_export.sh \
    -p oci \
    -b finops-exports-1a2b3c4d \
    -o ./exports/oci/
```

# Example

```bash
du -a ./exports/oci

4840    ./exports/oci/0001000001507847-00001.csv.gz
1232    ./exports/oci/0001000001511187-00001.csv.gz
2992    ./exports/oci/0001000001512411-00001.csv.gz
4920    ./exports/oci/0001000001513618-00001.csv.gz
7456    ./exports/oci/0001000001514794-00001.csv.gz
1656    ./exports/oci/0001000001518066-00001.csv.gz
4096    ./exports/oci/0001000001519299-00001.csv.gz
4828    ./exports/oci/0001000001520484-00001.csv.gz
6056    ./exports/oci/0001000001521629-00001.csv.gz
1000    ./exports/oci/0001000001524995-00001.csv.gz
4132    ./exports/oci/0001000001526258-00001.csv.gz
5372    ./exports/oci/0001000001527428-00001.csv.gz
16      ./exports/oci/README.md
92076   ./exports/oci/
```

# Convert `.csv.gz` to `.snappy.parquet` files

On OCI FOCUS exports are available with a `.csv.gz` extension.

Therefore, to keep ingestion jobs standardized with other Cloud Service Providers, we will convert them to `.snappy.parquet` files.

OCI doesn't use the same DateTime format between `BillingPeriodEnd/BillingPeriodStart` and `ChargePeriodEnd/ChargePeriodStart`. As we are using Apache Pinot and Apache Superset, we need to convert them all DateTime columns to TIMESTAMP.

```bash
# Display "native" formats of DateTime columns
duckdb -c "select BillingPeriodEnd, BillingPeriodStart, ChargePeriodEnd, ChargePeriodStart from read_csv('0001000001527428-00001.csv.gz', types={'BillingPeriodEnd': 'VARCHAR', 'BillingPeriodStart': 'VARCHAR'}) limit 1"
┌──────────────────────────┬──────────────────────────┬───────────────────┬───────────────────┐
│     BillingPeriodEnd     │    BillingPeriodStart    │  ChargePeriodEnd  │ ChargePeriodStart │
│         varchar          │         varchar          │      varchar      │      varchar      │
├──────────────────────────┼──────────────────────────┼───────────────────┼───────────────────┤
│ 2024-08-31T23:59:59.999Z │ 2024-08-01T00:00:00.000Z │ 2024-08-16T01:00Z │ 2024-08-16T00:00Z │
└──────────────────────────┴──────────────────────────┴───────────────────┴───────────────────┘
```

> We use the types= to force duckdb cli to NOT autodetect the format of `BillingPeriodEnd` and `BillingPeriodStart` columns

```bash
# Conversion to timestamp for all DateTime columns
duckdb -c "select BillingPeriodEnd, BillingPeriodStart, strptime(ChargePeriodEnd, '%xT%H:%MZ'), strptime(ChargePeriodStart, '%xT%H:%MZ') from read_csv('0001000001527428-00001.csv.gz', types={'BillingPeriodEnd': 'TIMESTAMP', 'BillingPeriodStart': 'TIMESTAMP', 'ChargePeriodEnd': 'VARCHAR', 'ChargePeriodStart': 'VARCHAR'}) limit 1"
┌─────────────────────────┬─────────────────────┬────────────────────────────────────────┬──────────────────────────────────────────┐
│    BillingPeriodEnd     │ BillingPeriodStart  │ strptime(ChargePeriodEnd, '%xT%H:%MZ') │ strptime(ChargePeriodStart, '%xT%H:%MZ') │
│        timestamp        │      timestamp      │               timestamp                │                timestamp                 │
├─────────────────────────┼─────────────────────┼────────────────────────────────────────┼──────────────────────────────────────────┤
│ 2024-08-31 23:59:59.999 │ 2024-08-01 00:00:00 │ 2024-08-16 01:00:00                    │ 2024-08-16 00:00:00                      │
└─────────────────────────┴─────────────────────┴────────────────────────────────────────┴──────────────────────────────────────────┘
```

> We use the types= to force duckdb to use the `BillingPeriodEnd` and `BillingPeriodStart` DateTime format as timestamp and use the `strptime` function to parse `ChargePeriodEnd` and `ChargePeriodStart` columns to timestamp.


To do so, we will use the `duckdb` command line with the `REPLACE` SQL statement:

```bash
# Convert all .csv.gz to a single .snappy.parquet file
# Convert BillingPeriodEnd, BillingPeriodStart, ChargePeriodEnd, ChargePeriodStart to timestamp format
duckdb -markdown -c "COPY (select * replace (strptime(ChargePeriodEnd, '%xT%H:%MZ') as ChargePeriodEnd, strptime(ChargePeriodStart, '%xT%H:%MZ') as ChargePeriodStart) from read_csv('*.csv.gz', union_by_name = true, types={'BillingPeriodEnd': 'TIMESTAMP', 'BillingPeriodStart': 'TIMESTAMP', 'ChargePeriodEnd': 'VARCHAR', 'ChargePeriodStart': 'VARCHAR'})) TO 'oci_focus_export.snappy.parquet' (FORMAT 'parquet');"
```

And we can check the DateTime format for converted columns in the parquet file

```bash
> duckdb -c "select BillingPeriodStart, BillingPeriodEnd, ChargePeriodStart, ChargePeriodEnd from 'oci_focus_export.snappy.parquet' limit 1"
┌─────────────────────┬─────────────────────────┬─────────────────────┬─────────────────────┐
│ BillingPeriodStart  │    BillingPeriodEnd     │  ChargePeriodStart  │   ChargePeriodEnd   │
│      timestamp      │        timestamp        │      timestamp      │      timestamp      │
├─────────────────────┼─────────────────────────┼─────────────────────┼─────────────────────┤
│ 2024-07-01 00:00:00 │ 2024-07-31 23:59:59.999 │ 2024-07-30 00:00:00 │ 2024-07-30 01:00:00 │
└─────────────────────┴─────────────────────────┴─────────────────────┴─────────────────────┘
```

# Table Schema of OCI `.snappy.parquet` file

```bash
# Display the generated schema
duckdb -markdown -c "select * from parquet_schema('./oci_focus_export.snappy.parquet');"
```

|             file_name             |            name            |    type    | type_length | repetition_type | num_children |  converted_type  | scale | precision | field_id |                                            logical_type                                             |
|-----------------------------------|----------------------------|------------|-------------|-----------------|-------------:|------------------|-------|-----------|----------|-----------------------------------------------------------------------------------------------------|
| ./oci_focus_export.snappy.parquet | duckdb_schema              |            |             | REQUIRED        | 49           |                  |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | AvailabilityZone           | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | BilledCost                 | DOUBLE     |             | OPTIONAL        |              |                  |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | BillingAccountId           | INT64      |             | OPTIONAL        |              | INT_64           |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | BillingAccountName         | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | BillingCurrency            | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | BillingPeriodEnd           | INT64      |             | OPTIONAL        |              | TIMESTAMP_MICROS |       |           |          | TimestampType(isAdjustedToUTC=0, unit=TimeUnit(MILLIS=<null>, MICROS=MicroSeconds(), NANOS=<null>)) |
| ./oci_focus_export.snappy.parquet | BillingPeriodStart         | INT64      |             | OPTIONAL        |              | TIMESTAMP_MICROS |       |           |          | TimestampType(isAdjustedToUTC=0, unit=TimeUnit(MILLIS=<null>, MICROS=MicroSeconds(), NANOS=<null>)) |
| ./oci_focus_export.snappy.parquet | ChargeCategory             | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | ChargeDescription          | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | ChargeFrequency            | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | ChargePeriodEnd            | INT64      |             | OPTIONAL        |              | TIMESTAMP_MICROS |       |           |          | TimestampType(isAdjustedToUTC=0, unit=TimeUnit(MILLIS=<null>, MICROS=MicroSeconds(), NANOS=<null>)) |
| ./oci_focus_export.snappy.parquet | ChargePeriodStart          | INT64      |             | OPTIONAL        |              | TIMESTAMP_MICROS |       |           |          | TimestampType(isAdjustedToUTC=0, unit=TimeUnit(MILLIS=<null>, MICROS=MicroSeconds(), NANOS=<null>)) |
| ./oci_focus_export.snappy.parquet | ChargeSubcategory          | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | CommitmentDiscountCategory | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | CommitmentDiscountId       | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | CommitmentDiscountName     | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | CommitmentDiscountType     | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | EffectiveCost              | DOUBLE     |             | OPTIONAL        |              |                  |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | InvoiceIssuer              | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | ListCost                   | DOUBLE     |             | OPTIONAL        |              |                  |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | ListUnitPrice              | DOUBLE     |             | OPTIONAL        |              |                  |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | PricingCategory            | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | PricingQuantity            | DOUBLE     |             | OPTIONAL        |              |                  |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | PricingUnit                | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | Provider                   | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | Publisher                  | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | Region                     | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | ResourceId                 | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | ResourceName               | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | ResourceType               | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | ServiceCategory            | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | ServiceName                | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | SkuId                      | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | SkuPriceId                 | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | SubAccountId               | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | SubAccountName             | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | Tags                       | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | UsageQuantity              | DOUBLE     |             | OPTIONAL        |              |                  |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | UsageUnit                  | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | oci_ReferenceNumber        | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | oci_CompartmentId          | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | oci_CompartmentName        | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | oci_OverageFlag            | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | oci_UnitPriceOverage       | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | oci_BilledQuantityOverage  | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | oci_CostOverage            | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | oci_AttributedUsage        | DOUBLE     |             | OPTIONAL        |              |                  |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | oci_AttributedCost         | DOUBLE     |             | OPTIONAL        |              |                  |       |           |          |                                                                                                     |
| ./oci_focus_export.snappy.parquet | oci_BackReferenceNumber    | BYTE_ARRAY |             | OPTIONAL        |              | UTF8             |       |           |          |                                                                                                     |


We can notice that OCI custom columns don't follow the FOCUS 1.0 standard (Should start with `x_` and not `oci_`).

Therefore we will transform these custom columns on the ingestion job (E.g.: `oci_ReferenceNumber` to `x_ReferenceNumber`).

In addition, OCI FOCUS exports exposes non-FOCUS 1.0 columns such as `ChargeSubcategory` and `UsageQuantity`.

Therefore we will transform them into custom columns using the `x_` prefix and will make them compliant with Pascal Case column naming (E.g. `ChargeSubcategory` to `x_ChargeSubCategory`)

Finally, some FOCUS columns are not correctly named, such `InvoiceIssuer` and `Provider` and `Publisher` and `Region`.

Therefore, we will transform them during the ingestion such as `InvoiceIssuer` to `InvoiceIssuerName` and `Region` to `RegionId` and `RegionName`.
