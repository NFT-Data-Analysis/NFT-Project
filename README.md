# NFT-Project
NFT Data Analysis


A collaboration project in which we analyzed a large dataset of NFT transactions using SQL, Snowflake and Power BI, and produced actionable insights.

The project consists of business questions (in an Excel spreadsheet), SQL queries (used to create views), Power BI dashboards, and a presentation.

The dataset represents the activity of the Ethereum NFT market between April 1, 2021 - September 25, 2021.

The dataset contains 4 tables: Mints, Transfers, NFTs, ETH-USD.


## The Team

Team leader: Ziva Grushka-Enav https://www.linkedin.com/in/zivag/

Team members:

- Meital Shemer https://www.linkedin.com/in/meital-shemer/

- Eli Hirsch https://www.linkedin.com/in/eli-b-hirsch/


## Glossary

| Term | Definition |
|-------| ---------- |
| NFT   | Non-fungible tokens (NFTs) are cryptographic assets on a blockchain with unique identification codes and metadata that distinguish them from each other. Unlike cryptocurrencies, they cannot be traded or exchanged at equivalency.|
| OpenSea | An American online non-fungible token marketplace headquartered in New York City. Currently the largest NFT marketplace. |
| Ethereum | A decentralized, open-source blockchain with smart contract functionality. Ether (ETH) is the cryptocurrency of Ethereum, and is used for buying and selling NFTs. |
| Wei | Wei is the smallest denomination of Ether - the cryptocurrency coin used on the Ethereum network. One ether = 1,000,000,000,000,000,000 wei (10<sup>18</sup>). |
| NFT Minting | Minting an NFT means converting digital data into crypto collections or digital assets recorded on the blockchain. The digital products or files are stored in a distributed ledger or decentralized database and cannot be edited, modified, or deleted. Each token minted has a unique identifier that is directly linked to one Ethereum address. There are usually fees associated with minting. |
|Transfer events | Every time an NFT is transferred on the Ethereum blockchain, it emits a Transfer event which is stored on the blockchain. |



## SQL Queries

### 1. Data Cleaning Queries

#### Mints Full
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

#### Transfers Full
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

#### ETH_USD_VIEW
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
 
 ### 2. What is the average and median values of the transactions (mints and transfers), and how are they distributed?
 
 #### MINTS_VALUE
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

 #### TRANSFERS_VALUE
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

### 3. What is the average and median duration between minting and selling an NFT (between the mint and transfer transaction)?

#### MINTS_2_TRANSFER
 
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
 
### 4. How many times was a unique NFT sold?

#### NUM_TIMES_AN_NFT_SOLD
 
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

### 5. Was the NFT sold (by the minter or by the seller) at a profit or at a loss, and how much?

#### VALUE_DIFF_MINTS_2_TRANSFERS

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

#### TRANSFERS_VALUE_DIFF
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

#### TRANSFERS_RETURN
 Finding the return the NFT seller gained (%):
 
 ```ruby
 create or replace view NFT_OPEN_SEA.PUBLIC.TRANSFERS_RETURN(
	Z_DATE,
	NFT_ID,
	NAME,
	SYMBOL,
	FROM_ADDRESS,
	TO_ADDRESS,
	END_VALUE,
	START_VALUE,
	DIFF_VALUE,
	PERCENT_RETURN
) as
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
      ROUND(((v."DIFF_VALUE"/v."START_VALUE"))*100,0) as PERCENT_RETURN

FROM 
(
    SELECT 
        Z_DATE,
        CONCAT(t."TOKEN_ID", '||', t."NFT_ADDRESS") AS nft_id,
        n."NAME",
        n."SYMBOL",
        t."FROM_ADDRESS",
        t."TO_ADDRESS",
        ROUND(((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) - lag(ROUND(((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2), 1, null) over (partition by CONCAT(t."TOKEN_ID", '||', t."NFT_ADDRESS") order by Z_DATE) as diff_value,
        ROUND(((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as end_value,
        end_value-diff_value as start_value
    FROM 
       (SELECT * FROM "NFT_OPEN_SEA"."RAW"."NFTS") n
        RIGHT JOIN
        (SELECT * FROM "NFT_OPEN_SEA"."PUBLIC"."TRANSFERS_FULL") t
            ON n."ADDRESS"=t."NFT_ADDRESS"
        LEFT JOIN "NFT_OPEN_SEA"."PUBLIC"."ETH_USD_VIEW" AS d
            ON t."Z_DATE"=d."TICKER_DATE"

    qualify 
    diff_value
    is not null
) v
WHERE v."START_VALUE" IS NOT NULL AND v."START_VALUE"!=0
group by "Z_DATE", NFT_ID, "NAME", "SYMBOL", v."END_VALUE", v."START_VALUE", v."DIFF_VALUE", "FROM_ADDRESS", "TO_ADDRESS", PERCENT_RETURN;
```

