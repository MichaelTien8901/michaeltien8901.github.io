# RCG Database change for Customer and DealTick table for Broker Changes

## Changes need

* Ticket broker name change from Alice to Kam
* Customer broker assigned to Alice change to Kam

## Basic Information

| Broker Name | Broker Number |
|-------------|---------------|
| Alice       | 18            |
| Kam         | 9             |

## DealTick Database Changes

```sql
UPDATE DealTick
SET iBrokerNo = 9
WHERE iBrokerNo = 18
  AND dQuoteDate > #2025/01/01#;

```

## Customer Database Changes

```sql
UPDATE Customer
SET iBrokerID = 9
WHERE iBrokerID = 18
```

## Disable Alice Account

## Backup DealTick and restore code

1. backup 

```sql
SELECT * INTO DealTick_Backup_20250510
FROM DealTick
WHERE iBrokerNo = 18
  AND dQuoteDate > #2025/01/01#;
```

2. Then run the update
```sql
UPDATE DealTick
SET iBrokerNo = 9
WHERE iBrokerNo = 18
  AND dQuoteDate > #2025/01/01#;
```

3. restore

```sql
UPDATE DealTick
INNER JOIN DealTick_Backup_20250510
ON DealTick.iDealNo = DealTick_Backup_20250510.iDealNo
SET DealTick.iBrokerNo = DealTick_Backup_20250510.iBrokerNo;
```

  * Steps to restore:

     * This joins the live DealTick table to the backup using iDealNo.
     * It then sets iBrokerNo back to the original value from the backup.

     