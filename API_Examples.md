Github Open Data Api Demo
================

This document contains embedded R code that demonstrates how to access
FI$Cal expenditure data located at the [California Open Data
Portal](https://data.ca.gov/) using the CKAN API.

The .Rmd version of this document can be knit within RStudio to produce
the .md version of this document (both included in this repo).

See also the documentation for the [CKAN
API](https://docs.ckan.org/en/ckan-2.7.3/api/) and the [CKAN Datastore
API](https://docs.ckan.org/en/ckan-2.7.3/maintaining/datastore.html#the-datastore-api).

## Load Necessary Packages

If you don’t have these packages installed, install them with
`install.packages("packagename")`.

``` r
library(jsonlite)  #JSON parser and generator
library(tidyverse) #a set of packages that make working in R more efficient
library(data.table) #more efficient extension of R's base data.frames
```

## Get Resource IDs

This script uses the CKAN API to get all the information about the
vendor-transactions package (i.e. dataset) and extract the IDs for each
of the resources that can be searched with the datastore API. For this
particular dataset, there is one resource (i.e. file) for each month of
data.

This is an important step, as the resulting list of resource IDs will
allow future scripts to loop through all of the resources to get
information on all months.

Note there are two FI$Cal datasets available on the Open Data Portal:

-   [Vendor
    Transactions](https://data.ca.gov/dataset/vendor-transactions) is
    the smaller dataset, containing all transactions for which there is
    an associated vendor name.
-   [Spending
    Transactions](https://data.ca.gov/dataset/spending-transactions) is
    the larger dataset, containing every expenditure transaction for the
    more than 150 departments that use FI$Cal for their accounting.

The code examples in this file use the Vendor Transactions dataset. To
use the Spending Transactions dataset, replace the URL in the following
code block with the URL in the commented line. All field names are
identical between the datasets, except that Spending Transactions does
not contain a “Vendor Name” field.

``` r
### Link to the api 
url <- "https://data.ca.gov/api/3/action/package_show?id=vendor-transactions" #specifies that the package we want information for is "vendor-transactions"

# For "spending-transactions", replace the URL with: "https://data.ca.gov/api/3/action/package_show?id=spending-transactions", then follow the same steps below

### Use fromJSON to read the text in the link
vendor <- fromJSON(url)
vendor_resources <- vendor$result$resources #extracts just the results$resources table from the response
vendor_resource_ids <- vendor_resources$resource_id[which(vendor_resources$datastore_active)] #creates a vector of just the resource IDs that have indexed, searchable data (are "datastore active") - WILL BE USED LATER
vendor_resource_ids
```

    ##  [1] "31858ad8-972e-4ac0-a31b-6b8a11687234"
    ##  [2] "dced8038-105d-40f1-a5d9-8f2ab60891f4"
    ##  [3] "6ce3d648-9e1c-496f-8457-790e539e9d8b"
    ##  [4] "e6e3396b-0dc3-48b3-8dd9-dd2016d588dc"
    ##  [5] "b38b4cfb-9f5d-4837-935a-1d2fb869f920"
    ##  [6] "74af207d-a89d-4f6f-b0c4-8e834c365983"
    ##  [7] "71b9c744-4b1f-48bc-bd40-e04ca9426722"
    ##  [8] "3b530002-ce32-4088-996c-520f540a7cac"
    ##  [9] "c4dc3ef3-c3d8-461d-976e-d3aeeffb8cf2"
    ## [10] "edc5c7f2-d2e8-4c60-8cbd-bdf1f4344ef0"
    ## [11] "d4203754-14a5-4669-96a2-9904c2149822"
    ## [12] "3f2ffdae-3118-4684-8af8-cc8bcb4ae408"
    ## [13] "f0a00ec9-41e0-4b8d-88f3-f78f51e2b64d"
    ## [14] "ef305ade-9942-4331-9683-1baea4a09e66"
    ## [15] "0faff69b-13a3-4ac0-af73-3dbbefccb1a0"
    ## [16] "4c63e1f3-b3b9-4969-ade2-00fe5b8d7c88"
    ## [17] "e72987ea-58f9-4b3c-a755-76f374164571"
    ## [18] "4686facc-80a3-487f-8187-ab774d9b2f0b"
    ## [19] "67bf91ad-d32a-4c2d-be58-9cd058aec003"
    ## [20] "2956bdb7-104a-4e76-9c38-b129eeba3702"
    ## [21] "5e8ca7c5-f707-49c3-a148-9a110fba6274"
    ## [22] "c688e42a-c0cf-4cdd-906a-f83b56bddb8b"
    ## [23] "42c17ba5-8231-4871-a029-888ae545d8b9"
    ## [24] "43e37ae0-1985-4b3b-9e20-355af9233121"
    ## [25] "84dbd7ce-70b5-4eb1-a900-903de51b86cf"
    ## [26] "1b3d8118-4e4e-4f8f-85ec-4b9ee35480f6"
    ## [27] "4a3aa423-4757-46e7-9543-f95181232646"
    ## [28] "4aafaf45-6ebc-4d9a-8998-72d2f84c0afc"
    ## [29] "bb86a946-ab69-4b4b-af7a-2fb360ba1cd8"
    ## [30] "a860b590-f15c-4435-9d95-686f78fb1907"
    ## [31] "daf4861f-51f0-4279-99ed-cb973de91ac7"
    ## [32] "780d9c8a-5505-4799-8eed-5cdc1606e0d7"
    ## [33] "709405ce-f1f7-4cc0-90c6-486b64152702"
    ## [34] "b7296592-87fa-4a38-86d0-5d5fb3996596"
    ## [35] "d3e840d3-f94c-4580-99fc-d7416607e696"
    ## [36] "7212f0b5-3154-4dd1-8ba6-4c366e1ac29e"
    ## [37] "23460e28-6a4b-419d-997e-7c51251e733e"
    ## [38] "890faf46-6596-4bc8-a5ab-f70de25f8028"
    ## [39] "f50bfac1-88b4-4a16-8c4a-769d0a6fd9e4"
    ## [40] "1ce05055-fa20-41fa-8468-e819a627cf3d"
    ## [41] "e40dee3f-de31-4547-a956-e826bb3fa9d6"
    ## [42] "7df976cb-640a-4b53-9364-1d367d9d49ab"
    ## [43] "54a5988f-ff19-4768-a8ae-53bc62733fb6"
    ## [44] "fdeea011-aca6-427b-999e-f68540c9f1f9"
    ## [45] "af8c0519-8301-47cb-9755-df90538bad6b"
    ## [46] "d81218b9-b972-43fe-8838-426bbb965621"
    ## [47] "1b14bae1-5e4f-400b-998a-af6e69bb7252"
    ## [48] "f2541d28-1cd5-4a79-9f03-de42a65db764"
    ## [49] "da7c0f99-3ef2-442f-bf2c-1404847c6155"
    ## [50] "0e62a8dc-4846-4baa-9d77-63b7cc7c3145"
    ## [51] "835350b8-0000-4b90-b1f0-ef55e5205dbe"
    ## [52] "e4bc0ae9-fff1-4fae-be3d-5f57903c45fb"
    ## [53] "0d75c95f-a48c-4e6a-8c04-b0fa8aeafeaf"
    ## [54] "5ef42696-fe74-477d-adf8-786242306b07"
    ## [55] "dd251a91-da1f-47c3-99c9-b9f5e41ca3a0"
    ## [56] "53778435-051b-4fff-a8da-627e6849e73b"
    ## [57] "791e474d-8ad1-42ca-901c-2f98f301f592"
    ## [58] "59295260-d57a-4aa8-af7d-135505d0aa90"
    ## [59] "36f2a5ec-fb4d-4141-88ad-ebb1589350df"
    ## [60] "d0003935-4980-44ed-bc4d-86eea65eafb3"
    ## [61] "b7be99fe-0727-4584-9a64-19bf34174595"
    ## [62] "9e56cd73-80a1-446b-a2eb-d27dee73c395"
    ## [63] "516e48d8-9d0a-40d5-bd90-d10a930d6c1e"
    ## [64] "53e57892-3a3b-4106-b897-093dc3de3ed3"
    ## [65] "ff8fd97c-938d-40b9-802c-e8061ad27bab"

## Code Demos Using One Resource

### Get the first 5 rows of data, from a specific resource.

``` r
#Use resource ID for FY19P01
five_rows_url <- "https://data.ca.gov/api/3/action/datastore_search?resource_id=23460e28-6a4b-419d-997e-7c51251e733e&limit=5"
five_rows_JSON <- fromJSON(five_rows_url)
five_rows_records <- five_rows_JSON$result$records
five_rows_records
```

    ##      fund_group budget_reference_category fiscal_year_begin accounting_period
    ## 1 Special Funds          State Operations              2019                 1
    ## 2  General Fund          State Operations              2019                 1
    ## 3 Special Funds          State Operations              2019                 1
    ## 4 Special Funds          State Operations              2019                 1
    ## 5 Special Funds          State Operations              2019                 1
    ##   program_code   account_sub_category business_unit
    ## 1      9990000 Other Items of Expense          3360
    ## 2      4410050        General Expense          4440
    ## 3      1150019               External          1111
    ## 4      1420049               External          1111
    ## 5      1115000               External          1111
    ##   budget_reference_sub_category accounting_date               fund_description
    ## 1                Non-Budget Act      2019-07-01  Renewable Resource Trust Fund
    ## 2                    Budget Act      2019-07-01                   General Fund
    ## 3                    Budget Act      2019-07-01  Contingent Fd Medic Brd of CA
    ## 4                    Budget Act      2019-07-01 Enhanced Fleet Modernization S
    ## 5                    Budget Act      2019-07-01 Behavioral Science Examiner Fd
    ##                    account_type budget_reference monetary_amount
    ## 1 Operating Expense & Equipment              530       1159.0000
    ## 2 Operating Expense & Equipment              011          2.0000
    ## 3 Operating Expense & Equipment              001        750.0000
    ## 4 Operating Expense & Equipment              002      10066.3500
    ## 5 Operating Expense & Equipment              001        376.0500
    ##              account_description                    agency_name
    ## 1  Other Items of Expense - Misc              Natural Resources
    ## 2     Services & Rentals - Other      Health and Human Services
    ## 3           Legal - Witness Fees Bus., Consumer Srvcs & Housing
    ## 4 Consult & Prof Svcs Extern Oth Bus., Consumer Srvcs & Housing
    ## 5 Reim Exp -Nontaxable (Non Emp) Bus., Consumer Srvcs & Housing
    ##        sub_program_description                document_id account _id
    ## 1  Unscheduled Items of Approp 3360.00016625.0.00001.0001 5390900   1
    ## 2                       Patton 4440.00080699.0.00002.0001 5301800   2
    ## 3      Medical Board - Support 1111.00178838.0.00001.0001 5340540   3
    ## 4 EFMP - Off-Cycle Vhcl Rtrmnt 1111.00179057.0.00001.0001 5340580   4
    ## 5 Board of Behavioral Sciences 1111.00178698.0.00002.0001 5340550   5
    ##                 account_category   budget_reference_description fund_code
    ## 1         Other Items of Expense Non-BA State Operations-Sup530      0382
    ## 2                General Expense BA State Operations-Support011      0001
    ## 3 Consulting & Professional Svcs BA State Operations-Support001      0758
    ## 4 Consulting & Professional Svcs BA State Operations-Support002      3122
    ## 5 Consulting & Professional Svcs BA State Operations-Support001      0773
    ##                  department_name                  vendor_name
    ## 1  Energy Resources Conservation SUNPOWER CORPORATION SYSTEMS
    ## 2  Department of State Hospitals                      US BANK
    ## 3 Department of Consumer Affairs             STEPHANIE W CHEN
    ## 4 Department of Consumer Affairs             SA RECYCLING LLC
    ## 5 Department of Consumer Affairs           KIMBERLY M JOHNSON
    ##            program_description year_of_enactment related_document
    ## 1  Unscheduled Items of Approp              1997                 
    ## 2              State Hospitals              2018                 
    ## 3  Medical Board of California              2018                 
    ## 4  Bureau of Automotive Repair              2018                 
    ## 5 Board of Behavioral Sciences              2018

### Get all records containing `jones` from a specific resource.

``` r
jones <- "https://data.ca.gov/api/3/action/datastore_search?resource_id=23460e28-6a4b-419d-997e-7c51251e733e&q=jones"
jones_JSON <- fromJSON(jones)
jones_records <- jones_JSON$result$records
head(jones_records)
```

    ##      fund_group      rank budget_reference_category fiscal_year_begin
    ## 1 Special Funds 0.0573088          State Operations              2019
    ## 2 Special Funds 0.0573088          State Operations              2019
    ## 3 Special Funds 0.0573088          State Operations              2019
    ## 4 Special Funds 0.0573088          State Operations              2019
    ## 5 Special Funds 0.0573088          State Operations              2019
    ## 6 Special Funds 0.0573088          State Operations              2019
    ##   accounting_period program_code           account_sub_category business_unit
    ## 1                 1      1215014                       External          1111
    ## 2                 1      1215014                       External          1111
    ## 3                 1      3835000 Other Special Items of Expense          4140
    ## 4                 1      3835000 Other Special Items of Expense          4140
    ## 5                 1      3835000 Other Special Items of Expense          4140
    ## 6                 1      1130010                       External          1111
    ##   budget_reference_sub_category accounting_date               fund_description
    ## 1                    Budget Act      2019-07-02 Prof Engineer Land Surv Geo FD
    ## 2                    Budget Act      2019-07-02 Prof Engineer Land Surv Geo FD
    ## 3                    Budget Act      2019-07-10 Registered Nurse Education Fun
    ## 4                    Budget Act      2019-07-10 Registered Nurse Education Fun
    ## 5                    Budget Act      2019-07-10 Registered Nurse Education Fun
    ## 6                    Budget Act      2019-07-30       Contractors License Fund
    ##                    account_type budget_reference monetary_amount
    ## 1 Operating Expense & Equipment              001       1350.0000
    ## 2 Operating Expense & Equipment              001          7.8500
    ## 3      Special Items of Expense              001       2500.0000
    ## 4      Special Items of Expense              001       2500.0000
    ## 5      Special Items of Expense              001       2500.0000
    ## 6 Operating Expense & Equipment              001         15.0800
    ##              account_description                    agency_name
    ## 1           Legal - Witness Fees Bus., Consumer Srvcs & Housing
    ## 2 Reim Exp -Nontaxable (Non Emp) Bus., Consumer Srvcs & Housing
    ## 3 Schol/Grant/Fell(Svc not Perf)      Health and Human Services
    ## 4 Schol/Grant/Fell(Svc not Perf)      Health and Human Services
    ## 5 Schol/Grant/Fell(Svc not Perf)      Health and Human Services
    ## 6 Reim Exp -Nontaxable (Non Emp) Bus., Consumer Srvcs & Housing
    ##         sub_program_description                document_id account   _id
    ## 1 Brd Prof Engnrs, Survrys, Geo 1111.00179385.0.00001.0001 5340540  4829
    ## 2 Brd Prof Engnrs, Survrys, Geo 1111.00179385.0.00002.0001 5340550  5118
    ## 3         Health Care Workforce 4140.00020668.0.00002.0001 5454000 17367
    ## 4         Health Care Workforce 4140.00020665.0.00002.0001 5454000 17459
    ## 5         Health Care Workforce 4140.00020666.0.00002.0001 5454000 17551
    ## 6 Contractors' State License Bd 1111.00184754.0.00002.0001 5340550 37516
    ##                 account_category   budget_reference_description fund_code
    ## 1 Consulting & Professional Svcs BA State Operations-Support001      0770
    ## 2 Consulting & Professional Svcs BA State Operations-Support001      0770
    ## 3 Other Special Items of Expense BA State Operations-Support001      0181
    ## 4 Other Special Items of Expense BA State Operations-Support001      0181
    ## 5 Other Special Items of Expense BA State Operations-Support001      0181
    ## 6 Consulting & Professional Svcs BA State Operations-Support001      0735
    ##                  department_name          vendor_name
    ## 1 Department of Consumer Affairs FREDERICK R JONES JR
    ## 2 Department of Consumer Affairs FREDERICK R JONES JR
    ## 3      Statewide Health Planning      CHERISE M JONES
    ## 4      Statewide Health Planning      CHERISE M JONES
    ## 5      Statewide Health Planning      CHERISE M JONES
    ## 6 Department of Consumer Affairs  BENNY JONES ROOFING
    ##              program_description year_of_enactment related_document
    ## 1 Board for Professional Enginee              2018                 
    ## 2 Board for Professional Enginee              2018                 
    ## 3          Health Care Workforce              2017                 
    ## 4          Health Care Workforce              2017                 
    ## 5          Health Care Workforce              2017                 
    ## 6 Contractors' State License Boa              2018

### Use SQL to get all account codes like `5442000` from a specific resource.

Account code 5442000 is Medical and Health Care Payments.

``` r
test_account <- "https://data.ca.gov/api/3/action/datastore_search_sql?sql=SELECT%20*%20from%20%2223460e28-6a4b-419d-997e-7c51251e733e%22%20WHERE%20%22account%22%20LIKE%20%275442000%27"
test_account_JSON <- fromJSON(test_account)
test_account_records <- test_account_JSON$result$records
head(test_account_records)
```

    ##                 fund_group budget_reference_category fiscal_year_begin
    ## 1 Other NonGovt Cost Funds          Local Assistance              2019
    ## 2 Other NonGovt Cost Funds          Local Assistance              2019
    ## 3 Other NonGovt Cost Funds          Local Assistance              2019
    ## 4 Other NonGovt Cost Funds          Local Assistance              2019
    ## 5 Other NonGovt Cost Funds          Local Assistance              2019
    ## 6 Other NonGovt Cost Funds          Local Assistance              2019
    ##   accounting_period program_code           account_sub_category business_unit
    ## 1                 1      9990000 Other Special Items of Expense          4260
    ## 2                 1      9990000 Other Special Items of Expense          4260
    ## 3                 1      9990000 Other Special Items of Expense          4260
    ## 4                 1      9990000 Other Special Items of Expense          4260
    ## 5                 1      9990000 Other Special Items of Expense          4260
    ## 6                 1      9990000 Other Special Items of Expense          4260
    ##   budget_reference_sub_category accounting_date         fund_description
    ## 1                Non-Budget Act      2019-07-17 Health Care Deposit Fund
    ## 2                Non-Budget Act      2019-07-10 Health Care Deposit Fund
    ## 3                Non-Budget Act      2019-07-10 Health Care Deposit Fund
    ## 4                Non-Budget Act      2019-07-10 Health Care Deposit Fund
    ## 5                Non-Budget Act      2019-07-10 Health Care Deposit Fund
    ## 6                Non-Budget Act      2019-07-11 Health Care Deposit Fund
    ##               account_type budget_reference monetary_amount
    ## 1 Special Items of Expense              601         19.8800
    ## 2 Special Items of Expense              601     121533.1100
    ## 3 Special Items of Expense              601     402632.5300
    ## 4 Special Items of Expense              601     417750.9900
    ## 5 Special Items of Expense              601      50220.0000
    ## 6 Special Items of Expense              601    2326742.5200
    ##              account_description               agency_name
    ## 1 Medical & Health Care Payments Health and Human Services
    ## 2 Medical & Health Care Payments Health and Human Services
    ## 3 Medical & Health Care Payments Health and Human Services
    ## 4 Medical & Health Care Payments Health and Human Services
    ## 5 Medical & Health Care Payments Health and Human Services
    ## 6 Medical & Health Care Payments Health and Human Services
    ##       sub_program_description                document_id account  _id
    ## 1 Unscheduled Items of Approp 4260.00035457.1.00001.0001 5442000 1249
    ## 2 Unscheduled Items of Approp 4260.00038326.0.00001.0001 5442000  503
    ## 3 Unscheduled Items of Approp 4260.00033896.3.00001.0001 5442000  515
    ## 4 Unscheduled Items of Approp 4260.00033916.1.00001.0001 5442000  516
    ## 5 Unscheduled Items of Approp 4260.00038296.0.00001.0001 5442000 1978
    ## 6 Unscheduled Items of Approp 4260.00035069.1.00001.0001 5442000 2079
    ##                 account_category budget_reference_description
    ## 1 Other Special Items of Expense  Non-BA Local Assistance 601
    ## 2 Other Special Items of Expense  Non-BA Local Assistance 601
    ## 3 Other Special Items of Expense  Non-BA Local Assistance 601
    ## 4 Other Special Items of Expense  Non-BA Local Assistance 601
    ## 5 Other Special Items of Expense  Non-BA Local Assistance 601
    ## 6 Other Special Items of Expense  Non-BA Local Assistance 601
    ##                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     _full_text
    ## 1 '-07':13 '-17':14 '0912':40 '1':16 '19.8800':72 '1965':71 '2019':12,15 '4260':1 '4260.00035457.1.00001.0001':11 '5442000':21 '601':58,70 '9990000':49 'act':64 'and':3 'approp':53,57 'assistance':60,69 'ba':67 'budget':63 'care':9,38,46 'community':19 'cost':43 'deposit':47 'dept':7 'expense':25,30,35 'fund':48 'funds':44 'health':2,17,37,45 'hlth':8 'human':4 'items':23,28,33,51,55 'local':59,68 'medical':36 'net':18 'non':62,66 'non-ba':65 'non-budget':61 'nongovt':42 'of':24,29,34,52,56 'other':26,31,41 'payments':39 'services':5,10 'solutions':20 'special':22,27,32 'state':6 'unscheduled':50,54
    ## 2      '-07':13 '-10':14 '0912':39 '1':16 '121533.1100':71 '1965':70 '2019':12,15 '4260':1 '4260.00038326.0.00001.0001':11 '5442000':20 '601':57,69 '9990000':48 'act':63 'and':3 'approp':52,56 'assistance':59,68 'ba':66 'budget':62 'care':9,37,45 'cost':42 'deposit':46 'dept':7 'expense':24,29,34 'fund':47 'funds':43 'hand':19 'health':2,36,44 'hlth':8 'human':4 'items':22,27,32,50,54 'local':58,67 'medical':35 'non':61,65 'non-ba':64 'non-budget':60 'nongovt':41 'of':23,28,33,51,55 'open':18 'other':25,30,40 'payments':38 'project':17 'services':5,10 'special':21,26,31 'state':6 'unscheduled':49,53
    ## 3      '-07':13 '-10':14 '0912':39 '1':16 '1965':70 '2019':12,15 '402632.5300':71 '4260':1 '4260.00033896.3.00001.0001':11 '5442000':20 '601':57,69 '9990000':48 'act':63 'and':3 'anthem':17 'approp':52,56 'assistance':59,68 'ba':66 'blue':18 'budget':62 'care':9,37,45 'cost':42 'cross':19 'deposit':46 'dept':7 'expense':24,29,34 'fund':47 'funds':43 'health':2,36,44 'hlth':8 'human':4 'items':22,27,32,50,54 'local':58,67 'medical':35 'non':61,65 'non-ba':64 'non-budget':60 'nongovt':41 'of':23,28,33,51,55 'other':25,30,40 'payments':38 'services':5,10 'special':21,26,31 'state':6 'unscheduled':49,53
    ## 4                '-07':13 '-10':14 '0912':39 '1':16 '1965':70 '2019':12,15 '417750.9900':71 '4260':1 '4260.00033916.1.00001.0001':11 '5442000':20 '601':57,69 '9990000':48 'act':63 'and':3 'approp':52,56 'assistance':59,68 'ba':66 'budget':62 'care':9,37,45 'cost':42 'deposit':46 'dept':7 'expense':24,29,34 'fund':47 'funds':43 'health':2,18,36,44 'hlth':8 'human':4 'items':22,27,32,50,54 'local':58,67 'medical':35 'non':61,65 'non-ba':64 'non-budget':60 'nongovt':41 'of':23,28,33,51,55 'other':25,30,40 'payments':38 'plan':19 'scan':17 'services':5,10 'special':21,26,31 'state':6 'unscheduled':49,53
    ## 5      '-07':13 '-10':14 '0912':40 '1':16 '1965':71 '2019':12,15 '4260':1 '4260.00038296.0.00001.0001':11 '50220.0000':72 '5442000':21 '601':58,70 '9990000':49 'act':64 'and':3 'approp':53,57 'assistance':60,69 'ba':67 'budget':63 'care':9,38,46 'corrections':19 'cost':43 'deposit':47 'dept':7,17 'expense':25,30,35 'fund':48 'funds':44 'health':2,37,45 'hlth':8 'human':4 'items':23,28,33,51,55 'local':59,68 'medical':36 'non':62,66 'non-ba':65 'non-budget':61 'nongovt':42 'of':18,24,29,34,52,56 'other':26,31,41 'payments':39 'rehab':20 'services':5,10 'special':22,27,32 'state':6 'unscheduled':50,54
    ## 6     '-07':13 '-11':14 '0912':39 '1':16 '1965':70 '2019':12,15 '2326742.5200':71 '4260':1 '4260.00035069.1.00001.0001':11 '5442000':20 '601':57,69 '9990000':48 'act':63 'and':3 'anthem':17 'approp':52,56 'assistance':59,68 'ba':66 'blue':18 'budget':62 'care':9,37,45 'cost':42 'cross':19 'deposit':46 'dept':7 'expense':24,29,34 'fund':47 'funds':43 'health':2,36,44 'hlth':8 'human':4 'items':22,27,32,50,54 'local':58,67 'medical':35 'non':61,65 'non-ba':64 'non-budget':60 'nongovt':41 'of':23,28,33,51,55 'other':25,30,40 'payments':38 'services':5,10 'special':21,26,31 'state':6 'unscheduled':49,53
    ##   fund_code               department_name                    vendor_name
    ## 1      0912 State Dept Hlth Care Services HEALTH NET COMMUNITY SOLUTIONS
    ## 2      0912 State Dept Hlth Care Services              PROJECT OPEN HAND
    ## 3      0912 State Dept Hlth Care Services              ANTHEM BLUE CROSS
    ## 4      0912 State Dept Hlth Care Services               SCAN HEALTH PLAN
    ## 5      0912 State Dept Hlth Care Services    DEPT OF CORRECTIONS & REHAB
    ## 6      0912 State Dept Hlth Care Services              ANTHEM BLUE CROSS
    ##           program_description year_of_enactment related_document
    ## 1 Unscheduled Items of Approp              1965                 
    ## 2 Unscheduled Items of Approp              1965                 
    ## 3 Unscheduled Items of Approp              1965                 
    ## 4 Unscheduled Items of Approp              1965                 
    ## 5 Unscheduled Items of Approp              1965                 
    ## 6 Unscheduled Items of Approp              1965

## Use a loop to get data from all resources

Using the `vendor_resource_ids` vector we created earlier, this script
loops through all of the resource IDs and gets information for the
specified account code for all months. This example uses Account Code
5326100, which is “electricity”.

Note that by default, a query will return no more than 100 rows.
However, the limit can be adjusted to return up to 32,000 rows per
query. This code adjusts the limit to the maximum and then checks
whether the total number of records that match the query is greater than
32,000. If it is, the code loops through the query as many times as
necessary to return all the matching records from a given resource
before moving on to the next.

``` r
FISCal_list <- vector(mode = "list", length = length(vendor_resource_ids)) #Creates a list to drop the retrieved data into; in this case, it will be a list of data frames

account_code_query <- "&q=5326100" #This is the account code for "Electricity"

count = 0

# Use a for loop to iterate through each resource in the list
for (item in vendor_resource_ids[37:length(vendor_resource_ids)]) {
  count = count + 1
  templist <- try(fromJSON(paste0("https://data.ca.gov/api/3/action/datastore_search?&resource_id=", item, "&limit=32000", account_code_query)), silent = T)
  tempDF <- templist$result$records # Creates a temporary data frame to store results
  row_total <- templist$result$total # Stores the number of records in the resource that match the query, which might be more than the 32K row max query limit
  resource_loops = 1
  #The following loop only runs if the number of matching records exceeds the 32K row max query limit, adding records until all the records that match the query are in the data frame
  while (row_total > nrow(tempDF)) {
    offset = 32000 * resource_loops
    templist <- try(fromJSON(paste0("https://data.ca.gov/api/3/action/datastore_search?&resource_id=", item, "&limit=32000", account_code_query, "&offset=", offset)), silent = T)
    tempDF <- rbind(tempDF, templist$result$records)
    resource_loops = resource_loops + 1
  }
  FISCal_list[[count]] <- tempDF #this drops the resulting data frame into the appropriate spot in the list
  
}
FISCal <- rbindlist(FISCal_list) #data.table method to quickly combine all list items into one data frame
```

## Group and graph data from previous query

``` r
options(scipen=999) #prevent scientific notation

electricity_spend <- FISCal[, c(10, 14, 16)] #extract date, agency, and monetary amount columns
electricity_spend$accounting_date <- as.Date(electricity_spend$accounting_date) #format accounting_date as date
electricity_spend$monetary_amount <- as.numeric(electricity_spend$monetary_amount) #format monetary_amount as numeric

### Summarizes dollar amount by month and agency
electricity_spend_sum <- electricity_spend %>%
  mutate(accounting_date = format(accounting_date, "%Y-%m")) %>%
  group_by(accounting_date, agency_name) %>%
  summarize(monthly_total = sum(monetary_amount), .groups = 'drop')

### Stacked bar plot by monthly monetary amount for different agencies
ggplot(data=electricity_spend_sum,
       aes(fill=agency_name, x=accounting_date, y=monthly_total))+
  geom_bar(position="stack", stat="identity")+
  theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust=1))+
  ggtitle("Electricity Spending By State Agency Over Time")+
  xlab("Accounting Date") + ylab("Monthly Expenditure ($)")
```

![](API_Examples_files/images/all_resources-1.png)<!-- -->
