# NFT-Project
NFT Data Analysis

The project consists of business questions (in an Excel spreadsheet), SQL queries (used to create views), Power BI dashboards, and a presentation.

## SQL Queries

### 1. Mints Full
The query used for cleaning the Mints table:

```ruby
create or replace view NFT_OPEN_SEA.PUBLIC.MINTS_FULL(
	EVENT_ID,
	TRANSACTION_HASH,
	BLOCK_NUMBER,
	NFT_ADDRESS,
	TOKEN_ID,
	FROM_ADDRESS,
	TO_ADDRESS,
	TRANSACTION_VALUE,
	Z_DATE,
	Z_TIME,
	Z_MONTH,
	Z_WEEK,
	Z_DOW
) as
select 
    EVENT_ID,
    TRANSACTION_HASH,
    BLOCK_NUMBER,
    NFT_ADDRESS,
    TOKEN_ID,
    FROM_ADDRESS,
    TO_ADDRESS,
    TRANSACTION_VALUE,
    to_date(TIMESTAMP) as z_date,
    to_time(TIMESTAMP) as z_time,
    MONTH( TIMESTAMP ) as z_month,
    WEEKOFYEAR( TIMESTAMP ) as z_week,
    DAYOFWEEK( TIMESTAMP ) as z_dow
    
FROM NFT_OPEN_SEA.RAW.MINTS;
```

### 2. Transfers Full
The query used for cleaning the Transfers table:

```ruby
create or replace view NFT_OPEN_SEA.PUBLIC.TRANSFERS_FULL(
	EVENT_ID,
	TRANSACTION_HASH,
	BLOCK_NUMBER,
	NFT_ADDRESS,
	TOKEN_ID,
	FROM_ADDRESS,
	TO_ADDRESS,
	TRANSACTION_VALUE,
	Z_DATE,
	Z_TIME,
	Z_MONTH,
	Z_WEEK,
	Z_DO
) as
select 
    EVENT_ID,
    TRANSACTION_HASH,
    BLOCK_NUMBER,
    NFT_ADDRESS,
    TOKEN_ID,
    FROM_ADDRESS,
    TO_ADDRESS,
    TRANSACTION_VALUE,
    to_date(TIMESTAMP) as z_date,
    to_time(TIMESTAMP) as z_time,
    MONTH( TIMESTAMP ) as z_month,
    WEEKOFYEAR( TIMESTAMP ) as z_week,
    DAYOFWEEK( TIMESTAMP ) as z_do
    
FROM NFT_OPEN_SEA.RAW.TRANSFERS;
```

### 3. ETH_USD_VIEW
The query used for cleaning the ETH_USD table:

```ruby
create or replace view NFT_OPEN_SEA.PUBLIC.ETH_USD_VIEW(
	TICKER_DATE,
	"Open",
	"High",
	"Low",
	"Close",
	"Adj Close",
	"Volume"
) as 

SELECT 
    to_date("Date") as ticker_date,
  "Open",
  "High",
  "Low" ,
  "Close",
  "Adj Close",
  "Volume"
FROM 
    "NFT_OPEN_SEA"."RAW"."ETH-USD";
 ```
 
 ### 4. MINTS_VALUE
 Finding the average and median value of mint transactions:
 
 ```ruby
 create or replace view NFT_OPEN_SEA.PUBLIC.MINTS_VALUE(
	NAME,
	SYMBOL,
	EVENTS,
	SUM_VALUE,
	AVG_VALUE,
	MED_VALUE,
	MIN_VALUE,
	MAX_VALUE
) as

select 
    n."NAME",
    n."SYMBOL",
    count(EVENT_ID) as events,
    ROUND(SUM((m."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as sum_value,
    ROUND(AVG((m."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as avg_value,
    ROUND(MEDIAN((m."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) AS med_value,
    ROUND(MIN((m."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as min_value,
    ROUND(MAX((m."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as max_value
    
FROM "NFT_OPEN_SEA"."RAW"."NFTS" AS n
RIGHT JOIN "NFT_OPEN_SEA"."PUBLIC"."MINTS_FULL" AS m
ON n."ADDRESS"=m."NFT_ADDRESS"
LEFT JOIN "NFT_OPEN_SEA"."PUBLIC"."ETH_USD_VIEW" AS d
ON m."Z_DATE"=d."TICKER_DATE"
       
group by n."NAME", n."SYMBOL";
```

 ### 5. TRANSFERS_VALUE
 Finding the average and median value of transfer transactions:
 
 ```ruby
 create or replace view NFT_OPEN_SEA.PUBLIC.TRANSFERS_VALUE(
	DATE,
	EVENTS,
	SUM_VALUE,
	AVG_VALUE,
	MED_VALUE,
	MIN_VALUE,
	MAX_VALUE
) as

select 
    t.Z_DATE AS date,
    count(EVENT_ID) as events,
    ROUND(SUM((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as sum_value,
    ROUND(AVG((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as avg_value,
    ROUND(MEDIAN((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) AS med_value,
    ROUND(MIN((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as min_value,
    ROUND(MAX((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as max_value
    
FROM "NFT_OPEN_SEA"."PUBLIC"."TRANSFERS_FULL" AS t
LEFT JOIN "NFT_OPEN_SEA"."PUBLIC"."ETH_USD_VIEW" AS d
ON t."Z_DATE"=d."TICKER_DATE"
       
group by date;
```
