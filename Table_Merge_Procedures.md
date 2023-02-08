# Voter Registration Task 

## Abstract

Relational database comprised of voter address data and poll precinct locations.  

## Objective

Merge tables into a single table matching addresses to polling places, and provide explanation of the process.

## Variables

![Voter_ERD](https://user-images.githubusercontent.com/112409778/217132603-db27d518-5872-476d-bf60-9b7243a7ccf2.png)

- There is curently no primary key connecting the two datasets.

## Step-By-Step 

1. **Data Collection**
2. **Data Cleaning**:
   -  Reformatting column data in the CSV Via Google Sheets
3. **Upload tables to SQL**
   - Assigning aliases to table columns in SQL
4. **Creating a primary key in the **Address** and **Precinct_Polling_List** tables**
  - Both tables have a column that contains portions of Polling Precinct information.
  - The last three digits in the *Address.Precinct_ID* and *Precinct_Polling_List.Precinct* columns are identical. 
  - To create a primary key that CONCAT state abbreviation of the *Precinct_Polling_List.State_Zip* and *Address.State* and the last three digits of the *Address.Precinct_ID* and *Precinct_Polling_List.Precinct* columns
    
    - SQL syntax to create primary key in *Address* table 
      - SQL Syntax for creating substring of *Address.Precinct_ID*
 ~~~ SQL
SELECT
  string_field_0 AS Street,string_field_1 AS Apt,
  string_field_2 AS City, 
  string_field_3 AS State, 
  string_field_4 AS Zip,
  string_field_5 AS Precinct_ID,
  SUBSTR(string_field_5,5,3) AS New --Last three digits of the Precinct_ID
FROM `my-data-project-36654.Democracy_Works_Perfromance_Task.Address`
ORDER BY Street
LIMIT 1; 
 ~~~
 
 **Result**
 |Street|Apt|City|State|Zip|Precinct_ID|New|
 |---|---|---|---|---|---|---|
 |10 RUSTIC DR|*null*|N BRUNSWICK|NJ|08902-4706|034-010|010|
 
 - CONCAT *Address.State* and *Address.New* to create primary key
 
 ~~~ SQL
SELECT
  string_field_0 AS Street,string_field_1 AS Apt,
  string_field_2 AS City, 
  string_field_3 AS State, 
  string_field_4 AS Zip,
  string_field_5 AS Precinct_ID,
  CONCAT(string_field_3,SUBSTR(string_field_5,5,3)) AS Primary
FROM `my-data-project-36654.Democracy_Works_Perfromance_Task.Address`
ORDER BY Street
LIMIT 1; 
 ~~~
 
  **Result**
 |Street|Apt|City|State|Zip|Precinct_ID|Primary|
 |---|---|---|---|---|---|---|
 |10 RUSTIC DR|*null*|N BRUNSWICK|NJ|08902-4706|034-010|NJ010|
 
  - SQL Syntax for creating primary key for *Precinct_Polling_List* table
     - Creating substring of *Precinct_Polling_List.State_Zip* to return state abbreviation
  ~~~ SQL
SELECT 
  string_field_0 AS Street, 
  string_field_1 AS City, 
  SUBSTR(string_field_2,1,2) AS State, --substring of State_Zip column
  string_field_2 AS State_Zip,
  string_field_3 AS Country,
  string_field_4 AS Precinct 
FROM `my-data-project-36654.Democracy_Works_Perfromance_Task.Poll1`
ORDER BY Street
LIMIT 1
  ~~~
  
  **Result**
  
 |Street|City|State|State_Zip|Country|Precinct|
 |---|---|---|---|---|---|
 |1007 Merchant Street|Ambridge|PA|PA 15003|USA|PEN-018|
 
 - SQL Syntax for cleaning data in the *Precinct_Polling_List.Precinct* column for NY and NJ to make state abbreviation to make character length uniform
~~~ SQL
SELECT 
  string_field_0 AS Street, 
  string_field_1 AS City, 
  SUBSTR(string_field_2,1,2) AS State, --substring of State_Zip column
  string_field_2 AS State_Zip,
  string_field_3 AS Country,
  string_field_4 AS Precinct, 
  REPLACE(string_field_4,'NEW','NW') AS Precinct_1
FROM `my-data-project-36654.Democracy_Works_Perfromance_Task.Poll1`
WHERE string_field_2 LIKE 'N%'
ORDER BY Street
LIMIT 2
~~~

**Result**

|Street|City|State|State_Zip|Country|Precinct|Precinct_1|
|---|---|---|---|---|---|---|
|114 10th Avenue|New York|NY|NY 10011|USA|NEWY-087|NWY-087|
|180 Nassau Street|Princeton|NJ|NJ 08542|USA|NEWJ-010|NWJ-010|

- CONCAT *Precinct_Polling_List.State* and *Precinct_Polling_List.Precinct_1*

~~~ SQL
SELECT 
  string_field_0 AS Street, 
  string_field_1 AS City, 
  SUBSTR(string_field_2,1,2) AS State, --substring of State_Zip column
  string_field_2 AS State_Zip,
  string_field_3 AS Country,
  string_field_4 AS Precinct, 
  REPLACE(string_field_4,'NEW','NW') AS Precinct_1,
  CONCAT(SUBSTR(string_field_2,1,2),SUBSTR(REPLACE(string_field_4,'NEW','NW'),5,3)) AS Primary
FROM `my-data-project-36654.Democracy_Works_Perfromance_Task.Poll1`
ORDER BY Street
LIMIT 1
~~~

|Street|City|State|State_Zip|Country|Precinct|Precinct_1|Primary|
|---|---|---|---|---|---|---|---|
|1007 Merchant Street|Ambridge|PA|PA 15003|USA|PEN-018|PEN-018|PA018|

5. **JOIN the *Address* and *Precinct_Polling_List* tables** using the primary key ***Primary*** 

   - The updated schema for each table is pictured below. Notice that the ***Primary*** column in each table has a one to one relationship, and will be the column used to JOIN each table.

![Untitled](https://user-images.githubusercontent.com/112409778/217402047-6a1644b2-4d6f-46a3-b586-6b69abf2e947.png)


- Wrap the individual tables in a Common Table Expression (CTE)

~~~ SQL
WITH Address AS --Creating CTE

(/*Creating Schema for Address Table*/
SELECT
  string_field_0 AS Street,
  string_field_1 AS Apt,
  string_field_2 AS City, 
  string_field_3 AS State, 
  string_field_4 AS Zip,
  string_field_5 AS Precinct_ID, 
  CONCAT(string_field_3,SUBSTR(string_field_5,5,3)) AS Primary
FROM `my-data-project-36654.Democracy_Works_Perfromance_Task.Address`),

Precinct_Polling_List AS --Second table
(/*Creating Schema for Poll Table*/
SELECT 
  string_field_0 AS Street, 
  string_field_1 AS City, 
  SUBSTR(string_field_2,1,2) AS State,
  string_field_2 AS State_Zip, 
  string_field_3 AS Country, 
  REPLACE(string_field_4,'NEW','NW') AS Precinct,--Cleaning the Precinct codes for NJ and NY
   CONCAT(SUBSTR(string_field_2,1,2),SUBSTR(REPLACE(string_field_4,'NEW','NW'),5,3)) AS Primary 
FROM `my-data-project-36654.Democracy_Works_Perfromance_Task.Poll1`)

~~~

- LEFT JOIN the two tables in the CTE using the ***Primary*** column. LEFT JOIN is preferred as any *null* values are indicative that there is no polling location in precinct associated with the address.

~~~ SQL
WITH Address AS --Creating CTE

(/*Creating Schema for Address Table*/
SELECT
  string_field_0 AS Street,
  string_field_1 AS Apt,
  string_field_2 AS City, 
  string_field_3 AS State, 
  string_field_4 AS Zip,
  string_field_5 AS Precinct_ID, 
  CONCAT(string_field_3,SUBSTR(string_field_5,5,3)) AS Primary
FROM `my-data-project-36654.Democracy_Works_Perfromance_Task.Address`),

Precinct_Polling_List AS --Second table
(/*Creating Schema for Poll Table*/
SELECT 
  string_field_0 AS Street, 
  string_field_1 AS City, 
  SUBSTR(string_field_2,1,2) AS State,
  string_field_2 AS State_Zip, 
  string_field_3 AS Country, 
  REPLACE(string_field_4,'NEW','NW') AS Precinct,--Cleaning the Precinct codes for NJ and NY
   CONCAT(SUBSTR(string_field_2,1,2),SUBSTR(REPLACE(string_field_4,'NEW','NW'),5,3)) AS Primary 
FROM `my-data-project-36654.Democracy_Works_Perfromance_Task.Poll1`)

SELECT 
  Address.Street, Address.City, Address.State, 
  Address.Precinct_ID, Precinct_Polling_List.Street AS Polling_Address, 
  Precinct_Polling_List.City AS Polling_City, Precinct_Polling_List.State AS Poll_State,
  Precinct_Polling_List.Precinct AS Polling_Precinct
FROM Address
LEFT JOIN Precinct_Polling_List
USING(Primary)
~~~

To review the results of the ***LEFT JOIN*** query click [HERE](https://docs.google.com/spreadsheets/d/1bosCrqtxfyz_oHjFv_ihfZt23CgBB0Gg2kbjPVt4P_A/edit#gid=245500414)
 
