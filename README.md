# EXPLORATORY DATA ANALYSIS OF THE FISHING TENDENCIES BY THE LITHUANIA'S FISHERY OFFICE

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Fishing-Tendencies-Lithuania/refs/heads/main/Related%20Images/Zuvininkystes_Tendencijos_page-0001.jpg)

## GOAL

The goal of this project is to apply data analysis tools - **SQL and Power BI** - to the dataset and then to understand and visualise what fishing tendencies and trends of the Lithuania's Fishery Office can be seen from the data.

## DATA OVERVIEW

The dataset was taken from Lietuvos Atvirų Duomenų Portalas and is publicly available on https://data.gov.lt/.
Data was originally provided by Žuvininkystės tarnyba prie LR žemės ūkio ministerijos.

The table presents information on the quantities of fish caught from the Fisheries Data Information System (ŽDIS) of the Fisheries Service under the Ministry of Agriculture of the Republic of Lithuania. The data has been provided since the beginning of their collection in the system (2015).
Data has been collected since 2015.

The dataset contains 226,100 rows.

## DATA CLEANING AND PREPARATION

Firstly, I created a new database titled 'Zuvys' and imported the raw csv file into Microsoft SQL Server Management Studio as a flat file.

The raw dataset looks like this:

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Fishing-Tendencies-Lithuania/refs/heads/main/Related%20Images/raw_dataset_zuvys.png)

As good practice, I duplicated the table 'ZuvuKiekis' and called it 'ZuvuKiekis_Edited' to keep the original raw dataset in case I need to access it later on.

```sql
SELECT * INTO ZuvuKiekis_edited
FROM ZuvuKiekis;
```

Then, I started to look over the data to see what needs clean-up, standartization and etc.

After a general look through, I immediately saw that the first four columns, **fao_zonos_kodas**,**laivo_registracijos_numeris** and **kapitono_id** columns will not be relevant in our project, hence I decided to remove them from our dataset.

```sql
ALTER TABLE ZuvuKiekis_edited
DROP COLUMN type,id,revision,page_next,fao_zonos_kodas,laivo_registracijos_numeris,kapitono_id;
```

Furthermore, I checked if there are any duplicate ID - **id1** - column entries.

```sql
SELECT COUNT(*) FROM ZuvuKiekis_edited
GROUP BY id1
HAVING COUNT(*) > 1;
```

Since there were no duplicates, I moved on forward.

Time series will be one of the main charts in the dashboard that I will create in **Power BI**, therefore, I chose the column of the beginning of the fishing activity - **zvejybos_pastangos_pradzia** - to be the main time series column.

I ordered the column both in ascending and descending order to check for annomalies at the beginning and the end of the time series.

```sql
SELECT zvejybos_pastangos_pradzia FROM ZuvuKiekis_edited
ORDER BY zvejybos_pastangos_pradzia;
```

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Fishing-Tendencies-Lithuania/refs/heads/main/Related%20Images/1966%20year.png)

```sql
SELECT zvejybos_pastangos_pradzia FROM ZuvuKiekis_edited
ORDER BY zvejybos_pastangos_pradzia DESC;
```

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Fishing-Tendencies-Lithuania/refs/heads/main/Related%20Images/2205%20year.png)

I saw that there is one row with an entry from the year 1966 and two rows from the year 2205. As these entries are probably typos from the data collection stage and, in our case, nonsensical, I removed these rows.

```sql
DELETE FROM ZuvuKiekis_edited
WHERE year(zvejybos_pastangos_pradzia) = 1966;
```
```sql
DELETE FROM ZuvuKiekis_edited
WHERE year(zvejybos_pastangos_pradzia) = 2205;
```

As I will be measuring the average duration of the fishing trips, I checked the duration of the fishing trip (h) - **pastangos_trukme_val** - column and after ordering the column in ascending order, I saw that there are a couple of negative values and entries with a value of zero, which in the case of duration - does not make sense. Therefore, I removed these rows.

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Fishing-Tendencies-Lithuania/refs/heads/main/Related%20Images/negative_values.png)

