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

Time series will be one of the main charts in the dashboard that I will create in **Power-BI**, therefore, I chose the column of the beginning of the fishing activity - **zvejybos_pastangos_pradzia** - to be the main time series column.

I ordered the column both in ascending and descending order to check for annomalies at the beginning and the end of the time series.

```sql
SELECT zvejybos_pastangos_pradzia FROM ZuvuKiekis_edited
ORDER BY zvejybos_pastangos_pradzia;
```
```sql
SELECT zvejybos_pastangos_pradzia FROM ZuvuKiekis_edited
ORDER BY zvejybos_pastangos_pradzia DESC;
```

I saw that there is one row with an entry from the year 1966 and two rows from the year 2205. As these entries are probably typos from the data collection stage and, in our case, nonsensical, I removed these rows.

```sql
DELETE FROM ZuvuKiekis_edited
WHERE year(zvejybos_pastangos_pradzia) = 1966;
```
```sql
DELETE FROM ZuvuKiekis_edited
WHERE year(zvejybos_pastangos_pradzia) = 2205;
```
