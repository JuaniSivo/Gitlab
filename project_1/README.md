# PDM Project

## Objectives

1. Write commented Python code;
2. Manipulate data with pandas;
3. Communicate with a database;

## Data

There are 2 datasets :
* Norway Wellbores in a SQLite database (file `wells_data.db`, table `wells_data`) ;
* Volve production in a XLSX file, with two tabs:
    * Daily Production Data ;
    * Monthly Production Data.

## Project statement

1. Open the datasets and look at it (warning! there are multi tabs in the Excel file).

2. Convert the database (file `wells_data.db`) to an XLSX and send it to a local PostgreSQL database (you can also send it to a new SQLite database if you do not have a PostgreSQL database).

3. Wells Data :
    1. Make a scatter Plot of the bottom hole temperature (vertical axis) according to the total depth (horizontal axis); try with and with out removing the 0° temperature value;
    2. Make a scatter Plot of the drilling days (vertical axis) according to the total depth (horizontal axis); use query function to keep only 0 < wlbDrillingDays < 800;
    3. Calculate the mean of the total depth according to the "Age At Td", show it as a table, sorted by mean total depth.

4. Production Data :
    1. Make a plot with the monthly production data (Oil, Gas and Water) of the `7405` NPDCode Volve according to the time. You will need to add and `monthly` column, create by concatenating the `Year` and the `Month` columns; Plot: vertical axis: the 3 productions (Oil, Gas and Water) and horizontal axis: the month and the year (ex: 2020-01).
    2. Calculate the sum of the Oil production by Volve.

5. Merged Data :
    1. Create 2 new dataframes:
        1. `partial_wells_data` from `wells_data` by keeping only `wlbTotalDepthcolumns` and the common column;
        2. `partial_mpd` from `monthly_production_data` (`mpd`) by keeping only `Oil`, `Gas` `Water` and the common column and making a sum aggregation grouping by the common column;
    2. Merge both dataframes as `merged_data`;
    3. Make a scatter plot of the pil production (vertical axis) according to the total depth (horizontal axis). Is there a correlation between the depth and the production?

## Send me

Send me a Python script: Jupyter Notebook (`.ipynb`) or standard Python file (`.py`).