```sql
DELETE FROM ZuvuKiekis_edited
WHERE pastangos_trukme_val <= 0;
```

In the fishing tool name column - **zvejybos_irankio_pav** - I saw that there were both **NULL** and **'Nežinomi arba išsamiai nenurodyti įrankiai'** entries which are overlapping in function. For the sake of standartization I replaced the **'Nežinomi arba išsamiai nenurodyti įrankiai'** values with **NULL**.

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Fishing-Tendencies-Lithuania/refs/heads/main/Related%20Images/NULL%20entries.png)

```sql
UPDATE ZuvuKiekis_edited
SET zvejybos_irankio_pav = NULL
WHERE zvejybos_irankio_pav LIKE N'Nežinomi%';
```

By checking the preliminary queries that will be used and visualized as bar charts in **Power BI**, I saw that a few of the entries are way too long in terms of character count, therefore, I shortened them but kept the original logic and meaning intact.

```sql
SELECT TOP (10) zuvies_pav_en, SUM(sugauta_kiekis)*0.001 as captured_amount_in_tonnes FROM ZuvuKiekis_edited
GROUP BY zuvies_pav_en
ORDER BY captured_amount_in_tonnes DESC;
```

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Fishing-Tendencies-Lithuania/refs/heads/main/Related%20Images/top%2010%20fish%20name%20OG.png)

```sql
SELECT TOP (10) ekonomines_zonos_pav, SUM(sugauta_kiekis)*0.001 as captured_amount_in_tonnes FROM ZuvuKiekis_edited
GROUP BY ekonomines_zonos_pav
ORDER BY captured_amount_in_tonnes DESC;
```

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Fishing-Tendencies-Lithuania/refs/heads/main/Related%20Images/top%2010%20economic%20zone%20OG.png)

```sql
SELECT TOP (10) fao_zonos_pav, SUM(sugauta_kiekis)*0.001 as captured_amount_in_tonnes FROM ZuvuKiekis_edited
GROUP BY fao_zonos_pav
ORDER BY captured_amount_in_tonnes DESC;
```

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Fishing-Tendencies-Lithuania/refs/heads/main/Related%20Images/top%2010%20fao%20zones%20OG.png)

```sql
UPDATE ZuvuKiekis_edited
SET zuvies_pav_en = 'Sardine'
WHERE zuvies_pav_en = 'European Pilchard(=Sardine)';

UPDATE ZuvuKiekis_edited
SET zuvies_pav_en = 'Blue Whiting'
WHERE zuvies_pav_en = 'Blue Whiting(=Poutassou)';

UPDATE ZuvuKiekis_edited
SET ekonomines_zonos_pav = N'Jungtinė Karalystė'
WHERE ekonomines_zonos_pav = N'Jungtinė Didžiosios Britanijos ir Šiaurės Airijos Karalystė';

UPDATE ZuvuKiekis_edited
SET fao_zonos_pav = 'South Central Baltic (East)'
WHERE fao_zonos_pav = 'Southern Central Baltic - East (Subdivision 26)';

UPDATE ZuvuKiekis_edited
SET fao_zonos_pav = 'Coast Of Scotland'
WHERE fao_zonos_pav = 'Northwest Coast Of Scotland And North Ireland Or As The West Of Scotland (Division Via)';

UPDATE ZuvuKiekis_edited
SET fao_zonos_pav = 'South Central Baltic (West)'
WHERE fao_zonos_pav = N'Southern Central Baltic Ā€“ West (Subdivision 25)';

UPDATE ZuvuKiekis_edited
SET fao_zonos_pav = 'Porcupine Bank'
WHERE fao_zonos_pav = 'Porcupine Bank - Non-Neafc Regulatory Area';
```

## EXPLORATORY DATA ANALYSIS AND VISUALIZATION

The data is now cleaned up and ready for visualization, therefore, these will be the next steps:

1. To calculate the average fishing trip duration, total fish caught in tonnes and get the most productive fishing tool using **Microsoft SQL Server**.
2. Get the Top 10 most caught fish, most productive economic zones and most productive FAO zones.
3. Import the data into Power BI via SQL Server method and build a dashboard visualizing the points above.

