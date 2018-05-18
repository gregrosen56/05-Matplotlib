
ANALYSIS

Observed Trend 1:  Intuitively, the number of rides per city decreases as an area becomes less densely populated.  This is illustrated nicely by the bubble plot, where the leftmost points are generally yellow for rural areas.  Moving right, we see the blue suburban cities, and finally the rightmost portion of the graph is filled with urban areas.

Observed Trend 2:  In general, a ride's average fare decreases the more densely populated the area.  This also makes sense intuitively, due to the presence or lack of other options for riders.

Observed Trend 3:  While rural areas made up only 1.8% of the total rides, they collected an outsize 6.6% of fares.  The cost of a ride might be a pain point for rural riders and a potential opportunity for improvement.


```python
#SCENARIO 1:  PYBER

#import necessary tools
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib
import csv
import numpy as np
import matplotlib.lines as lines
import matplotlib.axes as ax
```


```python
#read in CSV data and create data frames
city_data = "city_data.csv"
ride_data = "ride_data.csv"

df_city = pd.read_csv(city_data)
df_ride = pd.read_csv(ride_data)
df = pd.merge(df_city,df_ride,on="city",how='outer').sort_values(by="ride_id")
```


```python
#for data visualization
print(df.shape)
print(df_ride.shape)
print(df_city.shape)
df.head()
```

    (2407, 6)
    (2375, 4)
    (126, 3)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>driver_count</th>
      <th>type</th>
      <th>date</th>
      <th>fare</th>
      <th>ride_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1666</th>
      <td>West Evan</td>
      <td>4</td>
      <td>Suburban</td>
      <td>2016-02-03 15:50:57</td>
      <td>17.57</td>
      <td>2238752751</td>
    </tr>
    <tr>
      <th>1673</th>
      <td>South Gracechester</td>
      <td>19</td>
      <td>Suburban</td>
      <td>2016-09-08 06:53:25</td>
      <td>23.25</td>
      <td>7522667629</td>
    </tr>
    <tr>
      <th>2080</th>
      <td>Port Alexandria</td>
      <td>27</td>
      <td>Suburban</td>
      <td>2016-08-10 12:16:09</td>
      <td>31.75</td>
      <td>11622863980</td>
    </tr>
    <tr>
      <th>2330</th>
      <td>Kennethburgh</td>
      <td>3</td>
      <td>Rural</td>
      <td>2016-02-29 21:50:59</td>
      <td>47.48</td>
      <td>12105457917</td>
    </tr>
    <tr>
      <th>236</th>
      <td>Lisaville</td>
      <td>66</td>
      <td>Urban</td>
      <td>2016-09-30 22:29:40</td>
      <td>31.06</td>
      <td>18075235678</td>
    </tr>
  </tbody>
</table>
</div>




