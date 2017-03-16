
### Looking at crime data with Pandas function resample!!!!

Part of the point of this blog was learning new features and improving my skills; to accomplish this I have decided to find new functions in Pandas to play around. 

For my first edition of this I have decided to look at [Resample](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.resample.html).  Resample is used in time series analysis and is a quick and easy way for changing the frequency of your time series data  Let's dive into it! 

#### Load Libraries and Dataset 

My dataset this time is [KCPD Crime Data 2016](https://data.kcmo.org/Crime/KCPD-Crime-Data-2016/wbz8-pdv7) from the  Kansas City Open Data Portal.  This website has a very easy to use API, so I pulled the data directly from the website.


```python
import pandas as pd
from matplotlib import pyplot as plt
import numpy as np
import seaborn as sns
import matplotlib
import datetime
import urllib
from bokeh.plotting import *
from bokeh.models import HoverTool
from collections import OrderedDict
% matplotlib inline
plt.style.use('fivethirtyeight')
```


```python
query = ("https://data.kcmo.org/resource/c6e8-258d.json?$limit=500000")
df = pd.read_json(query)
```

Let's take a peak at our data


```python
# Take a peak at the data
print 'Shape'
print df.shape
print '------------------------------------------------------------------------------------------------------'
print 'dtypes'
print df.dtypes
```

    Shape
    (127877, 27)
    ------------------------------------------------------------------------------------------------------
    dtypes
    address                       object
    age                          float64
    area                          object
    beat                         float64
    city                          object
    description                   object
    dvflag                        object
    firearm_used_flag             object
    from_date                     object
    from_time                     object
    ibrs                          object
    invl_no                        int64
    involvement                   object
    location_1                    object
    location_1_address            object
    location_1_city               object
    location_1_zip               float64
    offense                        int64
    race                          object
    rep_dist                      object
    report_no                      int64
    reported_date                 object
    reported_time         datetime64[ns]
    sex                           object
    to_date                       object
    to_time                       object
    zip_code                     float64
    dtype: object



```python
df.head()
```

<img src = 'https://raw.githubusercontent.com/jherman1199/jherman1199.github.io/master/_posts/resample_sample/df_head.png' style='width: 1000px'/>


Have a big dataset with 127,877 rows and 27 columns.  To use resample it is helpful to have your date/time column as the index, so let's take a look at our date/time options to see which one to make our index.


```python
df.isnull().sum()
```




    address                  33
    age                   51974
    area                    303
    beat                    194
    city                     33
    description               0
    dvflag                    0
    firearm_used_flag         0
    from_date               233
    from_time               345
    ibrs                    784
    invl_no                   0
    involvement               0
    location_1            27201
    location_1_address       33
    location_1_city          33
    location_1_zip         4891
    offense                   0
    race                  17956
    rep_dist                303
    report_no                 0
    reported_date             0
    reported_time             0
    sex                   17956
    to_date               80660
    to_time               81177
    zip_code               4891
    dtype: int64




```python
df[['from_date', 'from_time', 'reported_date', 'reported_time']].head()
```

<img src = 'https://raw.githubusercontent.com/jherman1199/jherman1199.github.io/master/_posts/resample_sample/dates_times.png' style='width: 700px'/>


I am going to use reported_date and reported_time since they do not contain any missing values.  As we saw earlier, reported_date is currently an object and reported_time is in Pandas Date/Time form.  However, reported_time has an incorrect date (todays date) so we'll have to handle that as well. 

Let's start with reported date and convert it to a Pandas Date/Time


```python
df['reported_date'] = pd.to_datetime(df['reported_date'], format = '%Y-%m-%d')
```

Now that reported_reported time are both in Pandas Date/Time form I am going to convert them to strings, so that I can concatenate them.


```python
df['date'] = df['reported_date'].dt.strftime('%Y-%m-%d')
df['time'] = df['reported_time'].dt.strftime('%H:%M')
```

Time to concatenate and set as my index.


```python
df.set_index(pd.to_datetime(df.date + ' ' + df.time), inplace=True)
```

Take a peak, make sure everything looks as expected


```python
df.head()
```

<img src = 'https://raw.githubusercontent.com/jherman1199/jherman1199.github.io/master/_posts/resample_sample/final_head.png' style='width: 1000px'/>

We're looking good so far.  For the sake of my visualizations, I only want to look at the suspects.


```python
df_suspects = df[df['involvement'] == 'SUS']
```

Now let's break out the resample function.  I am going to break it down by quarter to start.


```python
plt.figure(figsize=(15,8))
adam_morrison = df_suspects.resample('Q')['firearm_used_flag'].count().plot(kind='line', 
                                                                           fontsize = 20)
adam_morrison.set_title('Number of Suspects by Quarter - KC 2016')
```

<img src = 'https://raw.githubusercontent.com/jherman1199/jherman1199.github.io/master/_posts/resample_sample/quarter.png' style='width: 1000px'/>


See an incline on the first 3 quarters and then a sharp decline on the last quarter.  Let's break it down by month next, see if we notice anything with this trend by braking it down smaller.


```python
plt.figure(figsize=(15,8))
linas_kleiza = df_suspects.resample('M')['firearm_used_flag'].count().plot(kind='line',
                                                                          fontsize = 20)
linas_kleiza.set_title('Number of Suspects by Month - KC 2016')
```

<img src = 'https://raw.githubusercontent.com/jherman1199/jherman1199.github.io/master/_posts/resample_sample/monthly.png' style='width: 1000px'/>

Number of suspects peaks in July and bottoms out in December.  Let's break it down by week next.


```python
plt.figure(figsize=(15,8))
kareem_rush = df_suspects.resample('W')['firearm_used_flag'].count().plot(kind='line',
                                                                         fontsize = 20)
kareem_rush.set_title('Number of Suspects` by Week - KC 2016')
```

<img src = 'https://raw.githubusercontent.com/jherman1199/jherman1199.github.io/master/_posts/resample_sample/weekly.png' style='width: 1000px'/>

This is starting to seem a little strange to me, as the monthly results show December having the least number of suspects, but the weekly results are showing different December being higher up.  Let's look at just at December to see if he have the entire month or possibly missing data.


```python
december_df = df_suspects.ix['2016-12-01':'2016-12-31']
plt.figure(figsize=(15,8))
mike_modano = december_df.resample('D')['firearm_used_flag'].count().plot(kind='line', 
                                                                         fontsize = 20)
mike_modano.set_title('Number of Suspects by Day - KC December 2016')
```

<img src = 'https://raw.githubusercontent.com/jherman1199/jherman1199.github.io/master/_posts/resample_sample/december_day.png' style='width: 1000px'/>


Looks like we only have data through 12/25, which explains the skewed results from above. 


As you can see from earlier, resample makes it extremely easy to change the frequency of your time series.  Below you will see a cheat sheet for frequency commands. 

<img src = 'https://raw.githubusercontent.com/jherman1199/jherman1199.github.io/master/_posts/resample_sample/frequency.png' style='width: 500px'/>

Next time, I will explore the KCPD Crime Data and hopefully have a full month of December to look through as well!
