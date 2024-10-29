
# Title : Uber New Work Data Analysis
Here is Detailed Project Code, data analysis steps and their Insights.

## Data_Set
* We can access all the relative dataset here: ![Data Set](/Datasets)

## Library_Used
* We have import following python library for data preprocessing, visualisation and analysis.
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
```
* We have installed some other python package to for more advanced.
```python
# Download these package if It's not already installed
!pip install chart_studio
!pip install plotly
!pip install folium
```
From these packages, we have imported other libraries as:
```python
import chart_studio.plotly as py
import plotly.graph_objs as go
import plotly.express as px

from plotly.offline import download_plotlyjs, init_notebook_mode, plot, iplot
```
## Listing_Directories_and_Files
* To list the all directory from local storage the to distinguish among the files, we have used following command-
```python
import os
os.listdir(r"Datasets")
```
## Reading_the_data_set
* To read the data in python, we used pandas library,
```python
uber_15 = pd.read_csv(r"Datasets/uber-raw-data-janjune-15_sample.csv")
```
## Initial_exploration
* We performed some initial exploration to know the brief view of data such as,

```python
print(uber_15.shape)

type(uber_15)
```

## Remove_duplicates
* Cleaned the data by removing duplicate records to ensure data accuracy
```python
uber_15.duplicated().sum()
uber_15.drop_duplicates(inplace= True)
uber_15.shape
```

## Converting_String_Dates_to_Timestamps
* To facilitate date-based analysis and calculations, we converted the string representation of dates into a more precise timestamp format.
```python
uber_15.dtypes

uber_15['Pickup_date'][0]

type(uber_15['Pickup_date'][0])

uber_15['Pickup_date']= pd.to_datetime(uber_15['Pickup_date'])

type(uber_15['Pickup_date'][0])
uber_15.dtypes
```

### Building upon our initial exploration, we will now conduct a comprehensive EDA to understand the underlying patterns and trends within the dataset.
Let's start some basic analysis-

## Analyzing_Monthly_Ride_Demand
Which month have maximum number of uber pickup?

* We'll analyze the monthly distribution of ride requests by extracting the month from the 'Pickup_date' column and visualizing it in a bar chart.
```python
uber_15['month']= uber_15['Pickup_date'].dt.month_name()

uber_15['month'].value_counts().plot(kind= 'bar')
plt.title('Total Pickups in months')
plt.figure(figsize = (8,6))
plt.savefig('MonthlyPickups.png')
plt.show()
```
![Monthly Pickup](/MonthlyPickups.png)

* **Insights** - Thus there is maximum Pickups in the month of
    - June
    - May


* To analysis the hours rush and highest weekday pickups, we extracted weekday and hours from the datetime-
```python
uber_15['weekday'] = uber_15['Pickup_date'].dt.day_name()
uber_15['day'] = uber_15['Pickup_date'].dt.day
uber_15['hour'] = uber_15['Pickup_date'].dt.hour
uber_15['minute'] = uber_15['Pickup_date'].dt.minute
```

## Visualizing_Monthly_Pickup_Trends
* To gain deeper insights into monthly pickup patterns, we generated pivot tables analyzing pickups by weekday. These insights were further visualized through charts to identify trends.
```
pivot = pd.crosstab(index= uber_15['month'], columns= uber_15['weekday'])
print(pivot)
```
* Visualisation
```python
pivot.plot( kind = 'bar',
          figsize = (12,10))
plt.savefig('Weekday-Pickup.png')
```

![Weekly Pickups](/Weekday-Pickup.png)

## Hourly_Rush
* We Visualises Hourly rush by using pointplot-
```python
summary = uber_15.groupby(['weekday', 'hour'], as_index= False).size()

plt.figure(figsize= (12,6))
sns.pointplot(x = 'hour',
             y= 'size',
             hue = 'weekday',
             data = summary)
plt.savefig('hourly-rush.png')
plt.show()
```

![Hourly rush in New-York](/hourly-rush.png)

##  Base_number_with_mostly_Active
* We performed this analysis using these lines of code and visualize them with charts:

```python
# Reading the dataset
uber_foil = pd.read_csv(r"Datasets/Uber-Jan-Feb-FOIL.csv")
uber_foil.head()

init_notebook_mode(connected= True)

px.box(x= 'dispatching_base_number', 
      y= 'active_vehicles',
      data_frame= uber_foil)
```

[Active base number boxplot](/boxplot.png)

* Also, we can perform this analysis using
```python
px.violin(x= 'dispatching_base_number', 
      y= 'active_vehicles',
      data_frame= uber_foil)
```

[Active base number violin-plot](/violin.png)

* **Insights-**
    - Thus, Base number `B02764` is mostlt active.


## Collecting_all_data
* To collect the all dataset into one file, we have performed following analysis:
```python
files = os.listdir(r"Datasets")
files

files.remove('uber-raw-data-janjune-15.csv')
files.remove( 'uber-raw-data-janjune-15_sample.csv')
files

final = pd.DataFrame()
path= r"Datasets"
for file in files:
    current_df = pd.read_csv(path+'/'+file)
    final = pd.concat([current_df, final])

```
* We also remove duplicates from out final file-
```python
final.duplicated().sum()
final.drop_duplicates(inplace= True)
```

## Rush_location
At which Location of city , we are getting rush?
* We have visualised rush location using heatmap, to look into it, we are refered to my jupyter notebook-
![Uber Data Analysis](/Uber-Data-Analysis.ipynb)

* Here is detailed python code for this-
```python
rush_uber = final.groupby(['Lat', 'Lon'], as_index= False).size()

import folium

base_map = folium.Map()
from folium.plugins import HeatMap
HeatMap(rush_uber).add_to(base_map)
```

## Perform_pair_wise_analysis:
* 
```python
final['Date/Time'] =pd.to_datetime(final['Date/Time'], format= '%m/%d/%Y %H:%M:%S')

final['day'] = final['Date/Time'].dt.day
final ['hour'] = final['Date/Time'].dt.hour

pivot = final.groupby(['day', 'hour']).size().unstack()

pivot.style.background_gradient()
```

### Automation
* 
```python
def gen_pivot_table(df, col1, col2):
    pivot = final.groupby([col1, col2]).size().unstack()
    return pivot.style.background_gradient()

# We can generate pivot for any column

gen_pivot_table(final, 'day', 'hour')
```