```python
#create data frame with average fare by city
df_average_fare = df.groupby(['city']).mean().reset_index().rename(columns={"fare": "average_fare"}).drop(['driver_count','ride_id'],axis=1)

#create data frame with total ride count by city
df_ride_count = df.groupby(['city']).count().reset_index().rename(columns={"ride_id": "ride_count"}).drop(['type','date','driver_count','fare'],axis=1)

#merge these data frames with the main data frame
df = pd.merge(df,df_average_fare,on="city",how='outer')
df = pd.merge(df,df_ride_count,on="city",how='outer')
df.head()

#create a new grouped data frame for our bubble plot
df_grouped = df.groupby(['city','type']).mean().reset_index().drop(['fare','ride_id'],axis=1)
df_grouped.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>type</th>
      <th>driver_count</th>
      <th>average_fare</th>
      <th>ride_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Alvarezhaven</td>
      <td>Urban</td>
      <td>21.0</td>
      <td>23.928710</td>
      <td>31.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Alyssaberg</td>
      <td>Urban</td>
      <td>67.0</td>
      <td>20.609615</td>
      <td>26.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Anitamouth</td>
      <td>Suburban</td>
      <td>16.0</td>
      <td>37.315556</td>
      <td>9.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Antoniomouth</td>
      <td>Urban</td>
      <td>21.0</td>
      <td>23.625000</td>
      <td>22.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Aprilchester</td>
      <td>Urban</td>
      <td>49.0</td>
      <td>21.981579</td>
      <td>19.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
#CREATE AND FORMAT THE BUBBLE PLOT

#create a color key and merge it with full data frame
new_dict = {'type': ['Urban','Suburban','Rural'], 'color': ['lightcoral','lightskyblue','gold'] }
type_colors = pd.DataFrame.from_dict(new_dict) 
df_grouped = pd.merge(df_grouped,type_colors,on="type",how='outer')

#set parameters of scatter plot
plt.scatter(x=df_grouped['ride_count'], 
            y=df_grouped['average_fare'],
            s=df_grouped['driver_count']*3, 
            c=df_grouped['color'],
            alpha = 0.8,
            edgecolors="black",
            zorder=2)

#legend stuff
line1 = lines.Line2D(range(1), range(1), linewidth=0, marker='o', markerfacecolor='lightcoral', label='Urban',alpha = 0.8)
line2 = lines.Line2D(range(1), range(1), linewidth=0, marker='o', markerfacecolor='lightskyblue', label='Suburban',alpha = 0.8)
line3 = lines.Line2D(range(1), range(1), linewidth=0, marker='o', markerfacecolor='gold', label='Rural',alpha = 0.8)
plt.legend(handles=[line1, line2, line3], title='City Types', frameon=False)

#labeling and other appearance specifications
plt.xlabel("Total Number of Rides Per City") #label x-axis
plt.ylabel("Average Fare ($)")  #label y-axis
plt.title("Pyber Ride Sharing Data (2016)")  #add title
plt.grid(color='white',zorder=1)  #white gridlines
plt.text(70, 36, "Note: \nCircle size correlates with driver count per city")  #add note on side of graph
ax = plt.gca()
ax.set_facecolor('xkcd:light grey')  #gray background

plt.show()
```


![png](output_5_0.png)



```python
#SET UP DATA FRAME FOR PIE CHARTS
df_types = df.groupby('type').sum().reset_index()
df_types
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>type</th>
      <th>driver_count</th>
      <th>fare</th>
      <th>ride_id</th>
      <th>average_fare</th>
      <th>ride_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Rural</td>
      <td>727</td>
      <td>4255.09</td>
      <td>658729360193746</td>
      <td>4255.09</td>
      <td>1015</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Suburban</td>
      <td>9730</td>
      <td>20335.69</td>
      <td>3139583688401015</td>
      <td>20335.69</td>
      <td>13475</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Urban</td>
      <td>64501</td>
      <td>40078.34</td>
      <td>7890194186030600</td>
      <td>40078.34</td>
      <td>41251</td>
    </tr>
  </tbody>
</table>
</div>




```python
#PERCENT OF TOTAL FARES BY CITY TYPE
plt.pie(df_types['fare'],
        labels = df_types['type'], 
        explode=[0,0,0.1],shadow=True, 
        colors=['gold','lightskyblue','lightcoral'],
        startangle=180,
        autopct="%1.1f%%")
plt.title("Percent of Total Fares by City Type")
plt.show()
```


![png](output_7_0.png)



```python
#PERCENT OF TOTAL RIDES BY CITY TYPE
plt.pie(df_types['ride_count'],
        labels = df_types['type'], 
        explode=[0,0,0.1],shadow=True, 
        colors=['gold','lightskyblue','lightcoral'],
        startangle=180,
        autopct="%1.1f%%")
plt.title("Percent of Total Rides by City Type")
plt.show()
```


![png](output_8_0.png)



```python
#PERCENT OF TOTAL DRIVERS BY CITY TYPE
plt.pie(df_types['driver_count'],
        labels = df_types['type'], 
        explode=[0,0,0.1],shadow=True, 
        colors=['gold','lightskyblue','lightcoral'],
        startangle=180,
        autopct="%1.1f%%")
plt.title("Percent of Total Drivers by City Type")
plt.show()
```


![png](output_9_0.png)

