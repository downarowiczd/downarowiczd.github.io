---
title: AX2009 Cancel SalesOrder with empty SalesLine
description: With the help of this job, you can cancel a sales order that has an empty salesline
---

## Description

One customer had the the problem, that some of their sales employees cancelled sales orders that they created by deleting all sales lines and letting the sales order open.
So now the customer had the problem that they had many sales orders open, and all the queries in their batch job selected more and more orders to check them. So some batches were taking much longer. To solve the problem I wrote a job that selectes all open sales orders (status = backorder) with a position count of 0 (sales line count). For the sales line count I used a already made function by use in the SalesTable controller class.

```text
public RefRecId getPositionCount()
{
    SalesLine     ltabSalesLine;
    ;

    select count (RecId) from ltabSalesLine
           where ltabSalesLine.SalesId == mtabSalesTable.SalesId;

    return ltabSalesLine.RecId;
}
```

After finding a sales order for that, the script creates a new sales line entry with one of our dummy items (999999999). And after that we cancel the appropriate position in the system. The part **InterCompanyUpdateRemPhys::synchronize** is needed to update sales remain amount in the sales line. It is doing exactly the same, as the form **SalesUpdateRemain** in AX2009. As the difference from remain qty to 0 (delta) is remain qty, I only took the entry **ledtQtyDifference = ltabSalesLine.RemainSalesPhysical;**.
For our case with inserting a sales line and cancelling it that is enough, but in other scenarions you need to differenciate between _remain invent physical_ and _remain sales physical_. Have fun :smile:

## AX2009 Job

```text
static void down1_AuftragStornieren(Args _args)
{
    //A200513157
    SalesTable  ltabSalesTable;
    SalesLine   ltabSalesLine;
    InventTable ltabInventTable;
    InventDim   ltabInventDim;
    Qty         ledtQtyDifference;
    ;

    while select forupdate ltabSalesTable where  ltabSalesTable.SalesType == SalesType::Sales && ltabSalesTable.SalesStatus == SalesStatus::Backorder{
        if(ltabSalesTable.axpController().getPositionCount() == 0 && DateTimeUtil::date(ltabSalesTable.createdDateTime) < mkdate(01,01,2021)){

            ttsbegin;
            ltabSalesLine.initValue();

            ltabSalesLine.SalesId = ltabSalesTable.SalesId;
            ltabSalesLine.initFromSalesTable(ltabSalesTable);

            ltabSalesLine.ItemId = '999999999';

            ltabInventDim.clear();
            ltabInventTable = InventTable::find(ltabSalesLine.ItemId);
            ltabInventDim.initFromInventTable(ltabInventTable);
            ltabInventDim = InventDim::findOrCreate(ltabInventDim);
            ltabSalesLine.InventDimId = ltabInventDim.inventDimId;

            ltabSalesLine.SalesQty =  1;

            ltabSalesLine.CreateLine(NoYes::Yes,NoYes::Yes,NoYes::Yes,NoYes::Yes,NoYes::Yes,NoYes::Yes,NoYes::Yes);
            ttscommit;


            while select forUpdate ltabSalesLine
           where ltabSalesLine.SalesId      == ltabSalesTable.SalesId
              && ltabSalesLine.SalesStatus  == SalesStatus::Backorder
              && ltabSalesLine.RemainSalesPhysical != 0
            {
                ledtQtyDifference = ltabSalesLine.RemainSalesPhysical;
                ltabSalesLine.RemainSalesPhysical   = 0;
                ltabSalesLine.RemainInventPhysical  = 0;

                if (ltabSalesLine.validateWrite() && ltabSalesLine.salesTable().checkUpdate())
                {
                    InterCompanyUpdateRemPhys::synchronize(ltabSalesLine,
                                                           ledtQtyDifference,
                                                           ledtQtyDifference);
                    ltabSalesLine.write();
                }
            }

        }
    }

}
```
