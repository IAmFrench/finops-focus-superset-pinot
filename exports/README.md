# Export folder

This folder will be mounted to Apache Pinot for data ingestion jobs.

We use one folder per Cloud Service Provider then we duplicate the structure defined by each CSP.

To populate these folders, you can use the [`download_focus_export.sh`](../scripts/download_focus_export.sh) scripts located in the [../scripts](../scripts/) folder:

- AWS
```bash
../scripts/download_focus_export.sh \
    -p aws \
    -b bucket_name \
    -e export_name \
    -d export/directory \
    -o /path/to/exports/folder/for/aws/
```
- Azure
```bash
../scripts/download_focus_export.sh \
    -p azure \
    -b storage_account_name \
    -c container_name \
    -e export_name \
    -d export/directory \
    -o /path/to/exports/folder/for/azure/
```
- OCI
```bash
../scripts/download_focus_export.sh \
    -p oci \
    -b bucket_name \
    -o /path/to/exports/folder/for/oci/
```

Once populated with FOCUS exports, the exports folder should look like that:
```
- aws
-- BILLING_PERIOD=2024-06
--- focus-1-0-preview-export-00001.snappy.parquet
--- focus-1-0-preview-export-00002.snappy.parquet
--- focus-1-0-preview-export-00003.snappy.parquet
-- BILLING_PERIOD=2024-07
--- focus-1-0-preview-export-00001.snappy.parquet
--- focus-1-0-preview-export-00002.snappy.parquet
--- focus-1-0-preview-export-00003.snappy.parquet
-- BILLING_PERIOD=2024-08
--- focus-1-0-preview-export-00001.snappy.parquet
--- focus-1-0-preview-export-00002.snappy.parquet
--- focus-1-0-preview-export-00003.snappy.parquet
--- focus-1-0-preview-export-00004.snappy.parquet
- azure
-- 20240601-20240730
--- 01abc023-a123-4567-a12b-123ab3c5ab23
---- manifest.json
---- part_0_0001.snappy.parquet
---- part_1_0001.snappy.parquet
--- 01abc023-a123-4567-a12b-123ab3c5ab23
---- manifest.json
---- part_0_0001.snappy.parquet
---- part_1_0001.snappy.parquet
-- 20240801-20240731
--- 01abc023-a123-4567-a12b-123ab3c5ab23
---- manifest.json
---- part_0_0001.snappy.parquet
---- part_1_0001.snappy.parquet
- oci
-- 0001000001507847-00001.csv.gz
-- 0001000001511187-00001.csv.gz
-- 0001000001512411-00001.csv.gz
-- 0001000001513618-00001.csv.gz
-- 0001000001514794-00001.csv.gz
-- 0001000001518066-00001.csv.gz
-- 0001000001519299-00001.csv.gz
-- 0001000001520484-00001.csv.gz
-- 0001000001521629-00001.csv.gz
-- 0001000001524995-00001.csv.gz
-- 0001000001526258-00001.csv.gz
-- 0001000001527428-00001.csv.gz
```

## References

- AWS: https://docs.aws.amazon.com/cur/latest/userguide/table-dictionary-focus-1-0-aws-columns.html
- Azure: https://learn.microsoft.com/en-us/azure/cost-management-billing/dataset-schema/cost-usage-details-focus#version-10
- OCI: https://docs.oracle.com/en-us/iaas/Content/Billing/Concepts/costusagereportsoverview.htm#costreports__focus-cost-report-schema
