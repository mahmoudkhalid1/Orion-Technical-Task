🛠️ ETL Logic & Data Quality Report
The transformation process turned a messy, flat file of 298246 records into a high-performance relational model. Below is the breakdown of the technical hurdles and how they were solved.

1. Data Cleaning & Quality Audit
Before modeling, a Python-based audit was conducted to identify data "noise."

Handling Null Values:

-Problem: Columns like Name ,Education and Occupation had missing values (approx. 90% of the      dataset).

-Solution: Instead of deleting rows (which would lose sales revenue), I used .fillna('Unknown').  This keeps the financial data accurate while allowing for a "Unknown" category in the slicers.

Duplicate Management:

-Audit: Identified 73% exact row duplicates likely caused by system extraction errors.

-Solution: Applied drop_duplicates() in Python to prevent the artificial inflation of Sales and   Quantity metrics.

Data Type Standardization:

-Dates: Converted OrderDate from object/string to datetime64 to enable Power BI Time              Intelligence (YoY and YTD).
-Convert other columns (not numeric cols) to string from object so the decerease the run time.

-Financials: Stripped currency symbols from Net Price and cast to float for calculation.

2. The ETL Pipeline (Python Logic)
The ETL followed a three-step modular approach:

Extract: Ingested the raw CSV using Pandas.

Transform:
Concatenation: Created a CityState unique key (City + "-" + State) to solve the "duplicate city name" problem across different regions as in USA , cities with the same name can be in different states.
Normalization: Split the flat file into a Star Schema (1 Fact, 3 Dimensions). This reduced the file size by nearly 40% by removing redundant text strings.

Load: Exported structured CSVs ready for Power BI ingestion.

📐 Data Model & Architecture

The Star Schema

I implemented a Star Schema to optimize query speed and filter behavior:
Fact Table (fact_sales): Contains the quantitative data (Quantity, Net Price) and the keys linking to dimensions.
Dimension Tables: * dim_product: Product attributes (Brand, Category, Color).
dim_customer: Demographic data (Education, Occupation).
dim_geography: The 5-level hierarchy (Continent > Region > Country > State > City).

📝 Key Assumptions

To ensure the dashboard remains accurate, the following assumptions were made during development:
Currency: All financial figures are assumed to be in a single consistent currency (USD), as no exchange rate column was provided in the raw data.
Net Price: Total Sales is calculated as Quantity * Net Price.
Year Span: The analysis focuses strictly on the 2008-2009 period; any data outside this range was filtered out to maintain the "Year-over-Year" logic.
Geography: The CityState composite key assumes that no two cities in the same state share the same name.

🚀 Technical Challenges & Solutions

Challenge                Solution
Flat File Performance    Solution: Architected a Star Schema. By moving repetitive text (like                             "United States") into a Dimension table, Power BI only has to store an                           Integer key in the Fact table, making the dashboard significantly                                faster.
City Name Ambiguity      Solution: Created the City-State key. This ensures "Portland, Maine"                             and "Portland, Oregon" are treated as two separate locations in the                              geographic hierarchy.
Forecast Matching        Solution: Since the Forecast was at a higher level than the Sales data,                          I used DAX to ensure the Gauge chart stays accurate even when a user                             drills down into a specific City.
