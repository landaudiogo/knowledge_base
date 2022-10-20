# Overview

Currently only a single warehouse is expected in a single excel file. 

This is an incorrect assumption and we should therefore change how a single excel file generates different GCS files.

# Implementation

## ClientWarehouse class
0. download the excel file
0. convert each sheet to their expected types
	0. here we should also implement categorical types
0. determine the number of warehouses in a single excel sheet
0. create an instance of `ClientWarehouseBillingFile` for each distinct warehouse in the excel sheet.
0. Change the init function to process 3 different dataframes and filter by warehouse name.
0. store the dataframe in a client/warehouse/{outbound, returns, stock, fuel, extra_items}.csv structure

## SharepointImportShippingPrices 

0. For each file returned by ClientWarehouseBillingFile, upload to GCS.