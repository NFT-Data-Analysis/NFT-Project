# NFT-Project
NFT Data Analysis

The project consists of business questions (in an Excel spreadsheet), 26 SQL queries (used to create views), Power BI dashboards, and a presentation.

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
 Finding the average and median values of mint transactions:
 
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
 Finding the average and median values of transfer transactions:
 
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

### 6. MINTS_2_TRANSFER
 Finding the average and median duration between minting and selling an NFT (between the mint and transfer transaction):
 
 ```ruby
 create or replace view NFT_OPEN_SEA.PUBLIC."mint_2_transfer1"(
	AVG_DATEDIFF_MINT_SALE,
	MED_DATEDIFF_MINT_SALE
) as
SELECT
      ROUND(AVG(DATEDIFF(day,m."Z_DATE",t."Z_DATE")),2) AS avg_datediff_mint_sale,
      MEDIAN(DATEDIFF(day, m."Z_DATE",t."Z_DATE")) AS med_datediff_mint_sale
      FROM (SELECT * FROM "NFT_OPEN_SEA"."PUBLIC"."MINTS_FULL") m
      LEFT JOIN (SELECT * FROM "NFT_OPEN_SEA"."PUBLIC"."TRANSFERS_FULL") t
      ON m."TOKEN_ID"=t."TOKEN_ID" AND m."NFT_ADDRESS"=t."NFT_ADDRESS";
 ```
 
### 7. NUM_TIMES_AN_NFT_SOLD
 Finding how many times a unique NFT was sold:
 
 ```ruby 
create or replace view NFT_OPEN_SEA.PUBLIC.NUM_TIMES_AN_NFT_SOLD
as
SELECT
    NFT_ID,
    "NAME",
    "SYMBOL",
    count(1) as transfers,
    ROUND(avg(v."VALUE"),2) as avg_value,
    sum(v."VALUE") as sum_value,
    min(v."VALUE") as min_value,
    max(v."VALUE") as max_value,
    median(v."VALUE") as median_value,
    CASE WHEN transfers=1 THEN '1'
         WHEN transfers BETWEEN 2 AND 20 THEN '2-20'
         WHEN transfers BETWEEN 21 AND 40 THEN '21-40'
         WHEN transfers BETWEEN 41 AND 60 THEN '41-60'
         WHEN transfers>60 THEN 'ABOVE_60'
         END AS transfers_range

FROM 
(
    SELECT 
        CONCAT(t."TOKEN_ID", '||', t."NFT_ADDRESS") AS nft_id,
        n."NAME",
        n."SYMBOL",
        Z_DATE,
        ROUND(((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as value
    FROM 
       "NFT_OPEN_SEA"."RAW"."NFTS" n
        RIGHT JOIN
        (SELECT * FROM "NFT_OPEN_SEA"."PUBLIC"."TRANSFERS_FULL") t
            ON n."ADDRESS"=t."NFT_ADDRESS"
        LEFT JOIN "NFT_OPEN_SEA"."PUBLIC"."ETH_USD_VIEW" AS d
            ON t."Z_DATE"=d."TICKER_DATE"
   
) v
group by NFT_ID, "NAME", "SYMBOL"
having  AVG_VALUE > 0;
```

