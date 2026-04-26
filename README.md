# ✈️ Airline Delay Analysis — End-to-End BI Project

![Power BI](https://img.shields.io/badge/Power%20BI-Dashboard-yellow?style=for-the-badge&logo=powerbi)
![Python](https://img.shields.io/badge/Python-Data%20Engineering-blue?style=for-the-badge&logo=python)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Cleaning-lightblue?style=for-the-badge&logo=pandas)

---

## 📌 Project Overview

A fully end-to-end Business Intelligence project built on the **US Airline Delay Causes** dataset. The project covers everything from raw data ingestion and cleaning in Python, through data modeling and dimensional design, all the way to interactive dashboards in Power BI — with no shortcuts taken at any stage.

The goal is to uncover the patterns behind flight delays across airlines, airports, and time periods, enabling data-driven decision-making through clear and interactive visualizations.

---

## 📂 Dataset

| Property | Details |
|----------|---------|
| **Source** | Kaggle — [Airline Delay Causes](https://www.kaggle.com/datasets/giovamata/airlinedelaycauses) |
| **File** | `DelayedFlights.csv` |
| **Size** | ~1.93 Million flight records |
| **Year** | 2008 |

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| **Python** | Data Engineering & Cleaning |
| **Pandas** | Data manipulation and transformation |
| **VS Code** | Development environment |
| **Power BI** | Data Modeling & Dashboard |
| **Power Query (M)** | In-tool transformations |
| **DAX** | Measures and KPIs |

---

## ⚙️ Project Pipeline

### 1️⃣ Data Ingestion
- Loaded the raw CSV file (~1.93M rows) into a Pandas DataFrame
- Performed initial exploration: `shape`, `info()`, `describe()`, `isnull().sum()`

### 2️⃣ Data Cleaning (Python)
- Identified and handled missing values across key columns
- Filled null values with `0` before type casting
- Dropped irrelevant columns (`Unnamed: 0`, columns already captured in dimensions)
- Created a unified `Date` column from `Year`, `Month`, and `DayofMonth`

### 3️⃣ Data Type Optimization
Aggressively downcasted data types to reduce file size and improve Power BI performance:

| Original Type | Optimized Type | Columns |
|---------------|----------------|---------|
| `int64` | `int16` | DepDelay, ArrDelay, Distance, TaxiIn, TaxiOut, AirTime, etc. |
| `int64` | `int8` | Cancelled, Diverted |
| `float64` | `int16` | CarrierDelay, WeatherDelay, NASDelay, SecurityDelay, LateAircraftDelay |

### 4️⃣ Dimensional Modeling (Star Schema)
Designed and exported each dimension as a separate CSV file:

| File | Description |
|------|-------------|
| `fact_flights.csv` | Core fact table with all measures & foreign keys |
| `dim_date.csv` | Date dimension |
| `dim_airline.csv` | Airline dimension (Carrier, Flight, Tail) |
| `dim_airport.csv` | Airport dimension (Origin & Destination) |

> ⚠️ CSV files are not included in the repo due to GitHub file size limits. Generate them locally by running the Python notebook.

#### Star Schema Design

```
                    ┌─────────────┐
                    │  dim_date   │
                    │─────────────│
                    │ Date_ID  PK │
                    │ Date        │
                    │ Year        │
                    │ Month       │
                    │ DayofMonth  │
                    │ DayOfWeek   │
                    └──────┬──────┘
                           │
┌─────────────┐    ┌───────┴──────┐    ┌──────────────┐
│ dim_airline │    │ fact_flights │    │ dim_airport  │
│─────────────│    │──────────────│    │──────────────│
│ Airline_ID  │◄───│ Airline_ID  │───►│ Airport_ID   │
│ UniqueCarri.│    │ Airport_ID  │    │ Origin       │
│ FlightNum   │    │ Date_ID     │    │ Dest         │
│ TailNum     │    │─────────────│    └──────────────┘
└─────────────┘    │ DepDelay    │
                   │ ArrDelay    │
                   │ Distance    │
                   │ TaxiIn/Out  │
                   │ CarrierDelay│
                   │ WeatherDelay│
                   │ NASDelay    │
                   │ SecurityDly │
                   │ LateAircDly │
                   │ AirTime     │
                   │ Cancelled   │
                   │ Diverted    │
                   └─────────────┘
```

### 5️⃣ Power BI — Data Modeling
- Imported all CSV files into Power BI
- Applied additional transformations in **Power Query (M Language)**:
  - Created composite surrogate keys by combining columns using `Text.From()`
  - Built `dim_date` calendar using `List.Dates()` filtered to actual dates in the dataset
  - Removed duplicates and added Index Columns per dimension
- Established relationships between fact and dimension tables in **Model View**

### 6️⃣ DAX Measures
Built a full set of DAX measures including:

```dax
Total Flights = COUNTROWS(fact_flights)
Avg Dep Delay = AVERAGE(fact_flights[DepDelay])
Avg Arr Delay = AVERAGE(fact_flights[ArrDelay])
Total Delay Minutes = SUM(fact_flights[DepDelay])
Best Month = TOPN(1, VALUES(dim_date[Month]), CALCULATE(AVERAGE(fact_flights[DepDelay])), ASC)
Worst Month = TOPN(1, VALUES(dim_date[Month]), CALCULATE(AVERAGE(fact_flights[DepDelay])), DESC)
Selected Delay = SWITCH(SELECTEDVALUE('Delay Causes'[Cause]), ...)
```

---

## 📊 Dashboard Pages

### Page 1 — Flight Operations & Delay Analysis
> A high-level operational overview of all flights and delays

- **KPIs:** Total Flights, AVG Arrival Delay, AVG Departure Delay
- **Line Chart:** AVG Departure & Arrival Delay by Minutes Across Months
- **Bar Chart:** Number of Flights by Airline
- **Bar Chart:** Number of Flights by Airport
- **Pie Chart:** AVG Delay breakdown by Category (Carrier / Weather / NAS / Security / Late Aircraft)
- **Slicers:** Date Range, Origin Airport, Destination Airport, Airport Code

### Page 2 — Performance Trends & Insights
> A deep-dive into performance patterns and outliers

- **KPIs:** Best Month, Worst Month, AVG Departure Delay
- **Area Chart:** Number of Flights by Year, Quarter and Month
- **Bar Chart:** AVG Arrival Delay by Best 7 Airlines
- **Treemap:** Top 10 Worst Flights by AVG Arrival Delay
- **Scatter Chart:** Relation Between Distance and Delay
- **Slicers:** Date Range, Origin Airport, Destination Airport

---

## 💡 Key Insights

- 🗓️ **December** is the worst month for delays, while **October** is the best
- 🏆 **WN (Southwest Airlines)** operates the highest number of flights
- ✈️ **AQ (Aloha Airlines)** has the lowest average arrival delay
- 🛫 **ATL (Atlanta)** is the busiest airport by number of flights
- ⏱️ **Late Aircraft Delay** is the #1 cause of delays at 40.03%, followed by Carrier Delay at 30.35%
- 📏 There is **no strong correlation** between distance and delay duration

---

## 🚀 How to Run

1. Clone the repository
2. Download `DelayedFlights.csv` from [Kaggle](https://www.kaggle.com/datasets/giovamata/airlinedelaycauses) and place it inside a folder named `DelayedFlights.csv/`
3. Run the Python notebook to generate all CSV files locally
4. 📥 Download the Power BI Dashboard from [Google Drive](https://drive.google.com/drive/folders/1EAkcpU0AhhcBvqfvMB1_CXxtNGTQbe-U) and refresh the data source paths

> ⚠️ The `.pbix` file and large CSV files are not included in this repo due to GitHub file size limits.

---

## 👤 Author

Built with 💙 as a full end-to-end BI project — from raw data to interactive dashboard.
