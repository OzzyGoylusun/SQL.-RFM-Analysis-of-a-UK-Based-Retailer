# SQL RFM Analysis of a UK-Based Retailer

## Table of Contents

- [Data Analysis](#data_analysis)
- [Findings](#findings)
- [Recommendations](#recommendations)


### RFM (Recency, Frequency, Monetary) Data Analysis Overview
---
This RFM data analysis work focusses on assisting a UK retailer with re-adjusting its customer segmentation strategy by quantitatively re-ranking and re-grouping its customers 
based on the recency, frequency and monetary value of each transactional data per customer.

<p align="center">
  <img src="https://github.com/OzzyGoylusun/SQL.-RFM-Analysis-of-UK-Retailer/blob/main/Visuals/Representation%20of%20RFM.png"
 alt="Representation of RFM" width="700">
</p>


This adopted data-driven approach is expected to enable the firm to continuously better identify its best customers, appropriately re-segment its customer groups and undertake targeted marketing campaigns accordingly.


### Data Sources

Sales Data: The dataset used for this analysis is the "data.csv" file, containing detailed information about each transaction made by the company's customers.

### Tools

- PostgreSQL Server - Data Analysis
  - [Download Here](https://www.postgresql.org/download/)
  

### Data Cleaning/Preparation

In the data preparation phase, I performed the following tasks:

1. Data loading and inspection
2. Handling missing values
3. Data cleaning
   
### Exploratory Data Analysis (EDA)

EDA primarily involved exploring the complex transaction data to answer these key questions:

- What percentage of all customers have achieved an ultimate RFM score of 100 or 125 out of 125 as its **Champions**?
  
  - For your information, customers receive a score from 1 to 5 from each category based on how recently, frequently and monetarily they have transacted with the retailer, benchmarked against other customers. For instance:
 
      |Recency|Frequency|Monetary|
      |--------|--------|--------|
      |5*|5*|5|
      | | |=125|

- How many potentially loyal customers can the retailer identify to upgrade them to Champions?
- How many customers is the retailer at the risk of losing due to lack of interaction or some other reasons? 


### Data_Analysis

Compared to my *Frequency* and *Monetary* tables, I found the coding with the **Recency** table a bit more complex due to the natural need to also have to calculate the customers' last booking date beforehand:

```sql
WITH RECENCY AS(

  WITH LAST_BOOKING_DATE_CALCS_TABLE AS(

      SELECT "customerID",
          MAX("invoiceDate") AS LAST_BOOKING_DATE
      FROM CLEANED_UP_DATASET
      GROUP BY 1
  ) 
	 
SELECT "customerID",
      EXTRACT(DAY FROM ('2011-12-09 12:50:00'::timestamp - LAST_BOOKING_DATE)) AS RECENCY_VALUE
  --As the dataset is somewhat old, we consider our last booking date the last processed invoice of a customer in the dataset

FROM LAST_BOOKING_DATE_CALCS_TABLE
)
```

In addition, I also found it very effective to assign, for instance, a recency score to each customer from 1 to 5, using the **NTILE() window function**:

```sql

SELECT "customerID",
        RECENCY_VALUE,
        ...,
        ...,
	--By means of NTILE functions, we are assigning each customer a recency, frequency and monetary score
	--from 1 to 5 based on their resulting recency, frequency and monetary values
        NTILE(5) OVER(ORDER BY RECENCY_VALUE DESC) AS RECENCY_SCORE,
        ...,
        ...,

FROM RECENCY
INNER JOIN FREQUENCY USING ("customerID")
INNER JOIN MONETARY USING ("customerID")

```

### Findings

The analysis results can predominantly be summarised as follows:

1. Approx. 15% of all customers have achieved a total Ultimate RFM Score of 100 or 125, considered **Champions**.
2. 96 customers out of 4335 have achieved 4 points on Recency, 3 points on Frequency and 4 points on Monetary, considered **Potential Loyalists**.
3. There are nearly 200 valuable customers who scored less than 3 points on Recency, but scored 4 or 5 points on Frequency and Monetary, considered **At Risk Customers**.

Please note that there were no repetititons/crossovers of customers amongst each customer segment analysed.

### Recommendations

Based on the analysis, I recommend the following actions:

- Consider further rewarding the **Champions** as they could very well become early adopters for new products and also assist with the promotion of the retailer's brand.
- Offer membership/loyalty  programs to the **Potential Loyalists** in an attempt to upgrade them to the **Champions** segment.
- Put in more effort to reconnect with the **At Risk Customers** via personalised communications which include coupons and/or other incentives of some sort.

### Limitations: 

The following records needed to be removed from the RFM analysis in order to ensure the utmost accuracy of my conclusions:

- Cancelled invoices
- Missing/corrupted customerIDs
- Prices lower than $0
- Records associated with manuals and postage fees


### References:

1. [Dataset from Kaggle](https://www.kaggle.com/datasets/carrie1/ecommerce-data/)
2. [RFM Visual](https://rfmcube.com/en/catch-all-guide-on-your-customer-base-rfm-analysis/)
