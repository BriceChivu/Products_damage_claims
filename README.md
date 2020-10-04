# Damage claims analysis

### 1. Introduction
With millions of units shipped every year, handling issues leading to product damages are inevitable. But can we get insights from past claims to better understand the root causes of those issues? What consignees are claiming the most? What kind of products are mostly affected? Can we improve our packaging method? <br/>
Let's try to use our past data to shine some light on this matter!

### 2. Problem statement
Our goal here is double: we want to better understand claims linked to damaged products, and we want at last to take actions to reduce those claims. <br/>
First, we will pull the data, clean it, and rearrange it. Afterwards, we will look to plot some graphs using Tableau.

### 3. Data cleaning
Let's first pull the data and clean it.
```ruby
# Importing the usual stuff

import pandas as pd
import matplotlib.pyplot as plt
%matplotlib inline

import math
import numpy as np

import datetime

pd.set_option('display.max_columns', None)
```
```ruby
#  Getting the dataframe that links every product to its description and brand

df_sku_master = pd.read_excel('MASTER_SKU_BRAND.xlsx')
df_sku_master['SKU family'] = df_sku_master['ITEM_NAME'].apply(lambda x: x[:-2])
df_sku_master.drop_duplicates(subset = 'SKU family', keep = 'first', inplace = True)
```
```ruby
# Getting the dataframe that links every product to its type

df_merch_type = pd.read_excel(r'C:\Users\btg168\Downloads\export_merch_type.xlsx')
df_merch_type['CODE_DESC'].replace({"L'Oreal LUXE Make up":"MAKE-UP","L'Oreal LUXE Skin Care":"SKIN CARE",\
                                    "L'Oreal LUXE Hair Care":"HAIR CARE","HAIR":"HAIR CARE",\
                                    "L'Oreal LUXE Miscellances":"MISCELLANEOUS"}, inplace = True)
df_merch_type['SKU family'] = df_merch_type['ITEM_NAME'].apply(lambda x: x[:-2])
df_merch_type.drop_duplicates(subset = 'SKU family', keep = 'first', inplace = True)
```
```ruby
# Taking a look at the different product type

df_merch_type['CODE_DESC'].unique()
```
<img src="https://github.com/BriceChivu/Products_damaged_claims/blob/main/df_merch_type.png" alt="alt text" width="451" height="56.5">

```ruby
# Getting the dataframe of claims 'df_raw'

df_raw = pd.read_excel(r'C:\Users\btg168\Downloads\Customer Complaint - 2020 UPDATED.xlsx')
```
```ruby
# Cleaning 'df_raw'

new_headers = df_raw.iloc[3]
df_raw = df_raw.iloc[4:]
df_raw.columns = new_headers
df_raw.columns.name = None
df_raw.dropna(thresh = 5, inplace = True)
df_raw['Complaint Date'] = pd.to_datetime(df_raw['Complaint Date'], format = '%Y/%m/%d')
df_raw.dropna(thresh = 5, axis = 1, inplace = True)
df_raw.rename(columns = {'Others\nDiscrepancies ':'category'}, inplace = True)
df_raw.replace('ySL','YSL', inplace = True)
```
```ruby
# Checking the missing values

df_raw.isna().sum()
```
<img src="https://github.com/BriceChivu/Products_damaged_claims/blob/main/df_raw.isna().sum().png" alt="alt text" width="325" height="360">

```ruby
# Checking the values of 'category'

df_raw['category'].unique()
```
<img src="https://github.com/BriceChivu/Products_damaged_claims/blob/main/df_raw['category'].unique().png" alt="alt text" width="455" height="34">

```ruby
# Creating the final dataframe that we will be using for the analysis

df = df_raw.copy()
# working only with 'DAMAGED' claims
df = df.loc[df['category'] == 'DAMAGED']
# working only with 'Complaint date' from 2020
df = df[df['Complaint Date'].dt.year == 2020]
df.reset_index(inplace = True, drop = True)
df['SKU family'] = df['SKU#'].apply(lambda x: x[:-2])
df['Total Amount'] = round(df['Total Amount'].astype(float), 1)
df.rename(columns = {df.columns[8]:'Shipment'}, inplace = True)
df['Total Amount SGD'] = df['Total Amount']
df.loc[df['Currency'] == 'THD','Total Amount SGD'] = df['Total Amount'].apply(lambda x: x*0.044)
df.loc[df['Currency'] == 'JPY','Total Amount SGD'] = df['Total Amount'].apply(lambda x: x*0.013)
df.loc[df['Currency'] == 'HKD','Total Amount SGD'] = df['Total Amount'].apply(lambda x: x*0.18)
df.loc[df['Currency'] == 'USD','Total Amount SGD'] = df['Total Amount'].apply(lambda x: x*1.37)
df = df.merge(df_merch_type, on = 'SKU family', how = 'left')
df.drop_duplicates(subset = ['Complaint Date','Loreal CS','Shipment #','DO #','SKU#','Claim Qty '],\
                   keep = 'first', inplace = True)
df = df.merge(df_sku_master[['SKU family','DESCRIPTION']].\
              drop_duplicates(subset = ['SKU family'], keep = 'first'), on = 'SKU family', how = 'left')
```
### 4. Analysis
Now that we have cleaned the data, let's see what graphs we can plot. If you have Tableau installed on your device, I suggest you download the Tableau file in my repository, you would be able to play around with the dashboards.

Otherwise, we can take a look at the 2 summary dashboards below. 

#### a. Claims value

<img src="https://github.com/BriceChivu/Products_damaged_claims/blob/main/Tableau%20damage%20claims.png" alt="alt text" width="1105" height="587">

This first dashboard is showing the claims with regard to their value.

* The first graph of the top left corner is a Treemap graph showing the distribution of the claims value per product type (Skin care, Make up, etc.) and per brand (Lancome, Kiehls, etc.)
* The second graph on the top right corner is giving information regarding the date of the claims.
* On the bottom left corner, we have the claims in regards to their orderlines. 
* Lastly, the graph at the bottom right corner is displaying the claims value per consignee.

One can easily note that 1 orderline (80202555) contributes for a huge percentage of the claims value (more than 60%). This orderline contains Lancome products and was shipped to the consignee Shilla on February 5th. It is clear that this orderline distorts the analysis. Let's try to exclude it from the graph.

#### a. bis. Claims value without 80202555

<img src="https://github.com/BriceChivu/Products_damaged_claims/blob/main/Tableau%20damage%20claims.png" alt="alt text" width="1105" height="587">