#### TRANSFERS_VALUE_DIFF_TOP_10
 Finding the top 10 profits gained in USD:
 
 ```ruby
 create or replace view NFT_OPEN_SEA.PUBLIC.TRANSFERS_VALUE_DIFF_TOP_10
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
       ROUND(((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) - lag(ROUND(((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2), 1, null) over (partition by CONCAT(t."TOKEN_ID", '||', t."NFT_ADDRESS") order by Z_DATE) as diff_value,
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
  ORDER BY diff_value DESC
  LIMIT 10
) v
group by "Z_DATE", NFT_ID, "NAME", "SYMBOL", v."END_VALUE", v."START_VALUE", v."DIFF_VALUE", "FROM_ADDRESS", "TO_ADDRESS";
```

#### TRANSFERS_VALUE_DIFF_BOTTOM_10
 Finding the bottom 10 profits (losses) gained in USD:
 
 ```ruby
 create or replace view NFT_OPEN_SEA.PUBLIC.TRANSFERS_VALUE_DIFF_BOTTOM_10
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
       ROUND(((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) - lag(ROUND(((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2), 1, null) over (partition by CONCAT(t."TOKEN_ID", '||', t."NFT_ADDRESS") order by Z_DATE) as diff_value,
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
  ORDER BY diff_value ASC
  LIMIT 10
) v
group by "Z_DATE", NFT_ID, "NAME", "SYMBOL", v."END_VALUE", v."START_VALUE", v."DIFF_VALUE", "FROM_ADDRESS", "TO_ADDRESS";
```

