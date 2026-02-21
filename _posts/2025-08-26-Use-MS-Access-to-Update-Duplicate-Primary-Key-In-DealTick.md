# Use MS Access to Update Duplicate DealTick Table and TradingSheetTbl

## DealTick Table

* Create DealTick2 Table by Copy and Paste(structure only)
* Set DealTick2 ID field as AutoNumber and Primary Key
* Insert(Append) to DealTick2 from DealTick without ID

```sql
INSERT INTO DealTick2 
   ( iDealNo, iClientNo, iBrokerNo, fScreenRate, 
   fRate, fAmount, fServiceCharge, iCurrencyType, dValueDate, iValueTerm, 
   dQuoteDate, sBank, iCheckNo, bCommit, Paid, Buy, iCurrencyType2, fScreenRate2, 
   fEquivAmount, bVoid, bPrinted, cProfit, cCommission, cExp, fCADRate, fCalculateRate, 
   iSalesRepNo, iDeliveryMethod, bReverse, bReversed, iReferTicketNo, iState, bAudit, 
   fCarryRate, fSpreadRate, cMiscCharge, sMemo, iPayment, sPayment, bPoliticalExposed, 
   bThirdParty, bFintracReport, sRelationship, sFintracMethod, cPoliticalExposed, sPoliticalExposedReason )
SELECT  iDealNo, iClientNo, iBrokerNo, fScreenRate, 
   fRate, fAmount, fServiceCharge, iCurrencyType, dValueDate, iValueTerm, 
   dQuoteDate, sBank, iCheckNo, bCommit, Paid, Buy, iCurrencyType2, fScreenRate2, 
   fEquivAmount, bVoid, bPrinted, cProfit, cCommission, cExp, fCADRate, fCalculateRate, 
   iSalesRepNo, iDeliveryMethod, bReverse, bReversed, iReferTicketNo, iState, bAudit, 
   fCarryRate, fSpreadRate, cMiscCharge, sMemo, iPayment, sPayment, bPoliticalExposed, 
   bThirdParty, bFintracReport, sRelationship, sFintracMethod, cPoliticalExposed, sPoliticalExposedReason 
FROM DealTick 
ORDER BY dQuoteDate

```

* verify if duplicate primary key still exist

```sql
SELECT ID, COUNT(*) AS DuplicateCount
FROM DealTick2
GROUP BY ID
HAVING COUNT(*) > 1;
```

* delete original DealTick table and rename DealTick2 to DealTick

## TradeSheetTbl Table

* Create TradeSheetTbl2 by copy original tradeSheetTbl and structure only
* Set TradingSheetTbl2 ID field as AutoNumber and Primary Key
* use the follow sql query to import TradingSheetTbl2 without ID(field)

```sql
INSERT INTO TradeSheetTbl2 
(iDealNo, iClientNo, iBrokerNo, fScreenRate, 
fRate, fAmount, fServiceCharge, iCurrencyType, dValueDate, iValueTerm, dQuoteDate, sBank, iCheckNo, bCommit, Paid, Buy, iCurrencyType2, fScreenRate2, fEquivAmount, bVoid, bPrinted, cProfit, cCommission, cExp, fCADRate, fCalculateRate, iSalesRepNo, iDeliveryMethod, bReverse, bReversed, iReferTicketNo, iState, bAudit, fCarryRate, fSpreadRate)
SELECT 
iDealNo, iClientNo, iBrokerNo, fScreenRate, 
fRate, fAmount, fServiceCharge, iCurrencyType, dValueDate, iValueTerm, dQuoteDate, sBank, iCheckNo, bCommit, Paid, Buy, iCurrencyType2, fScreenRate2, fEquivAmount, bVoid, bPrinted, cProfit, cCommission, cExp, fCADRate, fCalculateRate, iSalesRepNo, iDeliveryMethod, bReverse, bReversed, iReferTicketNo, iState, bAudit, fCarryRate, fSpreadRate
FROM TradeSheetTbl
ORDER BY dQuoteDate

```

* verify if duplicate primary key still exist

```sql
SELECT ID, COUNT(*) AS DuplicateCount
FROM TradeSheetTbl2
GROUP BY ID
HAVING COUNT(*) > 1;
```

* delete original TradingSheetTbl and rename TradingSheetTbl2 to TradingSheetTbl
