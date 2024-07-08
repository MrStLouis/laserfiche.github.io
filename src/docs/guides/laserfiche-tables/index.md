---
layout: default
title: Working with Lookup Tables
nav_order: 7
has_children: false
parent: Guides
---

<!--© 2024 Laserfiche.
See LICENSE-DOCUMENTATION and LICENSE-CODE in the project root for license information.-->

# Working with Lookup Tables

See the [table API overview](./../../api/odata-api-reference/) for instructions on how to configure your application to use the table API.

The Laserfiche Lookup Table API supports the OData 4 standard, which enables it to be used with popular reporting applications like PowerBI, Tableau, and Excel.  This guide will walk you through how to connect to your Lookup Tables from PowerBI and Excel, but these instructions should generally apply to most reporting tools that support connecting to data via the OData standard.


## 1. Create your Service Principal Account
Before you start, you'll want to set up a few items in Laserfiche.  To use the Laserfiche Lookup Table API with OData compliant reporting tools, you'll need to set up an app in the Developer Console using a Service Principal account.  You'll need to select an existing Service Principal account, or create a new one, and then generate a Service Principal Key (record the key string, you'll need it later).

## 2. Configure Project Security
Now that you've created or selected the Service Principal account you're going to use, you'll need to ensure that this account has access to the Lookup Tables you plan to access.  This step involves possible changes to Process Automation Project Security and you can follow the instructions in this guide on how to set that up.


## 3. Create your App in the Developer Console
Once you have your Service Principal Key String, you'll need to follow instructions here on creating an app in the Laserfiche Developer Console.  It is important to follow the instructions for the Username/Password authentication method, as that is the only authentication method that will work with most reporting tools like PowerBI, Excel, or Tableau.


## 4. Create an OData Connection to the Laserfiche API
At this point, you should be ready to set up your reporting tool of choice to access your Lookup Table data. In the steps below we will configure Excel and PowerBI, but the steps should be very similar for most other OData compliant applications.

### Excel (Microsoft Office 365)
	
1. From the "Data" tab in the Office Ribbon, select the "Get Data" button, then "From Other Sources…", then "From OData Feed".
2. In the Odata Feed window, enter the appropriate URL (see below) for your Laserfiche Cloud environment and click "OK".
			US Cloud Customers: https://api.laserfiche.com/odata4/table/
			CA Cloud Customers: https://api.laserfiche.ca/odata4/table/
			EU Cloud Customers: https://api.eu.laserfiche.com/odata4/table/
3. In the next window,  select "Basic" from the side navigation options, then enter the Username and Password you created in the Developer Console previously.  In the "Select which level to apply these settings to" drop down, you can select the URL that most closely matches the URLs in step 2 above.
4. In the Navigator window you should now see all the Lookup Tables that your app has access to view.  You can now select a table and click the "Load" button.  In our example, we will select the "BuildingPermits" table.
5. The Lookup Table rows should now be loaded into a new sheet in your Excel file.  You can work with the data within Excel to build reports and analyze. 
	
### PowerBI Desktop
	
1. In PowerBI Desktop, select "Get data" from the "Home" tab in the Office Ribbon, then select the "OData Feed" option.
2. In the Odata Feed window, enter the appropriate URL (see below) for your Laserfiche Cloud environment and click "OK".
			US Cloud Customers: https://api.laserfiche.com/odata4/table/
			CA Cloud Customers: https://api.laserfiche.ca/odata4/table/
			EU Cloud Customers: https://api.eu.laserfiche.com/odata4/table/
3. In the next window,  select "Basic" from the side navigation options, then enter the Username and Password you created in the Developer Console previously.  In the "Select which level to apply these settings to" drop down, you can select the URL that most closely matches the URLs in step 2 above.
4. In the Navigator window you should now see all the Lookup Tables that your app has access to view.  You can now select a table and click the "Load" button.  In our example, we will select the "BuildingPermits" table.
5. You should now see the schema for your Lookup Table in the Data pane.  You can now select individual columns from your table to drag onto the PowerBI canvas and build your visualizations and reports.
