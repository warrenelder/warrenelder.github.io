---
title: Data cleaning in Python
author: "Warren Elder"
date: 2018-01-15 09:17:15
category:
- Data science
tags:
- Data Cleaning
- Python
- Bloomberg
- Investment
clearReading: true
metaAlignment: top
coverImage: cleaning.jpg
coverMeta: in
coverSize: partial
comments: false
meta: false
actions: false
---

In this post I investigate some of the processes involved in cleaning dirty data.
<!-- more -->
In most data science projects you can expect the majority of time to be spent collecting and pre-processing a dataset before performing an analysis. During the collection and pre-processing phase, one can undertake a myriad of tasks to get a dataset into a usable format. These tasks include, but are not limited to; outlier checking, date parsing, missing value imputation, malformed or erroneous data management, normalising data types, and data structuring. In this post I wanted to discuss the process of data *tidying* and what we should expect the resulting output of *tidy* data to look like.

Data tidying is often one of the first steps in achieving the goal of *clean* data. Tidy data can be defined by three criteria discussed by *H Wickham* in *Tidy Data,* from the *Journal of Statistical Software, Vol59*;
1. Each variable forms a column.
2. Each observation forms a row.
3. Each type of observational unit forms a table.

To elucidate these points further I shall now consider tidying some messy financial data from Bloomberg that I have extracted from an Excel file. A snapshot of this data is shown below.
{% asset_img BB_data_snapshot.png Snapshot of Bloomberg sample data %}
In its original format we can identify a number of areas that require some tidying before we have met the criteria specified above. These are;
- Column headers are values, not variable names.
- Multiple variables are stored in one column.
- Variables are stored in both rows and columns.
- Multiple types of observational units are stored in the same table.
- A single observational unit is stored in multiple tables.

In general we can see that this dataset has been arranged in a landscape report format, and financial variables have been grouped together for each company. In addition, we find that values for these financial variables are indexed by *Date* in a row which is only displayed for the first company (*aapl*) group.

To rectify these issues I shall utilise the *NumPy* and *Pandas* libraries in Python, which are excellent tools with which to tidy data. The python file and Bloomberg data can be accessed directly from GitHub at *https://github.com/warrenelder/data-science-projects*

After reading the Excel file into a data frame I perform the following steps;

Step 1: Observations should be in rows and variables should be in columns.
The most complicated form of messy data occurs when variables are stored in both rows and columns, but in our case we can take a big step forward by performing a transpose of the data and then picking out data fragments.
{% codeblock %}
data_transpose = data.T
{% endcodeblock %}

Step 2: Convert to numpy object, as columns are not defined.
This simplifies the process of picking out specific company data sets using array indexing notation.
{% codeblock %}
ds = data_transpose.values
{% endcodeblock %}

Step 3: Extract variables and observations from different companies.
We observe that the data is grouped together by each company resulting in a single data type, namely the *Company* variable, being found in multiple table groupings.

Step 3.1: Select column variables and convert into a list to allow addition of *Date* and *Company* variables.
{% codeblock %}
variables = ds[1,1:8].tolist()
variables.extend(["DATE", "COMPANY"])
{% endcodeblock %}

Step 3.2: Extract *Date* variable data
{% codeblock %}
dates = ds[2:,0]
{% endcodeblock %}

Step 3.3: Extract data values for each company grouping
{% codeblock %}
g1_label = ds[1,0]
g1_data = ds[2:,1:8]

g2_label = ds[1,9]
g2_data = ds[2:,10:17]

g3_label = ds[1,18]
g3_data = ds[2:,19:26]

g4_label = ds[1,27]
g4_data = ds[2:,28:35]

g5_label = ds[1,36]
g5_data = ds[2:,37:44]
{% endcodeblock %}

Step 4: Construct data sets for each company.
Define helper functions which add date and company data to each data group, and convert groups to data frames.
{% codeblock %}
def preprocess_data_group(data, dates, company):
    N = data.shape[0]
    return np.c_[data, dates, np.repeat(company, N)]

def create_dataframe_from_data_group(data, variables):
    return pd.DataFrame(data=data, index=None, columns=[variables], dtype=None, copy=False)
{% endcodeblock %}

