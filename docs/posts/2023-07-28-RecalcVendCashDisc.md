---
title: AX2009 Recalculate Vendor Cash Disc
slug: AX2009 Recalculate Vendor Cash Disc with X++
date: 2023-07-28
authors:
    - downardo
---

## Description

With this job you can recalculate the cash disc amount in the vendor open transactions. You can change the query to filter different transactions.
<!-- more -->
## AX2009 Job

```xpp
static void down1_VendRecalcCashDisc(Args _args)
{
    VendTransCashDisc   ltabVendTransCashDisc;
    VendTransOpen       ltabVendTransOpen;
    VendTrans           ltabVendTrans;
    VendInvoiceJour     ltabVendInvoiceJour;
    int                 i;
    ;

    while select ltabVendTrans
           where ltabVendTrans.CashDiscCode != ""
//&& (ltabVendTrans.AmountCur > 0.36 || ltabVendTrans.AmountCur < -0.36)
            join ltabVendTransOpen
           where ltabVendTransOpen.RefRecId == ltabVendTrans.RecId
  notexists join ltabVendTransCashDisc
           where ltabVendTransCashDisc.RefTableId       == ltabVendTransOpen.TableId
              && ltabVendTransCashDisc.RefRecId         == ltabVendTransOpen.RecId
              && ltabVendTransCashDisc.CashDiscAmount   != 0
    {
        ltabVendInvoiceJour = VendInvoiceJour::findFromVendTrans(ltabVendTrans.Invoice, ltabVendTrans.TransDate, ltabVendTrans.AccountNum);

        info(ltabVendTrans.AccountNum + " " + ltabVendTrans.Invoice + " " + ltabVendInvoiceJour.createdBy);

        ltabVendTransCashDisc.calcCashDisc(ltabVendTrans.CurrencyCode, ltabVendTrans.AmountCur, ltabVendTrans.DueDate,
            ltabVendTrans.DocumentDate ? ltabVendTrans.DocumentDate : ltabVendTrans.TransDate,
            ltabVendTrans.CashDiscCode, ltabVendTransOpen.TableId, ltabVendTransOpen.RecId);

        if (ltabVendTrans.AmountCur < -1 || ltabVendTrans.AmountCur > 1)
            i++;
    }
    info(strfmt("Count of mutated entries: %1",i));
}
```