#### TRANSFERS_VALUE_DIFF_BoredApeYachtClub
 Finding the profit (or loss) the seller gained in USD for a BoredApeYachtClub NFT:
 
 ```ruby
 create or replace view NFT_OPEN_SEA.PUBLIC.TRANSFERS_VALUE_DIFF_BoredApeYachtClub
as
SELECT
    "Z_DATE",
    NFT_ID,
    "NAME",
    "SYMBOL",
    v."FROM_ADDRESS",
    v."TO_ADDRESS",
      v."END_VALUE",
      v."START_VALUE",
      v."DIFF_VALUE",
      CASE WHEN v."DIFF_VALUE"=0 THEN '0'
     WHEN v."DIFF_VALUE" BETWEEN 0.01 AND 1000 THEN '1-1000'
     WHEN v."DIFF_VALUE" BETWEEN -1000 AND -0.01 THEN '(-)1000-(-)1'
     WHEN v."DIFF_VALUE" BETWEEN -2000 AND -1000.01 THEN '(-)2000-(-)1001'
     WHEN v."DIFF_VALUE" BETWEEN 1000.01 AND 2000 THEN '1000-2000'
     WHEN v."DIFF_VALUE">2000 THEN 'ABOVE_2000'
     WHEN v."DIFF_VALUE"<-2000 THEN 'BELOW_(-)2000'
     END AS diff_value_range
FROM 
(
    SELECT 
        t.Z_DATE,
        CONCAT(t."TOKEN_ID", '||', t."NFT_ADDRESS") AS nft_id,
        n."NAME",
        n."SYMBOL",
        t."FROM_ADDRESS",
        t."TO_ADDRESS",
       ROUND(((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) - lag(ROUND(((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2), 1, null) over (partition by CONCAT(t."TOKEN_ID", '||', t."NFT_ADDRESS") order by t.Z_DATE) as diff_value,
        ROUND(((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as end_value,
        end_value-diff_value as start_value
    FROM 
       (SELECT * FROM "NFT_OPEN_SEA"."RAW"."NFTS") n
        RIGHT JOIN
        (SELECT * FROM "NFT_OPEN_SEA"."PUBLIC"."TRANSFERS_FULL") t
            ON n."ADDRESS"=t."NFT_ADDRESS"
        LEFT JOIN "NFT_OPEN_SEA"."PUBLIC"."ETH_USD_VIEW" AS d
            ON t."Z_DATE"=d."TICKER_DATE"
  WHERE n."SYMBOL"='BAYC'
    qualify 
    diff_value
    is not null
) v
group by "Z_DATE", NFT_ID, "NAME", "SYMBOL", v."END_VALUE", v."START_VALUE", v."DIFF_VALUE", v."FROM_ADDRESS", v."TO_ADDRESS"
```

### 6. What is the average and median duration between buying an NFT and reselling it (i.e. what is the average/ median holding time)?

#### AVG_HOLDING_TIME

```ruby
create or replace view NFT_OPEN_SEA.PUBLIC.AVG_HOLDING_TIME as
SELECT
    NFT_ID,
    "NAME",
    "SYMBOL",
    count(1) as transfers,
    ROUND(avg(v."DIFF_TO_PREV"),2) as avg_diff,
    median(v."DIFF_TO_PREV") as med_diff,
    ROUND(avg(v."VALUE"),2) as avg_value,
    sum(v."VALUE") as sum_value,
    min(v."VALUE") as min_value,
    max(v."VALUE") as max_value,
    median(v."VALUE") as median_value,
    CASE WHEN avg_diff BETWEEN 0 AND 10 THEN '0-10'
         WHEN avg_diff BETWEEN 11 AND 30 THEN '11-30'
         WHEN avg_diff BETWEEN 31 AND 50 THEN '31-50'
         WHEN avg_diff BETWEEN 51 AND 70 THEN '51-70'
         WHEN avg_diff BETWEEN 71 AND 90 THEN '71-90'
         ELSE 'Other'
     END AS avg_diff_range

FROM 
(
    SELECT 
        CONCAT(t."TOKEN_ID", '||', t."NFT_ADDRESS") AS nft_id,
        n."NAME",
        n."SYMBOL",
        Z_DATE,
        Z_DATE - lag(Z_DATE, 1, null) over (partition by CONCAT(t."TOKEN_ID", '||', t."NFT_ADDRESS") order by Z_DATE) as diff_to_prev ,
        ROUND(((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as value
    FROM 
       (SELECT * FROM "NFT_OPEN_SEA"."RAW"."NFTS") n
        RIGHT JOIN
        (SELECT * FROM "NFT_OPEN_SEA"."PUBLIC"."TRANSFERS_FULL") t
            ON n."ADDRESS"=t."NFT_ADDRESS"
        LEFT JOIN "NFT_OPEN_SEA"."PUBLIC"."ETH_USD_VIEW" AS d
            ON t."Z_DATE"=d."TICKER_DATE"

    qualify 
    diff_to_prev
    is not null
) v
group by NFT_ID, "NAME", "SYMBOL"
having  AVG_VALUE > 0;
```

### 7. Which users (addresses) minted the most amount of NFTs (who are the "heavy" minters)?

#### MINTS_BY_USER

