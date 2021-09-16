# GT Bootcamp Capstone Project 2 (ETL): Crime & Weather in Chicago

## Table of Contents
1. [Team Information](#team)
2. [Introduction](#introduction)
3. [Proposal](#proposal)
4. [Report](#report)  <br>
    * [Data Extraction](#extraction)<br>
    * [Data Transformation](#transformation)<br>
    * [Data Load](#load)

<a name="team"></a>
## Team Information
#### Jack Cohen
#### Karina Hutula
#### Raheem Paxton
<br>

<a name="introduction"></a>
## Introduction
For project two, we were tasked with identifying two or more sets of data to Extract (E), Transform (T), and Load (L) into a final production database.  After looking through some of the many available datasets, we decided upon utilizing Chicago Crime Data in conjunction with Historical Weather Data in order to build a database that could be used to analyze crime and weather conditions. We will utilize the principles of ETL to build our final production database, and our process is thoroughly outlined in the [Report](#report) section.

<a name="Proposal"></a>
## Proposal
* Data identification:<br>
    1. Chicago crime data: The original data was identified from the [Kaggle data set site](https://www.kaggle.com/chicago/chicago-crime).  According to the data documentation, the data was extracted from the Chicago Police Department data. For the purposes of this study, we will focus on select crimes.<br>
    2. Temperature data:  We purchased historical weather data via the [OpenWeatherMaps API](https://openweathermap.org/history-bulk) for the city of Chicago. According to the OpenWeatherMaps API documentation, the data file includes 18 different weather parameters and hourly data can be extracted, which we will use.

* Database:<br>
    We will use PostgreSQL, a relational database, for our final production database.

* Potential Use Cases:<br>
    To determine whether there is a correlation between weather and select crime data. We hypothesize that there are specific crime patterns that may be more prevalent during periods where the temperature is higher.  An interested business or resource could use weather forecast data to deploy more patrols on days when the heat index is higher.  Perhaps officers can be reduced in those areas where crime is less these days and deployed to crime pockets (hot spots) in the city. 

<a name="Report"></a>
## Report

<a name="extraction"></a>
### Data Extraction
1. Chicago crime data:<br>
    We idientified the [Chicago Crime dataset](https://www.kaggle.com/chicago/chicago-crime) on Kaggle, and used BigQuery Storage API to extract the data into a Pandas DataFrame. The data within this dataset was extracted from the Chicago Police Department's CLEAR (Citizen Law Enforcement Analysis and Reporting) system.  Privacy is respected with this dataset by showing location on block level only. Additionally, the preliminary crime classification for a record may be changed at any time due to investigation outcomes, human error, or other. The dataset is updated daily, and the data is collected from 2001 to present, minus the most recent seven days.<br><br>
    The original dataset features 22 columns:<br>
    * unique_key: a unique identifier for each record, where a record is defined as a reported incident of a crime (integer)<br>
    * case_number: Chicago Police Department Records Division Number (string)<br>
    * date: datetime of incident occurance. This may sometimes be a best estimate. Formatted as MM/DD/YYYY HH:MM:SS in 24-hour format in local Chicago time.<br>
    * block: partially redacted address (string)<br>
    * iucr: Illinois Uniform Crime Reporting code (directly linked to Primary Type and Description) (string)<br>
    * primary_type: primary description of IUCR code (string)<br>
    * description: secondary description of IUCR code (string)<br>
    * location_description: description of type of location where incident occurred (string)<br>
    * arrest: indicates whether an arrest was made (boolean)<br>
    * domestic: indicates whether the incident was domestic-related (boolean)<br>
    * beat: police beat where incident occurred (integer)<br>
    * district: police district where incident occurred (integer)<br>
    * ward: City Counsil district where incident occurred (integer)<br>
    * community_area: community area where incident occurred (integer)<br>
    * fbi_code: crime classification according to FBI NIBRS (string)<br>
    * x_coordinate: x coordinate of the location where the incident occurred in State Plane Illinois East NAD 1983 projection (integer)<br>
    * y_coordinate: y coordinate of the location where the incident occurred in State Plane Illinois East NAD 1983 projection (integer)<br>
    * year: year the incident occurred (integer)<br>
    * updated_on: datetime the record was last updated. Formatted as MM/DD/YYYY HH:MM:SS in 24-hour format in local Chicago time.<br>
    * latitude: latitude of incident location - shifted from actual location for privacy (float)<br>
    * longitude: longitude of incident location - shifted from actual location for privacy (float)<br>
    * location: latitude and longitude of incident location (tuple of floats)<br>
2. Temperature data:<br>
    In order to get thorough historical weather data to compare against our crime data, we decided to use the [History Bulk](https://openweathermap.org/history-bulk) option on OpenWeatherMap API. This option provided us with over 40 years of historical data, including 15 weather parameters recorded hourly, for only $10 per location. Since we are only looking at crime data for Chicago, we also pulled the weather data for only Chicago. After purchasing the data, we received the data as a [CSV file](Resources/chicago-hourly-weather-1980-2021.csv). We then used Pandas to read the CSV into a DataFrame.<br><br>
    The original dataset features 25 columns:<br>
    * city_name: the city name, which in this case will be Chicago for all records (string)<br>
    * lat: latitude coordinate for location, which in this case will be the same for all records (float)<br>
    * lon: longitude coordinate for location, which in this case will be the same for all records (float)<br>
    * temp: temperature (float, Celsius)<br>
    * feels_like: temperature accounting for human perception (float, Celsius)<br>
    * pressure: atmospheric pressure on sea level (integer, hPA)<br>
    * humidity: humidity (float, %)<br>
    * sea_level: atmospheric pressure on sea level (integer, hPA)<br>
    * grnd_level: atmosphereic pressure on the ground level (integer, hPa)<br>
    * temp_min: minimum temperature at the moment used to account for deviation from temperature in large cities (float, Celsius)<br>
    * temp_max: maxiumum temperature at the moment used to account for deviation from temperature in large cities (float, Celsius)<br>
    * wind_speed: wind speed (float, meter/sec)<br>
    * wind_deg: wind direction (integer, degrees)<br>
    * clouds_all: cloudiness (integer, %)<br>
    * rain_1h: rain volume for last hour (float, mm)<br>
    * rain_3h: rain volume for last 3 hours (float, mm)<br>
    * snow_1h: snow volume for last hour (float, mm)<br>
    * snow_3h: snow volume for last 3 hours (float, mm)<br>
    * weather_id: weather condition id (integer)<br>
    * weather_main: weather parameter group (string)<br>
    * weather_description: weather condition within group (string)<br>
    * weather_icon: weather icon id (string)<br>
    * dt: time of data calculation in UTC (integer)<br>
    * dt_iso: dt in date and time UTC format (string)<br>
    * timezone: shift in seconds from UTC (integer)<br>

<a name="transormation"></a>
### Data Transformation
1. Chicago crime data:<br>
    * Removed columns due to repetitiveness or not useful for analysis: case_number, block, beat, district, ward, community_area, fbi_code, x_coordinate, y_coordinate, latitude, longitude, location<br>


2. Temperature data:<br>
    * Adjusted "dt" to be in local time instead of UTC<br>
    * Created columns to reflect newly adjusted date/time: local_time, local_date, local_dt, year, TF<br>
    * Dropped weather data rows prior to 2000, since crime data begins in 2001<br>
    * Removed columns due to them being empty: rain_3h, snow_3h, sea_level, grnd_level<br>
    * Removed columns due to repetitiveness or not useful for analysis: city_name, lat, lon, weather_icon, dt, dt_iso, timezone<br>
    * Removed created columns after using: TF, year<br>
    * Filled NaN values with 0 for better analysis results: snow_1h, rain_1h<br>
    * Converted all temperatures from Celsius to Fahrenheit, since this is most commonly used in the United States: temp, feels_like, temp_min, temp_max<br>
    * Converted precipitation accumulations from metric units (mm or m/s) to imperial units (in or mph): rain_1h, snow_1h, wind_speed<br>
    * Renamed columns to include units, where applicable: temp, feels_like, temp_min, temp_max, pressure, humidity, wind_speed, rain_1h, snow_1h, clouds_all<br>
    * Set DataFrame index to "local_dt" to prepare for joining<br>

3. Joining data:<br>


<a name="load"></a>
### Data Load


![pic title](pic_link)