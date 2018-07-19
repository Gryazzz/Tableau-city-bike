# New York Citi Bike Data Analysis

This project is dedicated to the New York Citi Bike sharing program. The data which is collected end renewed monthly, can be found here: [Citi Bike Data](https://www.citibikenyc.com/system-data).

**There are two datasets that were used in the analysis**
1. Ride Data from New Jersey (September 21 2015 - June 30 2018), a light dataset for the tests.
2. Ride Data from New York (July 20 2017 - June 30 2018), a Public version of Tableau has a limit of 15 millions rows for a single dataset, therefore this dataset has a smaller time span.

**Tableau Dashboards can be found here:**
1. New Jersey - https://public.tableau.com/profile/sofia.smirnova#!/vizhome/JC_Citi_Bike/Overview
2. New York - https://public.tableau.com/profile/sofia.smirnova#!/vizhome/NY_Citi_Bike/Overview

Both Dashboards are almost identical, except the loading speed and data by itself.

## Tableau workbook description

### Overview dashboard:

* Time period can be choosen with the range of dates.

* Any station can be chosen for a deeper analysis.

* All data below is actual for the choosen period of time and station.

* Map shows all the stations with outgoing trips.

* Overall information about total number of trips, average duration and number of unique bikes.

* % of subscribers and customers.

* 9 bikes with the longest travel time.

* Trips length divided in 5 categories.

* Month to month growth or decrease in a number of rides.

* Most and less popular time of the day.

* Age and gender distribution.

* Top and bottom 10 station for outgoing trips.

### Single bike dashboard:

* Gives an information about a particular bike.

* Map shows all the station where the bike has ever been and the number of it's rides from that stations.

* Overall information about total number of rides, duration and average trip length.

* Age distribution by age groups.

* Customer type.

* Trips by duration.

### Other sheets:

* Map with outgoing trips.

* Trips by length depending on the day and time.

* Station popularity by month.

* Top/Bottom 10 end stations.

* Rides by age by station.

* Bikes rotation by station.

* Duration by age.


## Steps

**1. Data loading and cleaning**

```python
browser = webdriver.Chrome('/usr/local/bin/chromedriver')
browser.get('https://s3.amazonaws.com/tripdata/index.html')
```
```python
#get all links
all_links = [link.get_attribute('href') for link in browser.find_elements_by_tag_name('a')]
```
```python
# extract necessary links
linksJC_17 = [link for link in all_links if 'JC' in link if '2017' in link]
linksJC_18 = [link for link in all_links if 'JC' in link if '2018' in link]
links_17 = [link for link in all_links if 'JC' not in link if '2017' in link]
links_18 = [link for link in all_links if 'JC' not in link if '2018' in link]
```
```python
# function for downloading, unziping and dataframing data from a link
def create_df(link):
    
    url = urllib.request.urlopen(link)
    output = open('temporary.zip', 'wb')    
    output.write(url.read())
    output.close()
    dataframe = pd.read_csv('temporary.zip')
    
    if (len(dataframe.columns) == 15):
        dataframe.columns = ['Trip Duration (sec)', 'Start Time', 'Stop Time', 'Start Station ID',
       'Start Station Name', 'Start Station Latitude',
       'Start Station Longitude', 'End Station ID', 'End Station Name',
       'End Station Latitude', 'End Station Longitude', 'Bike ID', 'User Type',
       'Birth_Year', 'Gender']
    else:
        dataframe.columns = ['Trip Duration (sec)', 'Start Time', 'Stop Time', 'Start Station ID',
       'Start Station Name', 'Start Station Latitude',
       'Start Station Longitude', 'End Station ID', 'End Station Name',
       'End Station Latitude', 'End Station Longitude', 'Bike ID',
       'Localized Value', 'User Type', 'Birth_Year', 'Gender']
        dataframe = dataframe.drop('Localized Value',1)
        
    print(link)
    os.remove('temporary.zip')
    
    return dataframe
```

```python
# function for cleaning and preparing df
def prepare_df(df):
    
#     drop n/a
    df = df.dropna(how='any').reset_index(drop=True)
    
#     change data types
    df['Birth_Year'] = df.Birth_Year.astype(int)
    df['Start Time'] = pd.to_datetime(df['Start Time'])
    df['Stop Time'] = pd.to_datetime(df['Stop Time'])
    
#     add Age column
    df['Age'] = 2018 - df['Birth_Year']
    
#     exclude ages > 90 years
    df = df[df['Age'] < 90]
    
    return df
```
```python
# create empty df
ny17 = pd.DataFrame()
ny18 = pd.DataFrame()
```

```python
# append to new df
for link in links_17:
    temporary_df = create_df(link)
    ny17 = ny17.append(temporary_df, ignore_index=True, sort=False)

for link in links_18:
    temporary_df = create_df(link)
    ny18 = ny18.append(temporary_df, ignore_index=True, sort=False)
```

```python
# clean and prepare df
ny17 = prepare_df(ny17)
ny18 = prepare_df(ny18)
```

**2. Visualization in Tableau**

***Conclusions***

1. The further the station is from the city center the longer the average trip duration is.

2. More than 95% of all users are annual subscribers.

3. The common rider is a man around 30 years old. Women rides are only 25% of total (in average).

4. The main usage of the bikes for all ages are the trips shorter than 10 minutes.

5. On weekend bikes are mostly used from 10 am to 6 pm, while on weekdays the rush hours are 8 am and 5-6pm.

6. From May to October there is a peak in a rides number. From December to March the rides are significantly decreasing (more than 50%).

7. The average trip on weekend is longer than on weekday.

