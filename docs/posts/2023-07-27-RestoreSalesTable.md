---
title: AX2009 Restore deleted sales table without restoring sales line
description: Restore deleted sales table without restoring sales line with X++
date: 2023-07-27
authors:
    - downardo
---

## Problem

There are sometimes constelations where the user deletes the header of the sales order (**sales table**), but AX2009 will not delete the appropiate sales line entries. So we end up with open sales line entries without the header. That causes some problems when posting invoices and so on.

<!-- more -->

## Solution

You could try to cancel the entries directly over AOT in the databrowser. But I went with the approach to restore the header of the salesorder from **SalesTableDelete** without restoring the sales lines. And after that the user can manually cancel the sales order.

### AX2009 Job

```xpp
static void down1_RestoreSalesTableWithoutSalesLine(Args _args)
{
    SalesTableDelete    ltabsalesTableDelete;
    SalesTable          ltabsalesTable;
    ;
    ltabsalesTableDelete = SalesTableDelete::find('A103990087', true);
    ttsbegin;
    switch (ltabsalesTableDelete.Cancelled)
    {
        case Voided::Voided :
            ltabsalesTable  = conpeek(ltabsalesTableDelete.SalesTable, 1);
            ltabsalesTable.insert();
            ltabsalesTableDelete.delete();
            break;
   }
   ttscommit;
}
```