```ruby
create or replace view NFT_OPEN_SEA.PUBLIC.MINTS_BY_USER
as
SELECT 
    m."TO_ADDRESS",
    n."NAME",
    n."SYMBOL",
    COUNT(m."TO_ADDRESS") AS total_mints,
    ROUND(SUM((m."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as sum_value,
    ROUND(AVG((m."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as avg_value,
    ROUND(MEDIAN((m."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) AS med_value,
    ROUND(MIN((m."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as min_value,
    ROUND(MAX((m."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as max_value,
    MAX(m."Z_DATE") - MIN(m."Z_DATE") as time_active,
    CASE WHEN total_mints=1 THEN '1'
         WHEN total_mints BETWEEN 2 AND 20 THEN '2-20'
         WHEN total_mints BETWEEN 21 AND 40 THEN '21-40'
         WHEN total_mints>40 THEN 'ABOVE_40'
         END AS total_mints_range
FROM "NFT_OPEN_SEA"."RAW"."NFTS" AS n
RIGHT JOIN (SELECT * FROM "NFT_OPEN_SEA"."PUBLIC"."MINTS_FULL") AS m
ON n."ADDRESS"=m."NFT_ADDRESS"
LEFT JOIN "NFT_OPEN_SEA"."PUBLIC"."ETH_USD_VIEW" AS d
ON m."Z_DATE"=d."TICKER_DATE"
GROUP BY m."TO_ADDRESS", n."NAME", n."SYMBOL";
```

### 8. Which users (addresses) sold the most amount of NFTs (either minters or buyers)?

#### SALES_BY_USER

```ruby
create or replace view NFT_OPEN_SEA.PUBLIC.SALES_BY_USER
as
SELECT 
    t."FROM_ADDRESS", n."NAME", n."SYMBOL",
    COUNT(t."FROM_ADDRESS") AS total_sellers,
    COUNT(CASE WHEN t."FROM_ADDRESS"=t."TO_ADDRESS" THEN t."TO_ADDRESS" END) AS sellers_r_buyers, 
    ROUND(SUM((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as sum_value,
    ROUND(AVG((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as avg_value,
    ROUND(MEDIAN((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) AS med_value,
    ROUND(MIN((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as min_value,
    ROUND(MAX((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as max_value,
    MAX(t."Z_DATE") - MIN(t."Z_DATE") as time_active,
    CASE WHEN total_sellers=1 THEN '1'
         WHEN total_sellers BETWEEN 2 AND 20 THEN '2-20'
         WHEN total_sellers BETWEEN 21 AND 40 THEN '21-40'
         WHEN total_sellers>40 THEN 'ABOVE_40'
         END AS total_sellers_range
FROM "NFT_OPEN_SEA"."RAW"."NFTS" AS n
RIGHT JOIN (SELECT * FROM "NFT_OPEN_SEA"."PUBLIC"."TRANSFERS_FULL") AS t
ON n."ADDRESS"=t."NFT_ADDRESS"
LEFT JOIN "NFT_OPEN_SEA"."PUBLIC"."ETH_USD_VIEW" AS d
ON t."Z_DATE"=d."TICKER_DATE"
GROUP BY t."FROM_ADDRESS", n."NAME", n."SYMBOL";
```

### 9. Which users (addresses) bought the most amount of NFTs (who are the "heavy" buyers?)?

#### BUYS_BY_USER

