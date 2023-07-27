---
title: AX2009 Settle Customer Open Transcation
description: AX2009 Settle Customer Open Transcation with X++
---

## Description

We have sometimes the problem in AX2009 that a voucher was posted the settle an open transaction of a customer but only a partial amount has been settled. So the job search all open transaction that should have been closed and settles them.

## AX2009 Job

```text
static void down1_SettleCustOpenTransactions(Args _args)
{
    SpecTransManager    lclsSpecTransManager;
    CustTrans           ltabCustTrans, ltabCustTransSelect;
    CustTrans           ltabCustTransSecond;
    CustTransOpen       ltabCustTransOpen, ltabCustTransOpenSelect;
    CustSettlement      ltabCustSettlement;
    CustTable           ltabCustTable;
    boolean             lbolSettle;
    Set                 lsetRecIds = new Set(Types::Int64);
    int                 i;
    ;

    while select ltabCustTable
     exists join ltabCustTransSelect
           where ltabCustTransSelect.AccountNum == ltabCustTable.AccountNum
     exists join ltabCustTransOpenSelect
           where ltabCustTransOpenSelect.AccountNum == ltabCustTable.AccountNum
              && ltabCustTransSelect.RecId          == ltabCustTransOpenSelect.RefRecId
              && ltabCustTransSelect.AmountCur      != ltabCustTransOpenSelect.AmountCur
    {
        i++; print i;
        lbolSettle = false;
        while select ltabCustTrans
               where ltabCustTrans.Invoice
                  && ltabCustTrans.AccountNum   == ltabCustTable.AccountNum
                  && ltabCustTrans.AmountCur    >  0
                join firstOnly ltabCustTransOpen
               where ltabCustTransOpen.RefRecId == ltabCustTrans.RecId
                join firstOnly ltabCustSettlement
               where ltabCustSettlement.TransRecId  == ltabCustTrans.RecId
                join firstOnly ltabCustTransSecond
               where ltabCustTransSecond.AccountNum == ltabCustTrans.AccountNum
                  && ltabCustTransSecond.Voucher    == ltabCustSettlement.OffsetTransVoucher
        {
            if (lsetRecIds.in(ltabCustTrans.RecId))
                continue;
            if (ltabCustTransOpen && abs(ltabCustTrans.remainAmountCur()) && abs(ltabCustTrans.remainAmountCur()) == abs(ltabCustTransSecond.remainAmountCur()))
            {
                lclsSpecTransManager = SpecTransManager::construct(ltabCustTable);
                lclsSpecTransManager.insert(curExt(), tableNum(CustTransOpen), ltabCustTransSecond.transOpen().RecId, ltabCustTransSecond.remainAmountCur(), ltabCustTransSecond.CurrencyCode);
                lclsSpecTransManager.insert(curExt(), tableNum(CustTransOpen), ltabCustTrans.transOpen().RecId, ltabCustTrans.remainAmountCur(), ltabCustTransSecond.CurrencyCode);
                lbolSettle = true;
            }
            lsetRecIds.add(ltabCustTrans.RecId);
        }
        if (lbolSettle)
        {
            info(ltabCustTable.AccountNum);
            CustTrans::settleTransact(ltabCustTable, null, true, SettleDatePrinc::SelectDate, systemDateGet());
        }
    }
}
```
