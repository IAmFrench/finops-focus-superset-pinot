# About the `./exports/azure` directory

This folder holds Azure FOCUS billing export

# How to download Azure FOCUS export?

Let's assume you have created a FOCUS export with the following configuration:

Summary
- Name: `focus-export`
- Type of data: `Cost and usage details (FOCUS) - Preview`
- Frequency: Daily `export of month-to-date costs`
- Dataset version: `1.0`

Destination
- Storage account type: `Azure blob storage`
- Subscription: `e0a20953-d910-45f0-bcb2-c3883198edc7` (This is a fake uuid)
- Storage account name: `finops-exports-1a2b3c4d`
- Container: `focus`
- Directory: `123456789`

Settings
- File partitioning: `On`
- Overwrite data: `On`
- Format: `Parquet`
- Compression type: `Snappy`

The `focus` container in the `finops-exports-1a2b3c4d` storage account should have this structure:

```bash
# Name of the storage account
export AZURE_STORAGE_ACCOUNT_NAME="finops-exports-1a2b3c4d"
# Name of the container
export AZURE_CONTAINER_NAME="focus"
# Name of the directory
export AZURE_DIRECTORY="123456789"

az storage blob directory list --account-name $AZURE_STORAGE_ACCOUNT_NAME --container-name $AZURE_CONTAINER_NAME -d $AZURE_DIRECTORY --output table

Name                                                                                                                    IsDirectory    Blob Type    Blob Tier    Length    Content Type              Last Modified              Snapshot
----------------------------------------------------------------------------------------------------------------------  -------------  -----------  -----------  --------  ------------------------  -------------------------  ----------
123456789/focus-export/20240701-20240731/da16755b-a123-40ed-8850-e6025c77b987/manifest.json                              BlockBlob    Hot          1352      application/octet-stream  2024-08-13T17:00:41+00:00
123456789/focus-export/20240701-20240731/da16755b-a123-40ed-8850-e6025c77b987/part_0_0001.snappy.parquet                 BlockBlob    Hot          50350     application/octet-stream  2024-08-13T17:00:41+00:00
123456789/focus-export/20240801-20240831/311a485b-1176-4504-b59b-5dadc0d0a2e7/manifest.json                              BlockBlob    Hot          1351      application/octet-stream  2024-08-13T17:38:40+00:00
123456789/focus-export/20240801-20240831/311a485b-1176-4504-b59b-5dadc0d0a2e7/part_0_0001.snappy.parquet                 BlockBlob    Hot          42719     application/octet-stream  2024-08-13T17:38:40+00:00
```

You can then use the following script [`./scripts/download_focus_export.sh`](./scripts/download_focus_export.sh) to download your exports:

```bash
../scripts/download_focus_export.sh \
    -p azure \
    -b finops-exports-1a2b3c4d \
    -c focus \
    -e focus-export \
    -d 123456789 \
    -o ./exports/azure/
```

# Example

```bash
du -a ./exports/azure

4       ./exports/azure/20240701-20240731/da16755b-a123-40ed-8850-e6025c77b987/manifest.json
184     ./exports/azure/20240701-20240731/da16755b-a123-40ed-8850-e6025c77b987/part_0_0001.snappy.parquet
448360  ./exports/azure/20240701-20240731/da16755b-a123-40ed-8850-e6025c77b987/part_1_0001.snappy.parquet
448548  ./exports/azure/20240701-20240731/da16755b-a123-40ed-8850-e6025c77b987
448548  ./exports/azure/20240701-20240731
4       ./exports/azure/20240801-20240831/311a485b-1176-4504-b59b-5dadc0d0a2e7/manifest.json
112     ./exports/azure/20240801-20240831/311a485b-1176-4504-b59b-5dadc0d0a2e7/part_0_0001.snappy.parquet
79324   ./exports/azure/20240801-20240831/311a485b-1176-4504-b59b-5dadc0d0a2e7/part_1_0001.snappy.parquet
79440   ./exports/azure/20240801-20240831/311a485b-1176-4504-b59b-5dadc0d0a2e7
79440   ./exports/azure/20240801-20240831
4       ./exports/azure/README.md
527992  ./exports/azure
```

