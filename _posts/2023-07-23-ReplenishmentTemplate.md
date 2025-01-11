---
title: D365 F&O Replenishment Template Lines - set product queries by code
description: Create product queries for D365 FO WHS replenishment template lines by code
---

## Problem

One of our clients was using AX2009 for the whole company and we had to migrate as soon as possible to D365 F&O in the warehouse, because they had a strict deadline for the commissioning of a warehouse automation. The warehouse automation was developed to communicated with D365 F&O and not AX2009. We were faced with one problem for the migration of the replenishment templates. I developed in AX2009 a custom solution for the warehouse replenishment (AX2009 did not have any solution for that), and simply said the manager of the warehouse had only to fill out a table with the itemid, size (if needed) and the min/max values. But to migrate the data for AX2009 to D365 was a bit problematic. Microsoft decide to us queries in D365 for thier replenishment product selection. I see that a great idea, but for the warehousing team and the lack of time to do a complete rethinking how replenising the locations should work, we need to migrate each replenishment line from AX2009 to D365. The warehousing team needed first to seperate the products into different replenishment zones, so that they can select which zone they want to replenish today. Each replenishment zone was one replenishment template header. I imported according to the replenishment zone segmentation all the lines with the correct amounts into the system over the data administration framework.

## Solution

As the company had many product like cables where we need the inventory dimension **size** and so we needed to differentiate between products and product variants. In the import file I already set the productquerymode correctly to **item** or **variant**. And in the description field of the replenishment template line I put only the **itemid** in it if it was and item and if it was and **variant**, it put the following string in it **itemid\|sizeid**. So now my runnable class can readout if it is an item or variant and create the correct query for us.

The code can alos be run as a custom script!

```text
    /// <summary>
    /// Class entry point. The system will call this method when a designated menu
    /// is selected or when execution starts and this class is set as the startup class.
    /// </summary>
    /// <param name = "_args">The specified arguments.</param>
    /// Dominik Downarowicz - down1 - 8th May 2023
    public static void main(Args _args)
    {
        WHSReplenishmentTemplateLine ltabWHSReplenishmentTemplateLine;
        QueryRun    lclsQueryRun;
        Query       lclsQuery;
        QueryBuildDataSource lclsQueryBuildDataSource,lclsQueryBuildDataSourceDim;
        container   lconItemInfo;
        int         lintCounter = 0;
        ;

        while select forupdate ltabWHSReplenishmentTemplateLine
        {

            //Differnece between Product and ProductVariant for BaseQuery
            if(ltabWHSReplenishmentTemplateLine.ProductQueryMode == WHSProductQueryMode::Item)
            {
                if (ltabWHSReplenishmentTemplateLine.ReplenFixedOnly)
                {
                    lclsQuery = new Query(queryStr(WHSInventTableFixedLoc));
                }
                else
                {
                    lclsQuery = new Query(queryStr(WHSInventTable));
                }
            }
            else if(ltabWHSReplenishmentTemplateLine.ProductQueryMode == WHSProductQueryMode::Variant)
            {
                if (ltabWHSReplenishmentTemplateLine.ReplenFixedOnly)
                {
                    lclsQuery = new Query(queryStr(WHSReleasedProductVariantsFixedLoc));
                }
                else
                {
                    lclsQuery = new Query(queryStr(WHSReleasedProductVariants));
                }
            }

            if(ltabWHSReplenishmentTemplateLine.ProductQueryMode == WHSProductQueryMode::Item)
            {
                lclsQueryBuildDataSource = lclsQuery.dataSourceTable(tableNum(InventTable));
                lclsQueryBuildDataSource.addRange(fieldNum(InventTable, ItemId)).value(ltabWHSReplenishmentTemplateLine.Description);
            }
            else if(ltabWHSReplenishmentTemplateLine.ProductQueryMode == WHSProductQueryMode::Variant)
            {

                //Split Description Line (ITEMID|SIZEID)
                if(strContains(ltabWHSReplenishmentTemplateLine.Description, '|'))
                {

                    lconItemInfo = str2con_RU(ltabWHSReplenishmentTemplateLine.Description, '|');


                    lclsQueryBuildDataSource = lclsQuery.dataSourceTable(tableNum(InventDimCombination));

                    lclsQueryBuildDataSource.addRange(fieldNum(InventDimCombination, ItemId)).value(conPeek(lconItemInfo, 1));

                    lclsQueryBuildDataSourceDim = lclsQueryBuildDataSource.addDataSource(tableNum(InventDim));
                    lclsQueryBuildDataSourceDim.relations(true);
                    lclsQueryBuildDataSourceDim.joinMode(JoinMode::ExistsJoin);
                    lclsQueryBuildDataSourceDim.addRange(fieldNum(InventDim, InventSizeId)).value(queryValue(conPeek(lconItemInfo, 2)));

                }
                else
                {
                    warning(strFmt("The Replenishment Template Line with the description %1 has an incorrect description", ltabWHSReplenishmentTemplateLine.Description));
                }

            }


            lclsQueryRun = new QueryRun(lclsQuery);


            //Differnece between Product and ProductVariant for QueryName
            if(ltabWHSReplenishmentTemplateLine.ProductQueryMode == WHSProductQueryMode::Item)
            {
                if (ltabWHSReplenishmentTemplateLine.ReplenFixedOnly)
                {
                    lclsQueryRun.name("@WAX:ProductQueryFixedLocations");
                }
                else
                {
                    lclsQueryRun.name("@SYP4980032");
                }
            }
            else if(ltabWHSReplenishmentTemplateLine.ProductQueryMode == WHSProductQueryMode::Variant)
            {
                if (ltabWHSReplenishmentTemplateLine.ReplenFixedOnly)
                {
                    lclsQueryRun.name("@WAX:ProductVariantQueryFixedLocations");
                }
                else
                {
                    lclsQueryRun.name("@SYP4980030");
                }
            }

            lclsQueryRun.saveUserSetup(false);

            ttsbegin;

            if(ltabWHSReplenishmentTemplateLine.ProductQueryMode == WHSProductQueryMode::Item)
            {
                ltabWHSReplenishmentTemplateLine.ItemQuery = lclsQueryRun.pack();
            }
            else if(ltabWHSReplenishmentTemplateLine.ProductQueryMode == WHSProductQueryMode::Variant)
            {
                ltabWHSReplenishmentTemplateLine.ProductVariantQuery = lclsQueryRun.pack();
            }
            ltabWHSReplenishmentTemplateLine.update();
            lintCounter++;
            ttscommit;
        }

        info(strfmt("Completed: Processed %1 Replenishment Template Lines", lintCounter));
    }
```