### 8. VALUE_DIFF_MINTS_2_TRANSFERS
 Finding the profit (or loss) the minter gained in USD, for selling the NFT for the first time:
 
 ```ruby
 create or replace view NFT_OPEN_SEA.PUBLIC.VALUE_DIFF_MINTS_2_TRANSFERS(
	TRANSFERS_NFT_ID,
	MINTS_NFT_ID,
	NAME,
	SYMBOL,
	TRANSFERS_VALUE,
	MINTS_VALUE,
	TRANSFERS_DIFF,
	TRANSFERS_DIFF_RANGE
) as
SELECT
CONCAT(t."TOKEN_ID", '||', t."NFT_ADDRESS") AS transfers_nft_id,
CONCAT(m."TOKEN_ID", '||', m."NFT_ADDRESS") AS mints_nft_id,
n."NAME",
n."SYMBOL",
 ROUND(((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as transfers_value,
 ROUND(((m."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as mints_value,
transfers_value-mints_value as transfers_diff,
CASE WHEN transfers_diff=0 THEN '0'
     WHEN transfers_diff BETWEEN 0.01 AND 1000 THEN '1-1000'
     WHEN transfers_diff BETWEEN -1000 AND -0.01 THEN '(-)1000-(-)1'
     WHEN transfers_diff BETWEEN -2000 AND -1000.01 THEN '(-)2000-(-)1001'
     WHEN transfers_diff BETWEEN -3000 AND -2000.01 THEN '(-)3000-(-)2001'
     ELSE 'Other'
     END AS transfers_diff_range
    FROM 
       "NFT_OPEN_SEA"."RAW"."NFTS" n
        RIGHT JOIN
        (SELECT * FROM "NFT_OPEN_SEA"."PUBLIC"."MINTS_FULL") m
        ON n."ADDRESS"=m."NFT_ADDRESS" 
        LEFT JOIN
       (SELECT * FROM "NFT_OPEN_SEA"."PUBLIC"."TRANSFERS_FULL") t
            ON m."NFT_ADDRESS"=t."NFT_ADDRESS" AND m."TOKEN_ID"=t."TOKEN_ID"
        LEFT JOIN "NFT_OPEN_SEA"."PUBLIC"."ETH_USD_VIEW" AS d
            ON t."Z_DATE"=d."TICKER_DATE"
            WHERE m."TO_ADDRESS"=t."FROM_ADDRESS";
```

### 9. TRANSFERS_VALUE_DIFF
 Finding the profit (or loss) the NFT seller gained in USD:
 
 ```ruby
 create or replace view NFT_OPEN_SEA.PUBLIC.TRANSFERS_VALUE_DIFF
as
SELECT
    "Z_DATE",
    NFT_ID,
    "NAME",
    "SYMBOL",
    "FROM_ADDRESS",
    "TO_ADDRESS",
      v."END_VALUE",
      v."START_VALUE",
      v."DIFF_VALUE",
      CASE WHEN v."DIFF_VALUE"=0 THEN '0'
           WHEN v."DIFF_VALUE" BETWEEN 0.01 AND 1000 THEN '1-1000'
           WHEN v."DIFF_VALUE" BETWEEN -1000 AND -0.01 THEN '(-)1000-(-)1'
           WHEN v."DIFF_VALUE" BETWEEN -2000 AND -1000.01 THEN '(-)2000-(-)1001'
           WHEN v."DIFF_VALUE" BETWEEN 1000.01 AND 2000 THEN '1000-2000'
           ELSE 'Other'
           END AS diff_value_range
FROM 
(
    SELECT 
        Z_DATE,
        CONCAT(t."TOKEN_ID", '||', t."NFT_ADDRESS") AS nft_id,
        n."NAME",
        n."SYMBOL",
        t."FROM_ADDRESS",
        t."TO_ADDRESS",
        ROUND(((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) - lag(ROUND(((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2), 1, null) over (partition by CONCAT(t."TOKEN_ID", '||', t."NFT_ADDRESS") order by Z_DATE) as diff_value ,
        ROUND(((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as end_value,
        end_value-diff_value as start_value
    FROM 
       (SELECT * FROM "NFT_OPEN_SEA"."RAW"."NFTS") n
        RIGHT JOIN
        (SELECT * FROM "NFT_OPEN_SEA"."PUBLIC"."TRANSFERS_FULL" LIMIT) t
            ON n."ADDRESS"=t."NFT_ADDRESS"
        LEFT JOIN "NFT_OPEN_SEA"."PUBLIC"."ETH_USD_VIEW" AS d
            ON t."Z_DATE"=d."TICKER_DATE"

    qualify 
    diff_value
    is not null
) v
group by "Z_DATE", NFT_ID, "NAME", "SYMBOL", v."END_VALUE", v."START_VALUE", v."DIFF_VALUE", "FROM_ADDRESS", "TO_ADDRESS";
```
