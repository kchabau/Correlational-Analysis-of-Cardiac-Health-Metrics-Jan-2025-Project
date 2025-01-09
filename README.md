# Abstract

The objective of this study is to analyze a dataset of **270,000 patients** with reported heart disease to identify correlations between various metrics and cardiac health. Through structured data analysis, key demographic and behavioral factors are investigated to uncover potential risk contributors. This research also serves as a illustration of practical application of SQL and data visualization techniques, advancing data analysis skills in the field of cardiovascular health.

This project is intended to follow up on my SQL learning and illustrate Advanced Analyzing Techniques as well as Data Visualization ability.

We receive this data from [Kaggle](https://www.kaggle.com/datasets/ankushpanday1/heart-attack-in-china-youth-vs-adult) authored by Author Ankush Panday.

This dataset contains data representing the risk factors, demographics, and health behaviors related to heart attacks among youth and adults in China. (From when starting this project, the data set was last updated at approximately *7:00 AM Jan 6, 2025*). I began analyzing this dataset at approximately *12:00 AM Jan 7, 2025.*

The goal is to investigate demographic patterns, key metrics, and underlying factors influencing cardiovascular risk. 

Please refer to the Methods folder for [Results](/Method/README.md)

The [Analysis](/Analysis) contains `.csv` files of query results of our overall data set.

**Please refer** to the [Tableau Dashboard](https://public.tableau.com/app/profile/kevin.chabau/viz/Correlation-Heart-Attack-Project-Jan-2025/Dashboard1?publish=yes) created in conjunction with this project for a better visual understanding of the findings.

## Tools I Use:
- **MySQL**: For database creation, management, and query processing.
- **Visual Studio Code (VS Code)**: As an IDE to write and manage SQL scripts.
- **Python**: For descriptive analysis.
- **Notion**: For project management, task tracking, and timekeeping.
- **Excel**: Data cleaning and preparation.
- **Tableau**: Data visualization and presentation.

## What I Learned:

While analyzing the dataset, I came across multiple challenges including, but not limited to, query performance quality and aggregate data computation.

Query performance is defined as a measure of how quickly and efficiently a database system retrieves data. In other words, our query structure has to be written in a manner that efficiently communicates between nodes, computes, and outputs information. In the particular case of creating a `CASE` statement for categorizing all of the different metric value sets, I attempted to write the query using Common Table Expression (CTE) for each individual category. This caused the performance of the query to take over 30 sec do compute. Actually, the data never rendered. **For laughs**, the query overheated my Laptop, which in turn, ramped up the speed of my fans just to cool it down (*man I need a PC*). When attempting to run the query directly on the database server instead of VS Code, as an IDE, it would disconnect from the local server. This was due to the CTE, processing the information of the data set. At the time of writing the query, each individual row had to be analyzed, for each case group, on the data set of 270,000.

The workaround this, was to create *subquery* for computational efficiency. This allowed the database engine to optimize the execution plan more efficiently. Merging the subquery into the main query, instead of maintaining categorizations within Common Table Expressions, creating a more efficient execution plan. (*Although this is not always the case.*) No pun intended.

## Things I Would Add:

Further scientific statistical analysis, such as, chi-square tests to evaluate relationships between categorical values (ie. gender,cholesterol) and heart attack likelihood.