Using the fishing trip duration (h) column - **pastangos_trukme_val** - I calculated the average fishing trip duration (h).

```sql
SELECT AVG(pastangos_trukme_val) as average_fishing_trip_length FROM ZuvuKiekis_edited;
```

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Fishing-Tendencies-Lithuania/refs/heads/main/Related%20Images/average%20fishing%20trip%20duration.png)

Microsoft SQL Server automatically assigned the caught amount column - **sugauta_kiekis** - as **INT** column type. However, since the total sum of all caught fish will exceed the character limits of **INT** column type, I converted the column into **BIGINT**.

```sql
ALTER TABLE ZuvuKiekis_edited ALTER COLUMN sugauta_kiekis BIGINT;
```

Then I proceeded with the calculation of total amount of fish caught in tonnes using the caught amount column - **sugauta_kiekis**.

```sql
SELECT SUM(sugauta_kiekis)*0.001 as total_fish_caught_in_tonnes FROM ZuvuKiekis_edited;
```

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Fishing-Tendencies-Lithuania/refs/heads/main/Related%20Images/total_fish_caught.png)

Using the fishing tool name column - **zvejybos_irankio_pav** - I got the most productive fishing tool in terms of captured fish amount.

```sql
SELECT TOP (1)  zvejybos_irankio_pav, SUM(sugauta_kiekis)*0.001 as captured_amount_in_tonnes FROM ZuvuKiekis_edited
GROUP BY zvejybos_irankio_pav
ORDER BY captured_amount_in_tonnes DESC;
```

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Fishing-Tendencies-Lithuania/refs/heads/main/Related%20Images/most%20productive%20fishing%20tool.png)

In **Power BI** these queries were visualized as card values.

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Fishing-Tendencies-Lithuania/refs/heads/main/Related%20Images/cards_fishing.png)

Next step was to get the standartized TOP 10 queries of most caught fish, most productive economic zones, most productive FAO zones and to visualize them as bar charts in **Power BI**.

```sql
SELECT TOP (10) zuvies_pav_en, SUM(sugauta_kiekis)*0.001 as captured_amount_in_tonnes FROM ZuvuKiekis_edited
GROUP BY zuvies_pav_en
ORDER BY captured_amount_in_tonnes DESC;
```

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Fishing-Tendencies-Lithuania/refs/heads/main/Related%20Images/top%2010%20fish%20name%20STANDARTIZED.png)

```sql
SELECT TOP (10) ekonomines_zonos_pav, SUM(sugauta_kiekis)*0.001 as captured_amount_in_tonnes FROM ZuvuKiekis_edited
GROUP BY ekonomines_zonos_pav
ORDER BY captured_amount_in_tonnes DESC;
```

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Fishing-Tendencies-Lithuania/refs/heads/main/Related%20Images/top%2010%20economic%20zone%20STANDARDIZED.png)

```sql
SELECT TOP (10) fao_zonos_pav, SUM(sugauta_kiekis)*0.001 as captured_amount_in_tonnes FROM ZuvuKiekis_edited
GROUP BY fao_zonos_pav
ORDER BY captured_amount_in_tonnes DESC;
```

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Fishing-Tendencies-Lithuania/refs/heads/main/Related%20Images/top%2010%20fao%20zones%20STANDARDIZED.png)

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Fishing-Tendencies-Lithuania/refs/heads/main/Related%20Images/bar_charts_fishing.png)

After getting the wanted queries and values in SQL, the time series chart showing the caught fish amount trends over time could only be created using **Power BI**.

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Fishing-Tendencies-Lithuania/refs/heads/main/Related%20Images/captured_fish_trends.png)

The entire and complete Power-BI dashboard is below.

![alt text](https://raw.githubusercontent.com/robertasdvarionas/Fishing-Tendencies-Lithuania/refs/heads/main/Related%20Images/Zuvininkystes_Tendencijos_page-0001.jpg)


## CONCLUSION

In this project I have applied **SQL** and **Power BI** to understand and visualise what fishing tendencies or insights can be drawn from the dataset provided by Fisheries Data Information System (ŽDIS).
