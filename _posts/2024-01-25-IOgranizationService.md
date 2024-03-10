---
title: D365 F&O IOrganizationService for Dataverse
description: How to create a picking list in AX2009/AX2012 from X++ code.
---


## IOrganizationService for Dataverse connections

With F&O environments becoming linked to Power Platform environments opens us with the opportunity to interact from F&O straight with Dataverse.
Microsoft enabled us the usage of the IOrganizationService in F&O to connect to Dataverse and perform for example CRUD operations.

To be able to use that feature, we need to enable in the **PPAC (Power Platform Admin Center)** in your environment under settings -> Features the following setting:

![PPAC impersonation settings](/assets/img/FODataverseImpersonation.png)

*From March 2024 that setting should be enabled by default on all environments.*

With that we can create X++ code to interact with Dataverse. We can not only perform CRUD operations on Dataverse entities, we can check if a user has certain security roles, or if solutions/apps are installed, and we can call custom web APIs in Dataverse.
Microsoft already started using this feature in applications like the new Business Performance Analytics (BPA) preview or the CoPilot features use this connection. Microsoft made for example the SysDataverseLookup class to make it easier to create F&O lookups with Dataverse data.
Just imagine how many new possibilities emerge with the ability to connect F&O with Dataverse on the business logic level without the need to setup Dual Write. Wonderful!
 
I posted a short video demonstrating my little runnable class example on [LinkedIn](https://www.linkedin.com/posts/activity-7152688922899234816-01yO?utm_source=share&utm_medium=member_desktop).

![Example](/assets/img/IOrganizationServiceExample.png)

Link to my [Microsoft Dynamics 365 blog entry](https://community.dynamics.com/blogs/post/?postid=d5bfeb7e-99bb-ee11-92bd-000d3a01c528)

## Code example

```text
using Microsoft.Xrm.Sdk;
internal final class PDEDataverseTest
{
    Dialog dialog;
    DialogField inputField;
    DialogField lookupTest;
    IOrganizationService cds;
    private const str consumerName = 'PDE Dataverse Test';
    public static void main(Args _args)
    {
        new PDEDataverseTest().run();
    }
    
    public void run()
    {
        cds = PDEDataverseTest::getOrganizationServiceForCurrentUser(consumerName);
        dialog = new Dialog("Dataverse PDE");
        dialog.form().design().dialogSize(DialogSize::Large);
        dialog.addText("Let's test out Dataverse!");
        inputField = dialog.addField(extendedTypeStr(WebText), "Example input");
        inputField.multiLine(true);
        inputField.enabled(true);
        inputField.widthMode(240);
        lookupTest = dialog.addField(extendedTypeStr(String30), "ExampleLookup");
        lookupTest.registerOverrideMethod(methodStr(FormStringControl, lookup), methodStr(PDEDataverseTest, lookupCDS), this);
        lookupTest.enabled(true);
        
        dialog.run();
        if(dialog.closedOk())
        {
            Info(strFmt("Input: %1, Lookup: %2", inputField.value(), lookupTest.value()));
            try
            {
                
                Microsoft.Xrm.Sdk.Entity testEntity = new Entity("cr2cd_pde_fo_dataverse_test");
                Microsoft.Xrm.Sdk.AttributeCollection attr = testEntity.Attributes;
                attr.Add("cr2cd_name",inputField.value());
                attr.Add("cr2cd_secondinput","PDE TEST Dataverse Service");
                testEntity.Attributes = attr;                
                cds.Create(testEntity);
            }
            catch (Exception::CLRError)
            {
                System.Exception ex = CLRInterop::getLastException();
                error(ex.Message);
            }
        }
        
    }
    public void lookupCDS(FormStringControl _control)
    {
        SysDataverseLookup lookup = SysDataverseLookup::construct(_control, 'cr2cd_pde_fo_dataverse_test');
        
        // Set the org service explicity since implicit connects are not yet allowed
        lookup.parmOrgService(cds);
        lookup.addLookupColumn('cr2cd_name', Types::String, "@SYS7399");
        lookup.addLookupColumn('cr2cd_secondinput', Types::String, "@SYS7576");
        lookup.performLookup();
    }
    internal static IOrganizationService getOrganizationServiceForCurrentUser(str _consumerName)
    {
        if (!SysDataverseUtility::IsDataverseLinked())
        {
            throw error("Not Dataverse environment linked!");
        }
        try
        {
            var ret = SysDataverseUtility::GetOrganizationServiceForCurrentUser(_consumerName);
            if (ret == null)
            {
                throw error("Dataverse impersonation is not enabled!");
            }
            else
            {
                return ret;
            }
        }
        catch (Exception::CLRError)
        {
            throw error("Dataverse not available!");
        }
        
    }
}
```