Step 4.1: Create data frames for each company group.
{% codeblock %}
g1_data = preprocess_data_group(g1_data, dates, g1_label)
df_g1 = create_dataframe_from_data_group(g1_data, variables)

g2_data = preprocess_data_group(g2_data, dates, g2_label)
df_g2 = create_dataframe_from_data_group(g2_data, variables)

g3_data = preprocess_data_group(g3_data, dates, g3_label)
df_g3 = create_dataframe_from_data_group(g3_data, variables)

g4_data = preprocess_data_group(g4_data, dates, g4_label)
df_g4 = create_dataframe_from_data_group(g4_data, variables)

g5_data = preprocess_data_group(g5_data, dates, g5_label)
df_g5 = create_dataframe_from_data_group(g5_data, variables)
{% endcodeblock %}

Step 4.2: Concatenate company group data frames into a single dataset.
{% codeblock %}
df = pd.concat([df_g1, df_g2, df_g3, df_g4, df_g5]).reset_index(drop=True)
{% endcodeblock %}

Step 5: Define variable types.
While this step is not strictly required to tidy the data, I like to specify the types of each variable in the dataset.
{% codeblock %}
vars_float = ["PX_LAST", "CURR_ENTP_VAL", "TRAIL_12M_EBITDA", "EBITDA_MARGIN", "PE_RATIO", "LT_DEBT_TO_COM_EQY", "DIVIDEND_YIELD"]
vars_date = ["DATE"]
vars_cat = ["COMPANY"]

df[vars_float] = df[vars_float].apply(pd.to_numeric, errors='ignore')
df[vars_date] = df[vars_date].apply(pd.to_datetime)
df[vars_cat] = df[vars_cat].apply(lambda x: x.astype('category'))
{% endcodeblock %}

Step 6: Separate multiple observations types into distinct tables.
Datasets often involve values collected at multiple levels, on different types of observational units. After examining the variables we observe a number of repeated values in *LT_DEBT_TO_COM_EQY*, *TRAIL_12M_EBITDA* and *EBITDA_MARGIN*, indicating that these variables are observed over different time intervals. We must therefore separate these data type into their own tables
{% codeblock %}
df_a = df.filter(["DATE", "COMPANY", "PX_LAST", "CURR_ENTP_VAL", "PE_RATIO", "DIVIDEND_YIELD"])
df_b = df.filter(["DATE", "COMPANY", "LT_DEBT_TO_COM_EQY"])
df_c = df.filter(["DATE", "COMPANY", "TRAIL_12M_EBITDA", "EBITDA_MARGIN"])
{% endcodeblock %}

Step 7: Save cleaned data to csv.
Finally output the data to a csv file to store for further pre-processing or analysis.
{% codeblock %}
df_a.to_csv("cleaned_data_a.csv", encoding="utf-8")
df_b.to_csv("cleaned_data_b.csv", encoding="utf-8")
df_c.to_csv("cleaned_data_c.csv", encoding="utf-8")
{% endcodeblock %}

The end result is three data tables,
- *df_a* containing variables "DATE", "COMPANY", "PX_LAST", "CURR_ENTP_VAL", "PE_RATIO", "DIVIDEND_YIELD"
- *df_b* containing variables "DATE", "COMPANY", "LT_DEBT_TO_COM_EQY"
- *df_c* containing variables "DATE", "COMPANY", "TRAIL_12M_EBITDA", "EBITDA_MARGIN"

Now lets see an example of a tidy dataset. A shortened version for the *df_a* table is shown below containing the first 11 rows;