# Table Schema of Azure `.snappy.parquet` file

```bash
duckdb -markdown -c "select * from parquet_schema('./part_0_0001.snappy.parquet');"
```

|          file_name           |            name            |         type         | type_length | repetition_type | num_children | converted_type | scale | precision | field_id |            logical_type             |
|------------------------------|----------------------------|----------------------|-------------|-----------------|-------------:|----------------|-------|-----------|----------|-------------------------------------|
| .\part_0_0001.snappy.parquet | root                       |                      |             |                 | 96           |                |       |           |          |                                     |
| .\part_0_0001.snappy.parquet | BilledCost                 | FIXED_LEN_BYTE_ARRAY | 16          | OPTIONAL        |              | DECIMAL        | 18    | 38        |          | DecimalType(scale=18, precision=38) |
| .\part_0_0001.snappy.parquet | BillingAccountId           | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | BillingAccountName         | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | BillingAccountType         | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | BillingCurrency            | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | BillingPeriodEnd           | INT96                |             | OPTIONAL        |              |                |       |           |          |                                     |
| .\part_0_0001.snappy.parquet | BillingPeriodStart         | INT96                |             | OPTIONAL        |              |                |       |           |          |                                     |
| .\part_0_0001.snappy.parquet | ChargeCategory             | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | ChargeClass                | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | ChargeDescription          | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | ChargeFrequency            | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | ChargePeriodEnd            | INT96                |             | OPTIONAL        |              |                |       |           |          |                                     |
| .\part_0_0001.snappy.parquet | ChargePeriodStart          | INT96                |             | OPTIONAL        |              |                |       |           |          |                                     |
| .\part_0_0001.snappy.parquet | CommitmentDiscountCategory | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | CommitmentDiscountId       | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | CommitmentDiscountName     | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | CommitmentDiscountStatus   | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | CommitmentDiscountType     | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | ConsumedQuantity           | FIXED_LEN_BYTE_ARRAY | 16          | OPTIONAL        |              | DECIMAL        | 18    | 38        |          | DecimalType(scale=18, precision=38) |
| .\part_0_0001.snappy.parquet | ConsumedUnit               | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | ContractedCost             | FIXED_LEN_BYTE_ARRAY | 16          | OPTIONAL        |              | DECIMAL        | 18    | 38        |          | DecimalType(scale=18, precision=38) |
| .\part_0_0001.snappy.parquet | ContractedUnitPrice        | FIXED_LEN_BYTE_ARRAY | 16          | OPTIONAL        |              | DECIMAL        | 18    | 38        |          | DecimalType(scale=18, precision=38) |
| .\part_0_0001.snappy.parquet | EffectiveCost              | FIXED_LEN_BYTE_ARRAY | 16          | OPTIONAL        |              | DECIMAL        | 18    | 38        |          | DecimalType(scale=18, precision=38) |
| .\part_0_0001.snappy.parquet | InvoiceIssuerName          | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | ListCost                   | FIXED_LEN_BYTE_ARRAY | 16          | OPTIONAL        |              | DECIMAL        | 18    | 38        |          | DecimalType(scale=18, precision=38) |
| .\part_0_0001.snappy.parquet | ListUnitPrice              | FIXED_LEN_BYTE_ARRAY | 16          | OPTIONAL        |              | DECIMAL        | 18    | 38        |          | DecimalType(scale=18, precision=38) |
| .\part_0_0001.snappy.parquet | PricingCategory            | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | PricingQuantity            | FIXED_LEN_BYTE_ARRAY | 16          | OPTIONAL        |              | DECIMAL        | 18    | 38        |          | DecimalType(scale=18, precision=38) |
| .\part_0_0001.snappy.parquet | PricingUnit                | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | ProviderName               | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | PublisherName              | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | RegionId                   | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | RegionName                 | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | ResourceId                 | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | ResourceName               | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | ResourceType               | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | ServiceCategory            | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | ServiceName                | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | SkuId                      | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | SkuPriceId                 | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | SubAccountId               | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | SubAccountName             | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | SubAccountType             | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | Tags                       | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_AccountId                | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_AccountName              | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_AccountOwnerId           | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_BilledCostInUsd          | FIXED_LEN_BYTE_ARRAY | 16          | OPTIONAL        |              | DECIMAL        | 18    | 38        |          | DecimalType(scale=18, precision=38) |
| .\part_0_0001.snappy.parquet | x_BilledUnitPrice          | FIXED_LEN_BYTE_ARRAY | 16          | OPTIONAL        |              | DECIMAL        | 18    | 38        |          | DecimalType(scale=18, precision=38) |
| .\part_0_0001.snappy.parquet | x_BillingAccountId         | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_BillingAccountName       | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_BillingExchangeRate      | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_BillingExchangeRateDate  | INT96                |             | OPTIONAL        |              |                |       |           |          |                                     |
| .\part_0_0001.snappy.parquet | x_BillingProfileId         | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_BillingProfileName       | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_ContractedCostInUsd      | FIXED_LEN_BYTE_ARRAY | 16          | OPTIONAL        |              | DECIMAL        | 18    | 38        |          | DecimalType(scale=18, precision=38) |
| .\part_0_0001.snappy.parquet | x_CostAllocationRuleName   | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_CostCenter               | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_CustomerId               | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_CustomerName             | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_EffectiveCostInUsd       | FIXED_LEN_BYTE_ARRAY | 16          | OPTIONAL        |              | DECIMAL        | 18    | 38        |          | DecimalType(scale=18, precision=38) |
| .\part_0_0001.snappy.parquet | x_EffectiveUnitPrice       | FIXED_LEN_BYTE_ARRAY | 16          | OPTIONAL        |              | DECIMAL        | 18    | 38        |          | DecimalType(scale=18, precision=38) |
| .\part_0_0001.snappy.parquet | x_InvoiceId                | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_InvoiceIssuerId          | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_InvoiceSectionId         | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_InvoiceSectionName       | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_ListCostInUsd            | FIXED_LEN_BYTE_ARRAY | 16          | OPTIONAL        |              | DECIMAL        | 18    | 38        |          | DecimalType(scale=18, precision=38) |
| .\part_0_0001.snappy.parquet | x_PartnerCreditApplied     | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_PartnerCreditRate        | FIXED_LEN_BYTE_ARRAY | 16          | OPTIONAL        |              | DECIMAL        | 18    | 38        |          | DecimalType(scale=18, precision=38) |
| .\part_0_0001.snappy.parquet | x_PricingBlockSize         | FIXED_LEN_BYTE_ARRAY | 16          | OPTIONAL        |              | DECIMAL        | 18    | 38        |          | DecimalType(scale=18, precision=38) |
| .\part_0_0001.snappy.parquet | x_PricingCurrency          | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_PricingSubcategory       | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_PricingUnitDescription   | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_PublisherCategory        | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_PublisherId              | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_ResellerId               | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_ResellerName             | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_ResourceGroupName        | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_ResourceType             | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_ServicePeriodEnd         | INT96                |             | OPTIONAL        |              |                |       |           |          |                                     |
| .\part_0_0001.snappy.parquet | x_ServicePeriodStart       | INT96                |             | OPTIONAL        |              |                |       |           |          |                                     |
| .\part_0_0001.snappy.parquet | x_SkuDescription           | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_SkuDetails               | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_SkuIsCreditEligible      | BOOLEAN              |             | OPTIONAL        |              |                |       |           |          |                                     |
| .\part_0_0001.snappy.parquet | x_SkuMeterCategory         | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_SkuMeterId               | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_SkuMeterName             | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_SkuMeterSubcategory      | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_SkuOfferId               | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_SkuOrderId               | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_SkuOrderName             | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_SkuPartNumber            | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_SkuRegion                | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_SkuServiceFamily         | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_SkuTerm                  | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
| .\part_0_0001.snappy.parquet | x_SkuTier                  | BYTE_ARRAY           |             | OPTIONAL        |              | UTF8           |       |           |          | StringType()                        |
