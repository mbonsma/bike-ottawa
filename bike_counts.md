---
jupyter:
  jupytext:
    formats: ipynb,md
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.11.1
  kernelspec:
    display_name: Python 3
    language: python
    name: python3
---

# City of Ottawa bicycle count data analysis

This notebook loads the bicycle count data, does some minimal cleaning, and produces a few figures.

The data was downloaded from the [City of Ottawa open data portal](https://open.ottawa.ca/datasets/bicycle-trip-counters).

The metadata with collection information and location descriptions is [here](https://www.arcgis.com/home/item.html?id=f218592c7fe74788906cc6a0eb190af9).

```python
# load packages
import pandas as pd
import numpy as np
from matplotlib import pyplot as plt
import matplotlib.dates as mdates
import seaborn as sns
```

```python
# load the excel file using pandas
# requires the package openpyxl
xls = pd.ExcelFile('data/bike_counter.xlsx')
```

The Excel file the city provides has a different sheet for each year of data collection. We can load the entire file and then loop through each of the sheets.

```python
# the date and header format for 2010-2012 is different, will require extra parsing
# 2013 has a note at the bottom which breaks the parsing
years = np.arange(2013, 2022, 1)
dataframes = []
for year in years:
    dataframes.append(pd.read_excel(xls, str(year), header = 0))
```

## Cleaning the dates and removing annotations

Unfortunately several different date formats are used, so we have to be careful when converting the dates to a standard format.

Here, we loop through each sheet separately and parse the dates into a `datetime` object.

2014 & 2015 go okay, 2016 needs special attention, so does 2017

```python
dataframes_datetime = []

for i, df in enumerate(dataframes):
    year = years[i]
    if year == 2013: # remove note at the bottom
        df = df.iloc[:365].copy()
        df['date_dt'] = pd.to_datetime(df['Date'])
    elif year == 2016: # date format changes between March and April 2016
        df1 = df.iloc[:91].copy() # convert normally
        df2 = df.iloc[91:].copy() # use dayfirst = True
        
        df1['date_dt'] = pd.to_datetime(df1['Date'])
        df2['date_dt'] = pd.to_datetime(df2['Date'], dayfirst = True)
    
        df = pd.concat([df1, df2]) # stick them back together
        
    elif year == 2017:
        df['date_dt'] = pd.to_datetime(df['Date'], dayfirst = True)
        
    else:
        df['date_dt'] = pd.to_datetime(df['Date'])
    
    dataframes_datetime.append(df)
    
    
```

```python
# smoosh all the years together into one big table
count_data = pd.concat(dataframes_datetime)
```

```python
# take a look at the column names (these are mostly locations)
count_data.columns
```

```python
# check if the dates are in order (they are entered in order in the spreadsheet)
plt.plot(np.argsort(count_data['date_dt']))
```

Looks like we parsed the dates correctly!

```python
# plot one location to get a feel for the data
# 12a^ADAWE is the bicycle counter, 12b^ADAWE is the pedestrian counter
fig, ax = plt.subplots(figsize = (16,5))

ax.plot(count_data['date_dt'], count_data['12a^ADAWE'])

ax.xaxis.set_major_formatter(mdates.DateFormatter('%Y-%m-%d'))
ax.set_xlabel("Date")
ax.set_ylabel("Count")
```

```python
# drop columns created from text notes
count_data = count_data.drop(columns=['Unnamed: 10', 'Note:', '1. ALEX: internal Battery failed in August 2019.'])
```

```python
# check out what's left after dropping unused columns
count_data.columns
```

There are a few column names that should be grouped, like '11 OBVW' and '11^OBVW', same for 9.

```python
# reshape data to long format for faceting
count_data = pd.melt(count_data, id_vars = ['Date', 'date_dt'])
```

```python
# get month-day for overlaying plots
# use dt.strftime('%b-%d' for months in abbreviated words)
count_data['month_day'] = count_data['date_dt'].dt.strftime('%m-%d')
```

```python
# rename variable and value columns resulting from melt
count_data = count_data.rename(columns={"variable": "location", "value": "count"})
```

```python
# convert counts to numbers, make dashes into NaN
count_data['count'] = pd.to_numeric(count_data['count'], errors = 'coerce')
```

```python
# make year and month columns
count_data['year'] = count_data['date_dt'].dt.year
count_data['month'] = count_data['date_dt'].dt.month
```

## Add human-readable location names

This is copied from the [metadata](https://www.arcgis.com/home/item.html?id=f218592c7fe74788906cc6a0eb190af9).

1_ALEX: Ottawa approach to the NCC Alexandra Bridge Bikeway This counter was not operational for most of 2010 due to bridge construction

2_ORPY: NCC Ottawa River Pathway approximately 100m east of the Prince of Wales Bridge

3_COBY: NCC Eastern Canal Pathway approximately 100m north of the Corktown Bridge. WINTER counter

4_CRTZ: NCC Western Canal Pathway approximately 200m north of “The Ritz”

5_LMET Laurier Segregated Bike lane just west of Metcalfe WINTER counter

6_LLYN Laurier Segregated Bike lane just east of Lyon. WINTER counter

7_LBAY Laurier Segregated Bike lane just west of Bay. WINTER counter

8_SOMO Somerset bridge over O-Train west-bound direction only WINTER counter (best effort- see notes)

9_OYNG O-Train Pathway just north of Young Street

10_OGLD O-Train Pathway just north of Galdstone Avenue

11_OBVW O-Train Pathway just north of Bayview Station

12a_ADAWE Adàwe Crossing Bikes. WINTER counter

12b_ADAWE Adàwe Crossing Pedestrians. WINTER counter

```python
# Add human-readable location names
# nothing for it but to do this manually

locations = count_data['location'].unique()
print(locations)

# make this a dictionary so that it's stable if the column names move around at all
location_names_dict = {"1^ALEX" : "Alexandra Bridge",
                       "2^ORPY" : "ORP Prince of Wales",
                       "3^COBY" : "Canal Path East, Corktown",
                       "4^CRTZ" : "Canal Path West, Ritz",
                       "5^LMET" : "Laurier at Metcalfe",
                       "6^LLYN" : "Laurier at Lyon",
                       "7^LBAY" : "Laurier at Bay",
                       "8^SOMO" : "Somerset Bridge",
                       "9 OYNG 1" : "O-Train Path, Young",
                       "9^OYNG" : "O-Train Path, Young",
                       "10^OGLD" : "O-Train Path, Gladstone",
                       "11 OBVW" : "O-Train Path, Bayview",
                       "11^OBVW" : "O-Train Path, Bayview",
                       "Portage Bridge" : "Portage Bridge",
                       "12a^ADAWE" : "Adàwe Crossing, bikes",
                       "12b^ADAWE" : "Adàwe Crossing, pedestrians"}

# make a new column with human-readable location names
count_data["location_name"] = count_data['location'].map(location_names_dict)
```

```python
# take a look at our new column:
count_data.head()
```

## Plot a single location

As a first fun thing, it could be neat to look for a covid-related change in bike counts.
What do we expect: commuting decreased, recreational cycling increased?

```python
# look for a covid-related signal at one location for starters?
# 3_COBY: NCC Eastern Canal Pathway approximately 100m north of the Corktown Bridge. WINTER counter 
canal_path_corktown = count_data[count_data['location'] == '3^COBY']
```

```python
canal_path_grouped_by_year = canal_path_corktown.groupby('year')
```

```python
canal_path_grouped = canal_path_corktown.groupby(['year', 'month'])
```

```python
canal_path_stats = canal_path_grouped['count'].agg(['mean', 'std', 'sum'])

```

```python
fig, ax = plt.subplots(figsize = (10,5))

for group in canal_path_grouped_by_year:
    year = group[0]
    data = group[1]
    
    if year < 2018:
        continue
    
    ax.plot(data['month_day'], data['count'], label = str(year))

ax.legend(loc = 'upper right')

ax.set_xticks(['01-01', '02-01','03-01','04-01','05-01','06-01','07-01','08-01','09-01','10-01', '11-01', '12-01'])
ax.set_xlabel("Date")
ax.set_ylabel("Bike count")
```

```python
canal_path_corktown_recent = canal_path_corktown[(canal_path_corktown['year'] > 2016)]
```

```python
sns.boxplot(x='month', y='count', data=canal_path_corktown_recent, hue = 'year')
plt.savefig("bike_counts_by_month_canal_path_corktown_east.png", dpi = 150)
```

## Plot multiple locations

The following plot compares all bike counter locations for the years 2017 to 2020. 

For most locations, counts seem to be down in 2020 compared to previous years. One extreme example is LMET, the Laurier bike lane west of Metcalfe. I assume this is way down because people aren't commuting to work downtown? On the other hand, there is a noticeable 2020 uptick for 12b Adawe, which is the Adawe crossing pedestrian counter (12a Adawe is bikes). 

```python
count_data_recent = count_data[(count_data['year'] > 2016)]
```

```python
# remove OYNG, LBAY, and Portage - no data for these years
count_data_recent = count_data_recent[(count_data_recent['location'] != '6^LLYN') & 
                 (count_data_recent['location'] != '9 OYNG 1') &
                 (count_data_recent['location'] != '9^OYNG') &
                 (count_data_recent['location'] != '7^LBAY') &
                 (count_data_recent['location'] != 'Portage Bridge')]
```

```python
g = sns.catplot(x = 'month', y = 'count', col = 'location_name', col_wrap = 3, data = count_data_recent,
              hue = 'year', kind = 'bar', estimator = sum, ci = None) # each bar is the sum of all the days in the month
g.set(ylim=(None, 9.3*10**4))
g.set_titles(col_template = '{col_name}') # rename subplots, code from here: https://wckdouglas.github.io/2016/12/seaborn_annoying_title
plt.savefig("bike_counts_2017_2020_all_locations.png", dpi = 200)
```

## Reproducing the city's total count plot

The City [published a plot](https://cityofottawa.activehosted.com/index.php?action=social&chash=919d2356219c1fa0c0bd560246532c72.9030&nosocial=1) showing a percentage increase in cycling in total between 2010 and 2019. Let's try to reproduce it, at least approximately. The blog says "The analysis included annual data from 21 counters along 16 different cycling facilities in wards 11 to 18, those closest to the downtown core. The data was then analysed and weighted based on cyclist volumes and uniqueness. As shown in the chart below, the number of cycling trips along these routes increased by approximately 80% over this nine-year period." 

I have no idea was "weighted based on cyclist volumes and uniqueness" means, or why one would do that. 

```python
summer_months = [5,6,7,8,9,10]

summer_data = count_data[(count_data['month'].isin(summer_months))
                         & (count_data['location'] != '12b^ADAWE')]
```

```python
total_summer_counts = summer_data.groupby(['year', 'month'])['count'].sum().reset_index()
```

```python
total_summer_counts.head()
```

```python
fig, ax = plt.subplots()

ax.plot(total_summer_counts['count'])

skip_interval = int(len(total_summer_counts) / len(summer_months))

ax.set_xticks(total_summer_counts.index[::skip_interval])
ax.set_xticklabels(total_summer_counts['year'][::skip_interval])

ax.set_xlabel("Year")
ax.set_ylabel("Total count")
```

The City's plot had summed everything for a whole year:

```python
total_summer_counts_yearly = summer_data.groupby('year')['count'].sum().reset_index()
```

```python
total_summer_counts_yearly
```

```python
fig, ax = plt.subplots(figsize = (5,3))

ax.plot(total_summer_counts_yearly['year'], total_summer_counts_yearly['count'], marker = 'o')

ax.set_xlabel("Year")
ax.set_ylabel("Total count")
plt.title("Total counts at all locations, May-October")
plt.tight_layout()
plt.savefig("total_summer_counts.png", dpi = 150)
```

Well, I haven't included data from 2010 to 2012, and it seems like I don't have all the counters the City used, and I don't know what exactly they did in their analysis, but this does not look like their data. 


## Winter Cycling

We expect that the introduction of the Winter Cycling Network in 2015 increased the ridership on winterized routes. The locations 3 COBY, 5 LMET, 6 LLYN, 7 LBAY, 8 SOMO (cleared on a "best effort" basis), and the two 12 ADAWE routes are marked as winter counters. The metadata says that "only the counters designated as 'winter' counters have valid data after the first snow accumulation, until snow has been removed or melted away in spring."

The image below shows the City's winter-maintained network (from [GeoOttawa](https://maps.ottawa.ca/geoottawa/) with the existing cycling network layer).

It looks to me like the Somerset Bridge counter is not technically on one of the City's designated winter routes. That leaves Laurier, Adawa, and the Canal Path East at Corktown Bridge.


![title](img/WinterCyclingNetwork.jpg)

```python
# get winter routes
winter_counters = ['3^COBY', '5^LMET', '6^LLYN', '7^LBAY', '8^SOMO', '12a^ADAWE', '12b^ADAWE']
winter_months = [11,12,1,2,3,4]

winter_data = count_data[count_data['location'].isin(winter_counters)]
winter_data = winter_data[winter_data['month'].isin(winter_months)]
```

```python
winter_data.head()
```

```python
g = sns.catplot(x = 'year', y = 'count', col = 'location_name', col_wrap = 3, data = winter_data,
              hue = 'month', kind = 'bar', estimator = sum, ci = None) # each bar is the sum of all the days in the month
g.set(ylim=(None, 8*10**4))
g.set_titles(col_template = '{col_name}') # rename subplots, code from here: https://wckdouglas.github.io/2016/12/seaborn_annoying_title
```

```python
winter_data_yearly = winter_data.groupby(['year', 'location_name'])['count'].sum().reset_index()
```

```python
fig, ax = plt.subplots(figsize = (8,3.5))
g = sns.lineplot(x = 'year', y = 'count', data = winter_data,
              hue = 'location_name', marker = 'o', ci = None) # each bar is the sum of all the days in the month

plt.title("Total cycle counts, November-April")
plt.legend(bbox_to_anchor=(1.01, 1),
           borderaxespad=0)
plt.tight_layout()
plt.savefig("winter_cycling_yearly_with_pedestrians.png", dpi = 150)
```

```python
fig, ax = plt.subplots(figsize = (8,3.5))
g = sns.lineplot(x = 'year', y = 'count', data = winter_data[winter_data['location_name'] != "Adàwe Crossing, pedestrians"],
              hue = 'location_name', marker = 'o', ci = None) # each bar is the sum of all the days in the month
plt.legend(bbox_to_anchor=(1.01, 1),
           borderaxespad=0)
plt.title("Total cycle counts, November-April")
plt.tight_layout()
plt.savefig("winter_cycling_yearly.png", dpi = 150)
```