<table><thead><tr><th class="ID-cell">ID</th><th class="DATE-cell">DATE</th><th class="COMPANY-cell">COMPANY</th><th class="PX_LAST-cell">PX_LAST</th><th class="CURR_ENTP_VAL-cell">CURR_ENTP_VAL</th><th class="PE_RATIO-cell">PE_RATIO</th><th class="DIVIDEND_YIELD-cell">DIVIDEND_YIELD</th></tr></thead><tbody><tr class="firstRow"><td class="ID-cell">0</td><td class="DATE-cell">2013-05-22</td><td class="COMPANY-cell">aapl us equity</td><td class="PX_LAST-cell">441.354</td><td class="CURR_ENTP_VAL-cell">269589.5</td><td class="PE_RATIO-cell">10.536</td><td class="DIVIDEND_YIELD-cell">0.018012999999999998</td></tr><tr><td class="ID-cell">1</td><td class="DATE-cell">2013-05-21</td><td class="COMPANY-cell">aapl us equity</td><td class="PX_LAST-cell">439.66</td><td class="CURR_ENTP_VAL-cell">267999.4375</td><td class="PE_RATIO-cell">10.4956</td><td class="DIVIDEND_YIELD-cell">0.018082</td></tr><tr><td class="ID-cell">2</td><td class="DATE-cell">2013-05-20</td><td class="COMPANY-cell">aapl us equity</td><td class="PX_LAST-cell">442.93</td><td class="CURR_ENTP_VAL-cell">271068.7813</td><td class="PE_RATIO-cell">10.5736</td><td class="DIVIDEND_YIELD-cell">0.017949000000000003</td></tr><tr><td class="ID-cell">3</td><td class="DATE-cell">2013-05-19</td><td class="COMPANY-cell">aapl us equity</td><td class="PX_LAST-cell">433.26</td><td class="CURR_ENTP_VAL-cell">261992.0625</td><td class="PE_RATIO-cell">10.3428</td><td class="DIVIDEND_YIELD-cell">0.018349</td></tr><tr><td class="ID-cell">4</td><td class="DATE-cell">2013-05-16</td><td class="COMPANY-cell">aapl us equity</td><td class="PX_LAST-cell">434.578</td><td class="CURR_ENTP_VAL-cell">263229.2188</td><td class="PE_RATIO-cell">10.3743</td><td class="DIVIDEND_YIELD-cell">0.018294</td></tr><tr><td class="ID-cell">5</td><td class="DATE-cell">2013-05-15</td><td class="COMPANY-cell">aapl us equity</td><td class="PX_LAST-cell">428.85</td><td class="CURR_ENTP_VAL-cell">257852.625</td><td class="PE_RATIO-cell">10.2375</td><td class="DIVIDEND_YIELD-cell">0.018538000000000002</td></tr><tr><td class="ID-cell">6</td><td class="DATE-cell">2013-05-14</td><td class="COMPANY-cell">aapl us equity</td><td class="PX_LAST-cell">443.86</td><td class="CURR_ENTP_VAL-cell">271941.7188</td><td class="PE_RATIO-cell">10.5958</td><td class="DIVIDEND_YIELD-cell">0.017911</td></tr><tr><td class="ID-cell">7</td><td class="DATE-cell">2013-05-13</td><td class="COMPANY-cell">aapl us equity</td><td class="PX_LAST-cell">454.74</td><td class="CURR_ENTP_VAL-cell">282154.25</td><td class="PE_RATIO-cell">10.8556</td><td class="DIVIDEND_YIELD-cell">0.017483</td></tr><tr><td class="ID-cell">8</td><td class="DATE-cell">2013-05-12</td><td class="COMPANY-cell">aapl us equity</td><td class="PX_LAST-cell">452.97</td><td class="CURR_ENTP_VAL-cell">280492.8438</td><td class="PE_RATIO-cell">10.8133</td><td class="DIVIDEND_YIELD-cell">0.017551</td></tr><tr><td class="ID-cell">9</td><td class="DATE-cell">2013-05-09</td><td class="COMPANY-cell">aapl us equity</td><td class="PX_LAST-cell">456.77</td><td class="CURR_ENTP_VAL-cell">284059.6875</td><td class="PE_RATIO-cell">10.904</td><td class="DIVIDEND_YIELD-cell">0.017405</td></tr><tr class="lastRow"><td class="ID-cell">10</td><td class="DATE-cell">2013-05-08</td><td class="COMPANY-cell">aapl us equity</td><td class="PX_LAST-cell">463.84</td><td class="CURR_ENTP_VAL-cell">290695.9375</td><td class="PE_RATIO-cell">11.0728</td><td class="DIVIDEND_YIELD-cell">0.01714</td></tr></tbody></table>