```ruby
create or replace view NFT_OPEN_SEA.PUBLIC.BUYS_BY_USER
as
SELECT 
    t."TO_ADDRESS",
    n."NAME",
    n."SYMBOL",
    COUNT(t."TO_ADDRESS") AS total_buyers,
    COUNT(CASE WHEN t."TO_ADDRESS"=t."FROM_ADDRESS" THEN t."FROM_ADDRESS" END) AS buyers_r_sellers, 
    ROUND(SUM((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as sum_value,
    ROUND(AVG((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as avg_value,
    ROUND(MEDIAN((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) AS med_value,
    ROUND(MIN((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as min_value,
    ROUND(MAX((t."TRANSACTION_VALUE"/POWER(10,18))*d."Close"),2) as max_value,
    MAX(t."Z_DATE") - MIN(t."Z_DATE") as time_active,
    CASE WHEN total_buyers=1 THEN '1'
         WHEN total_buyers=2 THEN '2'
         WHEN total_buyers=3 THEN '3'
         WHEN total_buyers=4 THEN '4'
         WHEN total_buyers=5 THEN '5'
         ELSE 'Other'
         END AS total_buyers_catagories
FROM "NFT_OPEN_SEA"."RAW"."NFTS" AS n
RIGHT JOIN (SELECT * FROM "NFT_OPEN_SEA"."PUBLIC"."TRANSFERS_FULL") AS t
ON n."ADDRESS"=t."NFT_ADDRESS"
LEFT JOIN "NFT_OPEN_SEA"."PUBLIC"."ETH_USD_VIEW" AS d
ON t."Z_DATE"=d."TICKER_DATE"
GROUP BY t."TO_ADDRESS", n."NAME", n."SYMBOL";
```

### 10. Is there a day of the week/ time of day in which more transactions occured (either mints or transfers)?

#### mints_day_of_week

```ruby
create or replace view NFT_OPEN_SEA.PUBLIC."mints_day_of_week"(
	DAY,
	NUM_DAY
) as
SELECT *
FROM
(SELECT 
       DAYNAME (t."Z_DATE") AS Day, COUNT (1) AS Num_Day
       FROM "NFT_OPEN_SEA"."PUBLIC"."MINTS_FULL" AS t
       GROUP BY Day
       ORDER BY 
       CASE Day
          WHEN 'Sun' THEN 1
          WHEN 'Mon' THEN 2
          WHEN 'Tue' THEN 3
          WHEN 'Wed' THEN 4
          WHEN 'Thu' THEN 5
          WHEN 'Fri' THEN 6
          WHEN 'Sat' THEN 7
       END ASC);
 ```
 
 #### MINTS_TIME_OF_DAY
 
 ```ruby
 create or replace view NFT_OPEN_SEA.PUBLIC.MINTS_TIME_OF_DAY(
	TIME,
	NUM_TIME
) as
SELECT 
       HOUR (t."Z_TIME") AS TIME, COUNT (1) AS Num_Time
       FROM "NFT_OPEN_SEA"."PUBLIC"."MINTS_FULL" AS t
       GROUP BY TIME
       ORDER BY TIME;
```

#### transfers_day_of_week

```ruby
create or replace view NFT_OPEN_SEA.PUBLIC."transfers_day_of_week"(
	DAY,
	NUM_DAY
) as
SELECT *
FROM
(SELECT 
       DAYNAME (t."Z_DATE") AS Day, COUNT (1) AS Num_Day
       FROM "NFT_OPEN_SEA"."PUBLIC"."TRANSFERS_FULL" AS t
       GROUP BY Day
       ORDER BY 
       CASE Day
          WHEN 'Sun' THEN 1
          WHEN 'Mon' THEN 2
          WHEN 'Tue' THEN 3
          WHEN 'Wed' THEN 4
          WHEN 'Thu' THEN 5
          WHEN 'Fri' THEN 6
          WHEN 'Sat' THEN 7
       END ASC);
 ```
 
 #### TRANSFERS_TIME_OF_DAY
 
 ```ruby
 create or replace view NFT_OPEN_SEA.PUBLIC.TRANSFERS_TIME_OF_DAY(
	TIME,
	NUM_TIME
) as
SELECT 
       HOUR (t."Z_TIME") AS TIME, COUNT (1) AS Num_Time
       FROM "NFT_OPEN_SEA"."PUBLIC"."TRANSFERS_FULL" AS t
       GROUP BY TIME
       ORDER BY TIME;
 ```
