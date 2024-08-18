# About the `./exports/aws` directory

This folder holds AWS FOCUS billing export

# How to download AWS FOCUS export?

Let's assume you have created a FOCUS export to the `finops-exports-1a2b3c4d` s3 bucket using the [`billing-export` Terraform module for AWS](https://registry.terraform.io/modules/IAmFrench/billing-export/aws/latest).

This bucket has the folling structure:
```bash
# Set the Name of the bucket where AWS FOCUS v1.0-preview exports are
export AWS_FOCUS_EXPORT_BUCKET="finops-exports-1a2b3c4d"
export AWS_EXPORT_S3_PREFIX="focus/123456789"

# List all objects in the s3 bucket
aws s3 ls s3://${AWS_FOCUS_EXPORT_BUCKET}/${AWS_EXPORT_S3_PREFIX}/ --recursive --summarize --human-readable

2024-07-06 05:00:09   56.2 MiB focus-1-0-preview-export/data/BILLING_PERIOD=2024-06/focus-1-0-preview-export-00001.snappy.parquet
2024-07-06 05:00:09   56.1 MiB focus-1-0-preview-export/data/BILLING_PERIOD=2024-06/focus-1-0-preview-export-00002.snappy.parquet
2024-07-06 05:00:09   56.0 MiB focus-1-0-preview-export/data/BILLING_PERIOD=2024-06/focus-1-0-preview-export-00003.snappy.parquet
2024-07-06 05:00:29    2.6 KiB focus-1-0-preview-export/metadata/BILLING_PERIOD=2024-06/focus-1-0-preview-export-Manifest.json
2024-07-27 11:53:52   47.7 MiB focus-1-0-preview-export/data/BILLING_PERIOD=2024-07/focus-1-0-preview-export-00001.snappy.parquet
2024-07-27 11:53:52   47.6 MiB focus-1-0-preview-export/data/BILLING_PERIOD=2024-07/focus-1-0-preview-export-00002.snappy.parquet
2024-07-27 11:53:52   47.7 MiB focus-1-0-preview-export/data/BILLING_PERIOD=2024-07/focus-1-0-preview-export-00003.snappy.parquet
2024-07-27 11:54:02    2.6 KiB focus-1-0-preview-export/metadata/BILLING_PERIOD=2024-07/focus-1-0-preview-export-Manifest.json

Total Objects: 8
Total Size: 311.3 MiB
```

You can then use the following script [`./scripts/download_focus_export.sh`](./scripts/download_focus_export.sh) to download your exports:

```bash
../scripts/download_focus_export.sh \
    -p aws \
    -b finops-exports-1a2b3c4d \
    -e focus-1-0-preview-export \
    -d focus/123456789 \
    -o ./exports/aws/
```

# Example

```bash
du -a ./exports/aws

57588   ./exports/aws/BILLING_PERIOD=2024-06/focus-1-0-preview-export-00001.snappy.parquet
57412   ./exports/aws/BILLING_PERIOD=2024-06/focus-1-0-preview-export-00002.snappy.parquet
57316   ./exports/aws/BILLING_PERIOD=2024-06/focus-1-0-preview-export-00003.snappy.parquet
172316  ./exports/aws/BILLING_PERIOD=2024-06
44004   ./exports/aws/BILLING_PERIOD=2024-07/focus-1-0-preview-export-00001.snappy.parquet
43904   ./exports/aws/BILLING_PERIOD=2024-07/focus-1-0-preview-export-00002.snappy.parquet
44000   ./exports/aws/BILLING_PERIOD=2024-07/focus-1-0-preview-export-00003.snappy.parquet
44020   ./exports/aws/BILLING_PERIOD=2024-07/focus-1-0-preview-export-00004.snappy.parquet
175928  ./exports/aws/BILLING_PERIOD=2024-07
27992   ./exports/aws/BILLING_PERIOD=2024-08/focus-1-0-preview-export-00001.snappy.parquet
27992   ./exports/aws/BILLING_PERIOD=2024-08
0       ./exports/aws/README.md
376236  ./exports/aws
```

# Table Schema for AWS `.snappy.parquet` file

```bash
duckdb -markdown -c "select * from parquet_schema('./focus-1-0-preview-export-00001.snappy.parquet');"
```

