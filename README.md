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
The primary goal of the project was to identify two or more data sources that we would Extract (E), Transform (T), and Load (L) into a final production database.  We explored publically available datasets and chose historical Chicago Crime and weather data. The final product can be used to examine associations between criminal activity and weather conditions. We utilized the principles of ETL to deriving a final product, which was described in the [Report](#report) section.

<a name="Proposal"></a>
## Proposal
* Data identification:<br>
    1. Chicago crime data: The original data was identified from the [Kaggle data set site](https://www.kaggle.com/chicago/chicago-crime).  According to the data documentation, the data was extracted from the Chicago Police Department data. <br>
    2. Weather data:  We purchased historical weather data via the [OpenWeatherMaps API](https://openweathermap.org/history-bulk) for the city of Chicago. According to the OpenWeatherMaps API documentation, the data file includes 18 different weather parameters and hourly data can be extracted.

* Database:<br>
    We will use SQLite, a relational database, for our final production database.

* Potential Use Cases:<br>
    To determine whether there is an association between weather and select crime data. We hypothesize that specific crime patterns may be more prevalent during periods of high temperatures.  If there are positive associations between temperature and crime, data scientists could develop officer patrol algorithms to inform the Chicago Police Department and their patrol strategies.  

<a name="Report"></a>
## Report

<a name="extraction"></a>
### Data Extraction
1. Chicago crime data:<br>
    We idientified the [Chicago Crime dataset](https://www.kaggle.com/chicago/chicago-crime) on Kaggle. The data was extracted from BigQuery Storage API and transformed into a Pandas. There were several steps involved in extracting these data.  In particular, we registered for an API key, which had to be saved and stored as a file named "apikey.json" in the same folder as the Jupyter Notebook file. The original crime data were derived from the Chicago Police Department's CLEAR (Citizen Law Enforcement Analysis and Reporting) system.  The CLEAR system provides data at the block level to ensure confidentiality.   Additionally, the preliminary crime classification is subject to change as errors are corrected and the status of an investigation changes.  CLEAR system data is updated daily and available from 2001 to the prior 7days (i.e., reporting lag).<br><br>
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

2. Weather data:<br>
    To obtain historical weather data consistent with the reporting quality of the City of Chicago crime data, we utilized the [History Bulk](https://openweathermap.org/history-bulk) option on OpenWeatherMap API. The data option came with a $10 fee, which enabled us to obtain 25 different weather parameters collected hourly over the past 40 years.  Our data reported was limited to the City of Chicago. After the purchase, we received a [CSV file](Resources/chicago-hourly-weather-1980-2021.csv), which was later transformed into a Pandas DataFrame.<br><br>
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
    * Removed columns not needed for our hypothetical analysis: case_number, block, icur, beat, district, ward, community_area, fbi_code, x_coordinate, y_coordinate, latitude, longitude, location, year, updated_on<br>
    * Adjusted "date" column to match YY-MM-DD HH:MM format in order to match entries with weather data (saved as exact_dt column)<br>
    * Rounded time stamp to nearest hour in order to match entries with weather data (saved as local_dt column)<br>
    * Dropped "date" column after using<br>

2. Weather data:<br>
    * Adjusted "dt" to be in local time instead of UTC, including accounting for daylight savings time<br>
    * Created columns to reflect newly adjusted date/time: local_time, local_date, local_dt, year, TF<br>
    * Dropped weather data rows prior to 2000, since crime data begins in 2001<br>
    * Removed columns due to them being empty: rain_3h, snow_3h, sea_level, grnd_level<br>
    * Removed columns due to repetitiveness or not useful for analysis: city_name, lat, lon, weather_icon, dt, dt_iso, timezone<br>
    * Dropped created columns after using: TF, year<br>
    * Filled NaN values with 0 for better analysis results: snow_1h, rain_1h<br>
    * Converted all temperatures from Celsius to Fahrenheit, since this is most commonly used in the United States: temp, feels_like, temp_min, temp_max<br>
    * Converted precipitation accumulations from metric units (mm or m/s) to imperial units (in or mph): rain_1h, snow_1h, wind_speed<br>
    * Renamed columns to include units, where applicable: temp, feels_like, temp_min, temp_max, pressure, humidity, wind_speed, rain_1h, snow_1h, clouds_all<br>
    * Changed columns to list form in order to remove duplicate time stamps: weather_id, weather_main, weather_description<br>

3. Joining data:<br>
    * Used Pandas to left merge the crime and weather DataFrames on the "local_dt" column - The left merge option allowed us to keep all rows of crime data but drop any non-merged weather data rows.<br>
    * We retained the following columns for our final DataFrame: local_dt, exact_dt, primary_type, description, location_description, arrest, domestic, temp_F, feels_like_F, temp_min_F, temp_max_F, pressure_hPa, humidity_percent, wind_speed_mph, wind_deg, rain_1h_inches, snow_1h_inches, clouds_percent, weather_id, weather_main, weather_description<br>

<a name="load"></a>
### Data Load
We used SQLAlchemy to load our final DataFrame into an SQLite database. We decided upon uploading the data as one table instead of separate crime and weather data to provide an easier way for a client to query our database. They do not need to perform any joins to use our data. 

The class schema for our SQLite database:
```
class CW(Base):
    __tablename__ = 'crime_weather'
    id = Column(Integer, primary_key=True)
    local_dt = Column(String(255))
    exact_dt = Column(String(255))
    primary_type = Column(String(255))
    description = Column(String(255))
    location_description = Column(String(255))
    arrest = Column(Boolean)
    domestic = Column(Boolean)
    temp_F = Column(Float)
    feels_like_F = Column(Float)
    temp_min_F = Column(Float)
    temp_max_F = Column(Float)
    pressure_hPa = Column(Float)
    humidity_percent = Column(Float)
    wind_speed_mph = Column(Float)
    wind_deg = Column(Float)
    rain_1h_inches = Column(Float)
    snow_1h_inches = Column(Float)
    clouds_percent = Column(Float)
    weather_id = Column(String(255))
    weather_main = Column(String(255))
    weather_description = Column(String(255))
```

Our database is stored in <a href="https://drive.google.com/file/d/18gTA-tWKAu1Ti16dY9xdhIZKTzNTQKUi/view?usp=sharing" target="_blank">crime_weather.sqlite.zip</a> because the un-compressed verison of the file was too large to upload to GitHub. The only table in our database is named crime_weather, and it contains all columns from our final DataFrame with appropriate value types assigned.

The data is ready to be filtered, queried, and analyzed in any way the client sees fit; however, we recommend investigating associations between temperature and/or weather and crime frequency and severity.  We hypothesize a significant and positive association between specific crime patterns and higher temperatures. As discussed above, these data may one day serve as a use case for deriving weather-related police and patrol deployment algorithms.  Such efforts have the potential to minimize crime and allocate resources when they are needed most.  
