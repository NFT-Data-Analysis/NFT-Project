# NFT-Project
NFT Data Analysis

The project consists of business questions (in an Excel spreadsheet), SQL queries (used to create views), Power BI dashboards, and a presentation.

## SQL Queries

### 1. Mints Full
The query for cleaning the Mints table:

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
The query for cleaning the Transfers table:

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
The query for cleaning the ETH_USD table:

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