|                    file_name                    |            name            |    type    | type_length | repetition_type | num_children | converted_type | scale | precision | field_id | logical_type |
|-------------------------------------------------|----------------------------|------------|-------------|-----------------|-------------:|----------------|-------|-----------|----------|--------------|
| ./focus-1-0-preview-export-00001.snappy.parquet | spark_schema               |            |             |                 | 48           |                |       |           |          |              |
| ./focus-1-0-preview-export-00001.snappy.parquet | AvailabilityZone           | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | BilledCost                 | DOUBLE     |             | OPTIONAL        |              |                |       |           |          |              |
| ./focus-1-0-preview-export-00001.snappy.parquet | BillingAccountId           | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | BillingAccountName         | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | BillingCurrency            | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | BillingPeriodEnd           | INT96      |             | OPTIONAL        |              |                |       |           |          |              |
| ./focus-1-0-preview-export-00001.snappy.parquet | BillingPeriodStart         | INT96      |             | OPTIONAL        |              |                |       |           |          |              |
| ./focus-1-0-preview-export-00001.snappy.parquet | ChargeCategory             | BYTE_ARRAY |             | REQUIRED        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | ChargeClass                | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | ChargeDescription          | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | ChargeFrequency            | BYTE_ARRAY |             | REQUIRED        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | ChargePeriodEnd            | INT96      |             | OPTIONAL        |              |                |       |           |          |              |
| ./focus-1-0-preview-export-00001.snappy.parquet | ChargePeriodStart          | INT96      |             | OPTIONAL        |              |                |       |           |          |              |
| ./focus-1-0-preview-export-00001.snappy.parquet | CommitmentDiscountCategory | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | CommitmentDiscountId       | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | CommitmentDiscountName     | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | CommitmentDiscountType     | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | CommitmentDiscountStatus   | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | ConsumedQuantity           | DOUBLE     |             | OPTIONAL        |              |                |       |           |          |              |
| ./focus-1-0-preview-export-00001.snappy.parquet | ConsumedUnit               | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | ContractedCost             | DOUBLE     |             | OPTIONAL        |              |                |       |           |          |              |
| ./focus-1-0-preview-export-00001.snappy.parquet | ContractedUnitPrice        | DOUBLE     |             | OPTIONAL        |              |                |       |           |          |              |
| ./focus-1-0-preview-export-00001.snappy.parquet | EffectiveCost              | DOUBLE     |             | OPTIONAL        |              |                |       |           |          |              |
| ./focus-1-0-preview-export-00001.snappy.parquet | InvoiceIssuerName          | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | ListCost                   | DOUBLE     |             | OPTIONAL        |              |                |       |           |          |              |
| ./focus-1-0-preview-export-00001.snappy.parquet | ListUnitPrice              | DOUBLE     |             | OPTIONAL        |              |                |       |           |          |              |
| ./focus-1-0-preview-export-00001.snappy.parquet | PricingCategory            | BYTE_ARRAY |             | REQUIRED        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | PricingQuantity            | DOUBLE     |             | OPTIONAL        |              |                |       |           |          |              |
| ./focus-1-0-preview-export-00001.snappy.parquet | PricingUnit                | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | ProviderName               | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | PublisherName              | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | RegionId                   | BYTE_ARRAY |             | REQUIRED        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | RegionName                 | BYTE_ARRAY |             | REQUIRED        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | ResourceId                 | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | ResourceName               | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | ResourceType               | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | ServiceCategory            | BYTE_ARRAY |             | REQUIRED        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | ServiceName                | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | SkuId                      | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | SkuPriceId                 | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | SubAccountId               | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | SubAccountName             | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | Tags                       |            |             | OPTIONAL        | 1            | MAP            |       |           |          | MapType()    |
| ./focus-1-0-preview-export-00001.snappy.parquet | key_value                  |            |             | REPEATED        | 2            |                |       |           |          |              |
| ./focus-1-0-preview-export-00001.snappy.parquet | key                        | BYTE_ARRAY |             | REQUIRED        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | value                      | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | x_CostCategories           |            |             | OPTIONAL        | 1            | MAP            |       |           |          | MapType()    |
| ./focus-1-0-preview-export-00001.snappy.parquet | key_value                  |            |             | REPEATED        | 2            |                |       |           |          |              |
| ./focus-1-0-preview-export-00001.snappy.parquet | key                        | BYTE_ARRAY |             | REQUIRED        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | value                      | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | x_Discounts                |            |             | OPTIONAL        | 1            | MAP            |       |           |          | MapType()    |
| ./focus-1-0-preview-export-00001.snappy.parquet | key_value                  |            |             | REPEATED        | 2            |                |       |           |          |              |
| ./focus-1-0-preview-export-00001.snappy.parquet | key                        | BYTE_ARRAY |             | REQUIRED        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | value                      | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | x_Operation                | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | x_ServiceCode              | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
| ./focus-1-0-preview-export-00001.snappy.parquet | x_UsageType                | BYTE_ARRAY |             | OPTIONAL        |              | UTF8           |       |           |          | StringType() |
