---
title: AX2009 Create picking list
description: How to create a picking list in AX2009/AX2012 from X++ code.
---

## Create a picking list for all positions in the sales order

With the following code sample you can create a picking list for all sales positions in the sales order.
The SalesUpdate parameter defines the selected quantities from the sales order. In the case of a picking list we want to select all quantities. For example in the case of the invoice document we want to select the SalesUpdate as PackingSlip so you only invoice positions that have been delivered.
```text
static void down1_createSalesPickingList(Args _args)
{
    SalesTable ltabSalesTable = SalesTable::find("A221624845");
    SalesFormLetter lclsSalesFormLetter;
    ;
    if(ltabSalesTable){
        lclsSalesFormLetter = SalesFormLetter::construct(DocumentStatus::PickingList);
        // SalesUpdate - All, DeliverNow, PackingSlip, PickingList
        lclsSalesFormLetter.update(ltabSalesTable, systemDateGet(), SalesUpdate::All);
    }
}
```

## Create a picking list for only certain positions

Currently I do not know how to create a picking list by code for certain sales order positions. I once saw on an another blog some aproach by modifing the salesparm table before running the SalesFormLetter class. So let's go throught that aproach.

So we create the pickling list still with the SalesFormLetter class but we performe the ***update()*** (see the previous job) methode manually and before calling ***run()***  we edit the **SalesParmLine**.
```text
static void down1_createSalesPickingListLine(Args _args)
{
    SalesTable ltabSalesTable = SalesTable::find("A221624845");
    SalesFormLetter lclsSalesFormLetter;
    SalesParmLine ltabSalesParmLine;
    InventTable ltabInventTable;
    ;
    lclsSalesFormLetter = SalesFormLetter::construct(DocumentStatus::PickingList);


    lclsSalesFormLetter.salesTable(ltabSalesTable);
    
    lclsSalesFormLetter.transDate(systemDateGet());
    lclsSalesFormLetter.specQty(SalesUpdate::All);
    lclsSalesFormLetter.proforma(lclsSalesFormLetter.salesParmUpdate().Proforma);
    lclsSalesFormLetter.printFormLetter(lclsSalesFormLetter.printFormLetter());
    lclsSalesFormLetter.printCODLabel(NoYes::No);
    lclsSalesFormLetter.printShippingLabel(NoYes::No);
    lclsSalesFormLetter.printEntryCertificate_W(NoYes::No);
    lclsSalesFormLetter.printShippingLabel(NoYes::No);
    lclsSalesFormLetter.usePrintManagement(false);
    lclsSalesFormLetter.creditRemaining(lclsSalesFormLetter.creditRemaining());
    
    lclsSalesFormLetter.createParmUpdate();

    lclsSalesFormLetter.initParameters(
        lclsSalesFormLetter.salesParmUpdate(),
        Printout::Current);

    lclsSalesFormLetter.initLinesQuery();
    
    lclsSalesFormLetter.giroType(ltabSalesTable.GiroType);

    // Delete unwanted records in SalesParmLine
    while select forupdate ltabSalesParmLine
        where ltabSalesParmLine.ParmId == lclsSalesFormLetter.parmId()
            join ltabInventTable
            where ltabInventTable.ItemId == ltabSalesParmLine.ItemId
    {
        // Select lines to delete as an example
        if(ltabInventTable.DimGroupId == "LPGS"){
            ltabSalesParmLine.delete();
        }
    }

    lclsSalesFormLetter.run();
}
```

## Create a pickling list for multiple sales orders

```text
static void down1_createSalesPickingListQuery(Args _args)
{
    SalesFormLetter lclsSalesFormLetter;
    Query   lclsQuery;
    QueryRun lclsQueryRun;
    ;
    lclsSalesFormLetter = SalesFormLetter::construct(DocumentStatus::PickingList);
    lclsQuery = new Query(QueryStr(SalesUpdate));
    lclsQuery.dataSourceTable(tablenum(SalesTable)).addRange(fieldnum(SalesTable, SalesId)).value("A221624845, A221624846, A221624847");
    lclsQueryRun = new QueryRun(lclsQuery);
    
    lclsSalesFormLetter.chooseLinesQuery(lclsQueryRun);
    
    
    lclsSalesFormLetter.transDate(systemDateGet());
    lclsSalesFormLetter.specQty(SalesUpdate::All);
    lclsSalesFormLetter.proforma(lclsSalesFormLetter.salesParmUpdate().Proforma);
    lclsSalesFormLetter.printFormLetter(lclsSalesFormLetter.printFormLetter());
    lclsSalesFormLetter.printCODLabel(NoYes::No);
    lclsSalesFormLetter.printShippingLabel(NoYes::No);
    lclsSalesFormLetter.printEntryCertificate_W(NoYes::No);
    lclsSalesFormLetter.printShippingLabel(NoYes::No);
    lclsSalesFormLetter.usePrintManagement(false);
    lclsSalesFormLetter.creditRemaining(lclsSalesFormLetter.creditRemaining());
    
    lclsSalesFormLetter.createParmUpdate();
    
    lclsSalesFormLetter.chooseLines(null, true);
    lclsSalesFormLetter.reArrangeNow(false);
    
    lclsSalesFormLetter.run();
}
```
