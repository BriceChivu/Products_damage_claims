# Damage claims analysis

### 1. Introduction
With millions of units shipped every year, handling issues leading to product damages are inevitable. But can we get insights from past claims to better understand the root causes of those issues? What consignees are claiming the most? What kind of products are mostly affected? Can we improve our packaging method? <br/>
Let's try to use our past data to shine some light on this matter!

### 2. Problem statement
Our goal here is double: we want to better understand claims linked to damaged products, and we want at last to take actions to reduce those claims. <br/>
First, we will pull the data, clean it, and rearrange it. Afterwards, we will look to plot some graphs using Tableau.

### 3. Data cleaning
Let's first pull the data.
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
