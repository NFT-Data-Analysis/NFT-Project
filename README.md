# NFT-Project
NFT Data Analysis


A collaboration project in which we analyzed a large dataset of NFT transactions using SQL, Snowflake and Power BI, and produced actionable insights.

The project consists of business questions (in an Excel spreadsheet), SQL queries (used to create views), Power BI report, and a presentation.

The dataset represents the activity of the Ethereum NFT market between April 1, 2021 - September 25, 2021.

The dataset contains 4 tables: Mints, Transfers, NFTs, ETH-USD.

[Link to the web Power BI Report](https://app.powerbi.com/view?r=eyJrIjoiMzYwM2I0M2YtYjg3Ny00MWU5LWI0OWEtZWZiNGUzZTlhYjZmIiwidCI6IjMyMTc0NmM2LTQwMzQtNGZjYy1hZDczLTk4NjdlYTRmNGNiMiIsImMiOjl9)

[Link to the Power BI file](https://drive.google.com/file/d/1cOJTix6xpN6E1PjTTvJdpvWACHsEYuNF/view?usp=sharing)


## The Team

Team leader: [Ziva Grushka-Enav](https://www.linkedin.com/in/zivag/)

Team members:

- [Meital Shemer](https://www.linkedin.com/in/meital-shemer/)

- [Eli Hirsch](https://www.linkedin.com/in/eli-b-hirsch/)


## Glossary

| Term | Definition |
|-------| ---------- |
| NFT   | Non-fungible tokens (NFTs) are cryptographic assets on a blockchain with unique identification codes and metadata that distinguish them from each other. Unlike cryptocurrencies, they cannot be traded or exchanged at equivalency.|
| OpenSea | An American online non-fungible token marketplace headquartered in New York City. Currently the largest NFT marketplace. |
| Ethereum | A decentralized, open-source blockchain with smart contract functionality. Ether (ETH) is the cryptocurrency of Ethereum, and is used for buying and selling NFTs. |
| Wei | Wei is the smallest denomination of Ether - the cryptocurrency coin used on the Ethereum network. One ether = 1,000,000,000,000,000,000 wei (10<sup>18</sup>). |
| NFT Minting | Minting an NFT means converting digital data into crypto collections or digital assets recorded on the blockchain. The digital products or files are stored in a distributed ledger or decentralized database and cannot be edited, modified, or deleted. Each token minted has a unique identifier that is directly linked to one Ethereum address. There are usually fees associated with minting. |
|Transfer events | Every time an NFT is transferred on the Ethereum blockchain, it emits a Transfer event which is stored on the blockchain. |



## SQL Queries, Insights and Visualization

### 1. Data Cleaning Queries

#### Mints Full
The query used for cleaning the Mints table:

```sql
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

```sql
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

```sql
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
 
 ```sql
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

![image](https://user-images.githubusercontent.com/106432989/174639551-1bb6d9ee-04c5-4a87-980b-f879183cde92.png)

**The cost of minting an NFT is largely overlooked by minters. It is important to take the minting cost into account, so you can set a profitable selling price.**

**NFT marketplaces (such as OpenSea) can also use this information, for setting fees.**

![image](https://user-images.githubusercontent.com/106432989/174639583-36c2d5f3-ee3d-45c0-92c2-06b1062db91a.png)



 #### TRANSFERS_VALUE
 Finding the average and median values of transfer transactions:
 
 ```sql
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


![image](https://user-images.githubusercontent.com/106432989/174641824-cd0c6a06-592c-4885-938f-e44127d73601.png)

**Minters should be interested in the transfer transaction value in order to estimate the potential selling price.**

**NFT marketplaces (such as OpenSea) can also use this information, for setting fees.**

![image](https://user-images.githubusercontent.com/106432989/174641866-e7da34c5-c1a1-4e45-be7b-47bd403bc0a3.png)

**As for the distribution of transactions value, you can see that up until August 20, 2021, the value of transfer transactions didn't exceed 2,000$.**

**From August 20, 2021 to September 3, 2021, the value of transfer transactions was between 2,000 to around 3,500 USD.**

**From September 4, 2021 to September 25, 2021 (the end of our data), the value of transfer transactions is less than 1,800$.**



### 3. What is the average and median duration between minting and selling an NFT (between the mint and transfer transaction)?

#### MINTS_2_TRANSFER
 
 ```sql
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
 
 ![image](https://user-images.githubusercontent.com/106432989/174647983-230dafe6-0f8d-4785-9f10-7c3ac489e873.png)
 
 **Minters can expect to sell the NFT in a very short time after minting.**

 
### 4. How many times was a unique NFT sold?

#### NUM_TIMES_AN_NFT_SOLD
 
 ```sql 
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

![image](https://user-images.githubusercontent.com/106432989/174648619-c645f5e7-dfe5-40d3-bc8c-ee0618b7c910.png)


### 5. Was the NFT sold (by the minter or by the seller) at a profit or at a loss, and how much?

#### VALUE_DIFF_MINTS_2_TRANSFERS

 Finding the profit (or loss) the minter gained in USD, for selling the NFT for the first time:
 
 ```sql
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

![image](https://user-images.githubusercontent.com/106432989/174671929-c38c1777-0708-451c-85f5-e3d1c35eef76.png)


#### TRANSFERS_VALUE_DIFF
 Finding the profit (or loss) the NFT seller gained in USD:
 
 ```sql
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

![image](https://user-images.githubusercontent.com/106432989/174675213-992292b7-7af9-4934-a14b-61eeed52d2fc.png)

![image](https://user-images.githubusercontent.com/106432989/174713944-c1078f86-3e4b-4e6c-b04e-597d6d8f85c3.png)

**Regulatory Authorities (Tax Authority, AML Authority, etc.) should monitor the profits being made and enforce taxes in accordance with tax policies and treaties.**

#### TRANSFERS_RETURN
 Finding the return the NFT seller gained (%):
 
 ```sql
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

![image](https://user-images.githubusercontent.com/106432989/174875940-3a9373b8-32fb-429e-85cc-98023fb2c5b8.png)


#### TRANSFERS_VALUE_DIFF_TOP_10
 Finding the top 10 profits gained in USD:
 
 ```sql
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

![image](https://user-images.githubusercontent.com/106432989/174876725-ea3acf53-83e6-43ff-8b6b-4c67f78a6ed9.png)


#### TRANSFERS_VALUE_DIFF_BOTTOM_10
 Finding the bottom 10 profits (losses) gained in USD:
 
 ```sql
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

![image](https://user-images.githubusercontent.com/106432989/174880547-645f45c0-e0c3-45ef-9c5f-f4b7ffa8aa40.png)

#### TRANSFERS_VALUE_DIFF_BoredApeYachtClub
 Finding the profit (or loss) the seller gained in USD for a BoredApeYachtClub NFT:
 
 ```sql
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

![image](https://user-images.githubusercontent.com/106432989/174882088-3122c868-6def-4482-9f12-0a2883e5e895.png)

![image](https://user-images.githubusercontent.com/106432989/174882232-4fd56e67-81d0-4828-b464-873122593b75.png)

### 6. What is the average and median duration between buying an NFT and reselling it (i.e. what is the average/ median holding time)?

#### AVG_HOLDING_TIME

```sql
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

![image](https://user-images.githubusercontent.com/106432989/174891221-ac3db2d3-b018-4f8d-bdf6-0ae73bf80089.png)

**The median holding time is around 10 days, which seem to contradict our findings regarding the number of times unique NFTs are sold (once, in the majority of cases, and an average of 1.65 times).**

**A possible explanation to this contradiction is that we found a huge spike in the volume of transfer transactions towards the end of our dataset, which might be skewing the median holding time downwards.**

![image](https://user-images.githubusercontent.com/106432989/174891568-1f49c8bc-081a-4cd7-8697-492d8b62fda0.png)

### 7. Which users (addresses) minted the most amount of NFTs (who are the high volume minters)?

#### MINTS_BY_USER

```sql
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

![image](https://user-images.githubusercontent.com/106432989/174941232-c2927880-025b-4126-a4b3-b4969fcdcf25.png)

**Nearly 60% of users only minted 1 NFT, and around 40% of users minted 2 to 20 NFTS, with a minority of users who minted more than 20 NFTs.**

**NFT marketplaces (such as OpenSea) Can offer discounts, special deals, promotions etc. to the high volume (VIP) minters, and should also consider giving an incentive for minting more than 20 NFTs.**

### 8. Which users (addresses) sold the most amount of NFTs (either minters or buyers)?

#### SALES_BY_USER

```sql
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

![image](https://user-images.githubusercontent.com/106432989/175104791-303e4596-ba37-48d1-92d6-69190cacb4b5.png)

**Around 56% of users only sold 1 NFT, and around 40% of users sold 2 to 20 NFTS, with a minority of users who sold more than 20 NFTs.**

**NFT marketplaces (such as OpenSea), Can offer discounts, special deals, promotions etc. to the high volume (VIP) sellers, and should also consider offers to encorage sellers to sell more NFTs.**

**Regulatory Authorities (Tax Authority, AML Authority, etc.) should monitor the high volume sellers and check if they paid the required taxes, if they're laundering money, etc.**


### 9. Which users (addresses) bought the most amount of NFTs (who are the high volume buyers?)?

#### BUYS_BY_USER

```sql
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

![image](https://user-images.githubusercontent.com/106432989/175105671-16e93ab6-8548-44ac-bccd-86c316be9754.png)

**Most users bought 1 NFT, and up to 5 NFTs.**

**NFT marketplaces (such as OpenSea) should check why users are only one-time buyers and what would make them buy more (survey, A/B testing etc.).**

**Regulatory Authorities (Tax Authority, AML Authority, OpenSea etc.) should monitor the high volume buyers and check the legitamacy of the transactions (that the accounts are real, the transaction makes economic sense, etc.).**

### 10. Is there a day of the week/ time of day in which more transactions occured (either mints or transfers)?

#### mints_day_of_week

```sql
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
 
 ```sql
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

```sql
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
 
 ```sql
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

 ![image](https://user-images.githubusercontent.com/106432989/175106869-9dd93fda-49d2-421a-9936-962cb88588f6.png)
 
* **Most mints occured on Friday.**
* **Most transfer transactions occured on Sunday.**

![image](https://user-images.githubusercontent.com/106432989/175107459-bccd4c28-e3e7-41c5-a62a-e40ace400992.png)

**Both mints' and transfers' strongest hour of the day is 22:00.**

**NFT marketplaces (such as OpenSea) can come up with special deals on other days of the week/ times of day to incentivize more transactions.**

**Minters or other sellers can put the NFT on sale on the most popular day of the week and/ or time of day for transfer transactions in order to attract as many potential buyers as possible.**
 
