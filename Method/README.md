# Methods and Results

SQL was used to query and preprocess the dataset. Basic demographic queries included the calculation of average age by gender, identification of age ranges, and categorization of data using CASE statements.

```
SELECT ROUND(AVG(age),2) AS `Average Female Age`
FROM heart_attack_china_youth_vs_adult
WHERE Gender = 'Female';

SELECT ROUND(AVG(age),2) AS `Average Male Age`
FROM heart_attack_china_youth_vs_adult
WHERE Gender = 'Male';
```

**Average Female Age:** 35.54 yoa

**Average Male Age:** 35.52 yoa


**Age of Participant Range:** 12 - 59 yoa
```
SELECT MIN(age)
FROM heart_attack_china_youth_vs_adult;

SELECT MAX(age)
FROM heart_attack_china_youth_vs_adult;
```

The average age of participants is approximately 35.5 years, with minimal variation between genders. Age distribution spans from 12 to 59 years, indicating a wide range of youth and adult populations.


## Metrics

To understand risk factors affecting the cardio-vascular system, I researched the available resources on the [American College of Cardiology](https://tools.acc.org/ascvd-risk-estimator-plus/#!/content/resources/) website. The  organization presents both Clinical and Patient Resources available which I used for looking at key factors associated with *at-risk* individuals including a Risk-Estimator calculator. 

Key factors affecting cardiovascular health are (but not limited to):
- Diet and Physical Activity
- Weight 
- Blood Cholesterol
- Blood Pressure
- Family History

For this section, we will be analyzing individuals who have experienced heart attacks and measuring the average across key metrics, to isolate the suspecting factors heart attacks. Our data set also includes other factors such as sleeping hours, employment status, income level, education level, alcohol consumption, and more. We will be measuring this data to see if we can retrieve insights from it. One of my hypothesis is, that there is a correlation with income level and stress level, which play key roles in affecting individual health.

But first, let's measure our patient data set. 
```
WITH confirmed AS (
    SELECT 
        COUNT(*) AS `Total Patients`
FROM heart_attack_china_youth_vs_adult
WHERE Heart_Attack = 'Yes' 
),
unconfirmed AS (
    SELECT 
        COUNT(*) AS `Total Patients`
    FROM heart_attack_china_youth_vs_adult
    WHERE Heart_Attack = 'No'
)
SELECT
    confirmed.`Total Patients` + unconfirmed.`Total Patients` AS `Total Patients`,
    confirmed.`Total Patients` AS `Confirmed Heart Attack`,
    unconfirmed.`Total Patients` AS `Unconfirmed Heart Attack`,
    ROUND((confirmed.`Total Patients` / (confirmed.`Total Patients` + unconfirmed.`Total Patients`)) * 100, 2) AS `% of Confirmed Heart Attack`,
    ROUND((unconfirmed.`Total Patients` / (confirmed.`Total Patients` + unconfirmed.`Total Patients`)) * 100, 2) AS `% of Unconfirmed Heart Attack`
FROM confirmed, unconfirmed;
```
$\text{Percentage of Confirmed Heart Attack} = \left( \frac{\text{Confirmed Total Patients}}{\text{Total Patients}} \right) \times 100$



$\text{Percentage of Unconfirmed Heart Attack} = \left( \frac{\text{Unconfirmed Total Patients}}{\text{Total Patients}} \right) \times 100$

Utilizing a mathematical formula we can first determine the percentage of individuals with confirmed and unconfirmed heart attacks within our data set. The query provides a quick overview of the distribution of heart attack cases (confirmed vs. unconfirmed) in the dataset. This is useful for summarizing the data without inspecting individual records.

| Total Patients | Confirmed Heart Attack | Unconfirmed Heart Attack | % of Confirmed Heart Attack | % of Unconfirmed Heart Attack |
|----------------|-------------------------|---------------------------|---------------------------------------|-----------------------------------------|
| 270,000        | 31,959                 | 238,041                  | 11.84%                                | 88.16%                                  |


A significant proportion (**88.16%**) of individuals in the dataset **did not have a confirmed heart attack.** This could indicate either a lack of medical verification or alternative diagnoses that do not meet the criteria for a confirmed heart attack. The **confirmed heart attack** rate of 11.84% is specific to this group and may not apply to other populations with different risk factors or health conditions.

To further interpret the data, we will need more context about the population (age, gender, medical history, lifestyle, etc.) and the criteria used for confirming a heart attack.

We will begin by analyzing groups and define our metrics. 
- Age:
    - Adolescent: Age 15 and below
    - Young Adult: Ages 16 to 24
    - Middle Age Adult: Ages 25 to 39
    - Adult: Ages 40 to 59
    - Senior: Age 60 and above 
```
WITH age_cat AS (
    SELECT 
        gender,
        Heart_Attack,
        CASE
            WHEN age <= 15 THEN 'Adolescent'
            WHEN age BETWEEN 16 AND 24 THEN 'Young Adult'
            WHEN age BETWEEN 25 AND 39 THEN 'Middle Age Adult'
            WHEN age BETWEEN 40 AND 59 THEN 'Adult'
            ELSE 'Senior'
        END AS age_group
    FROM heart_attack_china_youth_vs_adult
)
SELECT 
    age_group AS 'Age Group',
    gender AS 'Gender',
    heart_attack AS 'Heart Attack',
    COUNT(*) AS Count,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) 
        OVER(PARTITION BY age_group), 2) AS 'Percentage',
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) 
        OVER(PARTITION BY age_group, gender), 2) AS '% of Heart Attack by Gender',
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) 
        OVER(PARTITION BY heart_attack), 2) AS '% of Heart Attack'
FROM age_cat
GROUP BY age_group, gender, heart_attack
ORDER BY age_group, gender, heart_attack;

```
This query provides a clear breakdown of heart attack prevalence across different age groups. The **Percentage** help identify the relative distribution of individuals within each age group (across all genders) who have or haven't had a heart attack. This can be useful for public health analysis or identifying which age groups are most affected by heart attacks, and for targeting health interventions or further studies on heart attack causes in specific age groups.

**% of Heart Attack by Gender**: The percentage of individuals with a heart attack for each age group and gender.

**% of Heart Attack**: The percentage of heart attacks in each group relative to the total number of heart attacks in the dataset.


| Age Group         | Gender | Heart Attack | Count | Percentage | % of Heart Attack by Gender | % of Heart Attack |
|-------------------|--------|--------------|-------|------------|-----------------------------|--------------------|
| Adolescent        | Female | No           | 9272  | 41.40      | 87.52                       | 3.90              |
| Adolescent        | Female | Yes          | 1322  | 5.90       | 12.48                       | 4.14              |
| Adolescent        | Male   | No           | 9675  | 43.20      | 88.56                       | 4.06              |
| Adolescent        | Male   | Yes          | 1250  | 5.58       | 11.44                       | 3.91              |
| Adolescent        | Other  | No           | 759   | 3.39       | 86.45                       | 0.32              |
| Adolescent        | Other  | Yes          | 119   | 0.53       | 13.55                       | 0.37              |
| Adult             | Female | No           | 47472 | 42.15      | 87.89                       | 19.94             |
| Adult             | Female | Yes          | 6543  | 5.81       | 12.11                       | 20.47             |
| Adult             | Male   | No           | 47647 | 42.31      | 88.06                       | 20.02             |
| Adult             | Male   | Yes          | 6462  | 5.74       | 11.94                       | 20.22             |
| Adult             | Other  | No           | 3998  | 3.55       | 88.82                       | 1.68              |
| Adult             | Other  | Yes          | 503   | 0.45       | 11.18                       | 1.57              |
| Middle Age Adult  | Female | No           | 36059 | 42.59      | 88.52                       | 15.15             |
| Middle Age Adult  | Female | Yes          | 4678  | 5.52       | 11.48                       | 14.64             |
| Middle Age Adult  | Male   | No           | 35849 | 42.34      | 88.44                       | 15.06             |
| Middle Age Adult  | Male   | Yes          | 4687  | 5.54       | 11.56                       | 14.67             |
| Middle Age Adult  | Other  | No           | 2961  | 3.50       | 87.17                       | 1.24              |
| Middle Age Adult  | Other  | Yes          | 436   | 0.51       | 12.83                       | 1.36              |
| Young Adult       | Female | No           | 21402 | 42.54      | 88.27                       | 8.99              |
| Young Adult       | Female | Yes          | 2843  | 5.65       | 11.73                       | 8.90              |
| Young Adult       | Male   | No           | 21143 | 42.03      | 88.06                       | 8.88              |
| Young Adult       | Male   | Yes          | 2868  | 5.70       | 11.94                       | 8.97              |
| Young Adult       | Other  | No           | 1804  | 3.59       | 87.91                       | 0.76              |
| Young Adult       | Other  | Yes          | 248   | 0.49       | 12.09                       | 0.78              |

**Females** and **males** have a relatively similar percentage of heart attacks (approximately 11%-12% of their gender group within each age category). The **percentage of heart attacks by gender** is consistent across the different age groups. For example, **females** and **males** have similar heart attack rates within each age group, with some variations.

In general, **females** tend to have slightly higher percentages of heart attack cases across all age groups compared to males, especially in the **Adult** and **Middle Age** Adult categories.

Heart attack rates (both confirmed and unconfirmed) are generally higher among the **Adult** and **Middle Age Adult** groups. The **Young Adult** group (ages 16-39) shows lower overall heart attack rates, but this group still experiences noticeable rates of heart attacks, especially among males.

This data can be useful for understanding how heart attack rates are distributed across gender and age groups, and for identifying which demographics are more likely to experience heart attacks. But let's dig deeper by using aggregating the units of measurement for things such as BMI, cholesterol Level, blood pressure, heart rate and some other factors.

The table below outlines the ranges for healthy, elevated, or high blood pressure, as per the [American Heart Association (AHA)Trusted Source](https://www.heart.org/en/health-topics/high-blood-pressure/understanding-blood-pressure-readings):
![Figure 1](/Assets/healthline-blood-pressure-ranges.jpeg)

The BMI indicator has the following formula to determine different categories of Body Mass. is a measurement that doctors use to gauge the relationship between weight and overall health. It's based on a person's height and tissue mass, which includes muscle, fat, and bone. BMI can be calculated using a weighing scale and stadiometer, or by using a BMI calculator or lookup table. 
> *It can be less useful than other methods when predicting an individual's health, especially for people with short stature, abdominal obesity, or high muscle mass.*

$BMI = \frac{Weight \, (kg)}{Height \, (m)^2}$

![Figure 2](/Assets/BMIndex.png)

According to the [CDC](https://www.cdc.gov/cholesterol/about/index.html#:~:text=What%20it%20is-,Blood%20cholesterol%20is%20a%20waxy%2C%20fat%2Dlike%20substance%20made%20by,your%20body%20uses%20for%20energy.), cholesterol is a waxy, fat-like substance made by your liver. Blood cholesterol is essential for good health. Your body needs it to perform important jobs, such as making hormones and digesting fatty foods. It is essential for things such as cell structures, hormone production, vitamin synthetization of vitamins, and more. 

![Cleveland Clinic Organization](/Assets/cleveland-clinic-org-cholesterol-level.jpg)

```
SELECT 
    Gender,
    Heart_Attack,
    age_group,
    bp_group,
    bmi_group,
    cholesterol_group,
    SUM(CASE WHEN Heart_Attack = 'Yes' THEN 1 ELSE 0 END) AS 'Confirmed Heart Attack Count',
    ROUND(SUM(CASE WHEN Heart_Attack = 'Yes' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS '% of Heart Attack in Group',
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM heart_attack_china_youth_vs_adult), 2) AS '% of Total Population'
FROM (
    SELECT 
        Gender,
        Heart_Attack,
        CASE
            WHEN Age <= 15 THEN 'Adolescent'
            WHEN Age BETWEEN 16 AND 24 THEN 'Young Adult'
            WHEN Age BETWEEN 25 AND 39 THEN 'Middle Age Adult'
            WHEN Age BETWEEN 40 AND 59 THEN 'Adult'
            ELSE 'Senior'
        END AS age_group,
        CASE
            WHEN Blood_Pressure < 120 THEN 'Healthy'
            WHEN Blood_Pressure BETWEEN 120 AND 129 THEN 'Elevated'
            WHEN Blood_Pressure BETWEEN 130 AND 139 THEN 'Stage 1 Hypertension'
            WHEN Blood_Pressure BETWEEN 140 AND 180 THEN 'Stage 2 Hypertension'
            ELSE 'Hypertensive Crisis'
        END AS bp_group,
        CASE
            WHEN BMI < 18.5 THEN 'Underweight'
            WHEN BMI BETWEEN 18.5 AND 24.9 THEN 'Normal'
            WHEN BMI BETWEEN 25 AND 29.9 THEN 'Overweight'
            WHEN BMI BETWEEN 30 AND 34.9 THEN 'Obese Class 1'
            WHEN BMI BETWEEN 35 AND 39.9 THEN 'Obese Class 2'
            ELSE 'Obese Class 3'
        END AS bmi_group,
        CASE
            WHEN Cholesterol < 200 THEN 'Heart-Healthy'
            WHEN Cholesterol BETWEEN 200 AND 239 THEN 'At-Risk'
            WHEN Cholesterol BETWEEN 240 AND 279 THEN 'Dangerous'
            ELSE 'Very High'
        END AS cholesterol_group
    FROM heart_attack_china_youth_vs_adult
) AS subquery
GROUP BY Gender, Heart_Attack, age_group, bp_group, bmi_group, cholesterol_group
ORDER BY Gender, Heart_Attack, age_group, bp_group, bmi_group, cholesterol_group;
```

To analyze heart attack cases based on all the factors such as BMI, gender, blood pressure (BP) group, cholesterol levels, and age group, we need a more direct approach that focuses on analyzing the heart attack cases in relation to these factors. Specifically, we want to look at the impact of each factor on the likelihood of having a heart attack, and also analyze the distribution of these factors among people with and without heart attacks.

This query tells us what percentage of people within a specific group (age, gender, BMI, blood pressure, cholesterol) have had a heart attack. We can easily see which categories (age, gender, BMI, etc.) have the highest and lowest rates of heart attack. By looking at the heart attack rate in relation to different risk factors, you can infer which combinations of factors (e.g., high BMI + high cholesterol) are associated with a higher likelihood of heart attack

**% of Heart Attack in Group**: This is the percentage of people who have had a heart attack within each group. It shows how much of the group (based on age, BMI, blood pressure, or cholesterol) has experienced a heart attack.

**% of Total Population**: This is the percentage of each group relative to the total population of individuals in the database

### Statistical Testing 
If you're looking for deeper insights, consider running statistical tests (e.g., chi-square tests, logistic regression) to confirm which factors are significantly associated with heart attacks.

At a later time, we will use **Visualization Tools** to better understand impact of each risk factor.

This query resulted in over **2,245 records**. Below is an illustration of some of the data queried.

| Gender | Heart_Attack | age_group | bp_group | bmi_group | cholesterol_group | Confirmed Heart Attack Count | % of Heart Attack in Group | % of Total Population |
| ------ | ------------ | --------- | -------- | --------- | ----------------- | ---------------------------- | -------------------------- | --------------------- |
| Female | No | Adolescent | Elevated | Normal | At-Risk | 0 | 0.00 | 0.13 |
| Female | No | Adolescent | Elevated | Normal | Dangerous | 0 | 0.00 | 0.06 |
| Female | No | Adolescent | Elevated | Normal | Heart-Healthy | 0 | 0.00 | 0.20 |
| Female | No | Adolescent | Elevated | Normal | Very High | 0 | 0.00 | 0.02 |
| Female | No | Adolescent | Elevated | Obese Class 1 | At-Risk | 0 | 0.00 | 0.02 |
| Female | No | Adolescent | Elevated | Obese Class 3 | Heart-Healthy | 0 | 0.00 | 0.00 |
| Female | No | Adolescent | Elevated | Obese Class 3 | Very High | 0 | 0.00 | 0.00 |
| Female | No | Adolescent | Elevated | Overweight | At-Risk | 0 | 0.00 | 0.08 |
| Female | No | Adolescent | Elevated | Overweight | Dangerous | 0 | 0.00 | 0.04 |
| Female | No | Adolescent | Elevated | Overweight | Heart-Healthy | 0 | 0.00 | 0.12 |
| Female | No | Adolescent | Elevated | Overweight | Very High | 0 | 0.00 | 0.02 |
| Female | No | Adolescent | Elevated | Underweight | At-Risk | 0 | 0.00 | 0.02 |
| Female | No | Adolescent | Elevated | Underweight | Dangerous | 0 | 0.00 | 0.01 |
| Female | No | Adolescent | Elevated | Underweight | Heart-Healthy | 0 | 0.00 | 0.04 |
| Female | No | Adolescent | Elevated | Underweight | Very High | 0 | 0.00 | 0.01 |
| Female | No | Adolescent | Healthy | Normal | At-Risk | 0 | 0.00 | 0.24 |
| Female | No | Adolescent | Healthy | Normal | Dangerous | 0 | 0.00 | 0.13 |
| Female | No | Adolescent | Healthy | Normal | Heart-Healthy | 0 | 0.00 | 0.41 |
| Female | No | Adolescent | Healthy | Normal | Very High | 0 | 0.00 | 0.06 |
| Female | No | Adolescent | Healthy | Obese Class 1 | At-Risk | 0 | 0.00 | 0.03 |
| Female | No | Adolescent | Healthy | Obese Class 1 | Dangerous | 0 | 0.00 | 0.02 |
| Female | No | Adolescent | Healthy | Obese Class 1 | Heart-Healthy | 0 | 0.00 | 0.06 |
| Female | No | Adolescent | Healthy | Obese Class 1 | Very High | 0 | 0.00 | 0.01 |
| Male | Yes | Middle Age Adult | Stage 2 Hypertension | Overweight | At-Risk | 44 | 100.00 | 0.02 |
| Male | Yes | Middle Age Adult | Stage 2 Hypertension | Overweight | Dangerous | 23 | 100.00 | 0.01 |
| Male | Yes | Middle Age Adult | Stage 2 Hypertension | Overweight | Heart-Healthy | 75 | 100.00 | 0.03 |
| Male | Yes | Middle Age Adult | Stage 2 Hypertension | Overweight | Very High | 11 | 100.00 | 0.00 |
| Male | Yes | Middle Age Adult | Stage 2 Hypertension | Underweight | At-Risk | 15 | 100.00 | 0.01 |
| Male | Yes | Middle Age Adult | Stage 2 Hypertension | Underweight | Dangerous | 6 | 100.00 | 0.00 |
| Male | Yes | Middle Age Adult | Stage 2 Hypertension | Underweight | Heart-Healthy | 20 | 100.00 | 0.01 |
| Male | Yes | Middle Age Adult | Stage 2 Hypertension | Underweight | Very High | 1 | 100.00 | 0.00 |
| Male | Yes | Young Adult | Elevated | Normal | At-Risk | 95 | 100.00 | 0.04 |
| Male | Yes | Young Adult | Elevated | Normal | Dangerous | 48 | 100.00 | 0.02 |
| Male | Yes | Young Adult | Elevated | Normal | Heart-Healthy | 173 | 100.00 | 0.06 |
| Male | Yes | Young Adult | Elevated | Normal | Very High | 18 | 100.00 | 0.01 |
| Male | Yes | Young Adult | Elevated | Obese Class 1 | At-Risk | 4 | 100.00 | 0.00 |
| Male | Yes | Young Adult | Elevated | Obese Class 1 | Dangerous | 3 | 100.00 | 0.00 |
| Male | Yes | Young Adult | Elevated | Obese Class 1 | Heart-Healthy | 22 | 100.00 | 0.01 |
| Other | No | Adolescent | Healthy | Underweight | Very High | 0 | 0.00 | 0.00 |
| Other | No | Adolescent | Hypertensive Crisis | Normal | At-Risk | 0 | 0.00 | 0.00 |
| Other | No | Adolescent | Hypertensive Crisis | Normal | Dangerous | 0 | 0.00 | 0.00 |
| Other | No | Adolescent | Hypertensive Crisis | Normal | Heart-Healthy | 0 | 0.00 | 0.00 |
| Other | No | Adolescent | Hypertensive Crisis | Normal | Very High | 0 | 0.00 | 0.00 |
| Other | No | Adolescent | Hypertensive Crisis | Obese Class 1 | At-Risk | 0 | 0.00 | 0.00 |
| Other | No | Adolescent | Hypertensive Crisis | Overweight | At-Risk | 0 | 0.00 | 0.00 |
| Other | No | Adolescent | Hypertensive Crisis | Overweight | Dangerous | 0 | 0.00 | 0.00 |
| Other | No | Adolescent | Hypertensive Crisis | Overweight | Heart-Healthy | 0 | 0.00 | 0.00 |
| Other | No | Adolescent | Hypertensive Crisis | Overweight | Very High | 0 | 0.00 | 0.00 |
| Other | No | Adolescent | Hypertensive Crisis | Underweight | At-Risk | 0 | 0.00 | 0.00 |
| Other | No | Adolescent | Hypertensive Crisis | Underweight | Heart-Healthy | 0 | 0.00 | 0.00 |

> Please see the full .CSV reference file containing the complete data set is [here](/Analysis/aggregate_data_analysis_by_groups.csv) in the [Analysis](Analysis) folder.

We can utilize Python to `Describe` the data in the csv file and provide summary statistics for all columns, including counts, unique values, means, and standard deviations, depending on the data type of each column.

```
# Python Code
import pandas as pd

# Load the uploaded CSV file to analyze its content
file_path = 'Analysis/aggregate_data_analysis_by_groups.csv'
data = pd.read_csv(file_path)

# Display all rows and columns
pd.set_option('display.max_rows', None)
pd.set_option('display.max_columns', None)

# Display the first few rows for a quick overview
print("First few rows of the dataset:")
print(data.head())

# Display a summary of the dataset
print("\nDataset Info:")
data.info()

# Display descriptive statistics for numerical and categorical columns
print("\nDescriptive Statistics for Numerical Columns:")
print(data.describe())

print("\nDescriptive Statistics for All Columns:")
print(data.describe(include='all'))

# Analyze null values to understand data completeness
print("\nMissing Values Count:")
print(data.isnull().sum())

# Display unique values for each column to understand categorical variables
print("\nUnique Values per Column:")
for column in data.columns:
    unique_values = data[column].unique()
    print(f"{column}: {len(unique_values)} unique values")

# Save descriptive statistics to a CSV file for a detailed view
data.describe(include='all').to_csv('full_descriptive_statistics.csv')
```

|        | Gender   | Heart_Attack   | age_group   | bp_group   | bmi_group   | cholesterol_group   |   Confirmed Heart Attack Count |   % of Heart Attack in Group |   % of Total Population |
|--------|----------|----------------|-------------|------------|-------------|---------------------|--------------------------------|------------------------------|-------------------------|
| count  | 2245     | 2245           | 2245        | 2245       | 2245        | 2245                |                      2245      |                    2245      |            2245         |
| unique | 3        | 2              | 4           | 5          | 6           | 4                   |                       nan      |                     nan      |             nan         |
| top    | Female   | No             | Adult       | Healthy    | Normal      | Heart-Healthy       |                       nan      |                     nan      |             nan         |
| freq   | 837      | 1271           | 601         | 522        | 463         | 621                 |                       nan      |                     nan      |             nan         |
| mean   | nan      | nan            | nan         | nan        | nan         | nan                 |                        14.2356 |                      43.3853 |               0.0440579 |
| std    | nan      | nan            | nan         | nan        | nan         | nan                 |                        52.7183 |                      49.5716 |               0.144743  |
| min    | nan      | nan            | nan         | nan        | nan         | nan                 |                         0      |                       0      |               0         |
| 25%    | nan      | nan            | nan         | nan        | nan         | nan                 |                         0      |                       0      |               0         |
| 50%    | nan      | nan            | nan         | nan        | nan         | nan                 |                         0      |                       0      |               0.01      |
| 75%    | nan      | nan            | nan         | nan        | nan         | nan                 |                         5      |                     100      |               0.02      |
| max    | nan      | nan            | nan         | nan        | nan         | nan                 |                       809      |                     100      |               2.22      |

|    | Column                       |   MissingValues |
|----|------------------------------|-----------------|
|  0 | Gender                       |               0 |
|  1 | Heart_Attack                 |               0 |
|  2 | age_group                    |               0 |
|  3 | bp_group                     |               0 |
|  4 | bmi_group                    |               0 |
|  5 | cholesterol_group            |               0 |
|  6 | Confirmed Heart Attack Count |               0 |
|  7 | % of Heart Attack in Group   |               0 |
|  8 | % of Total Population        |               0 |

The data table offers insights into heart attack prevalence across different demographic and health-related groups.

The age group distribution shows that adults are the most common demographic, with 601 individuals categorized in this group. In terms of health metrics, the most frequent categories are "Healthy" for blood pressure (BP) and "Normal" for BMI, which may indicate a relatively health-conscious demographic or potential underreporting of abnormal metrics. Similarly, the "Heart-Healthy" cholesterol group is the most prevalent, with 621 individuals, suggesting some level of effective cholesterol management among the participants. However, further analysis is needed to determine whether these health metrics correlate with lower heart attack risks.

The confirmed heart attack metrics reveal significant variability. On average, there are 14.24 confirmed heart attacks per group, with a standard deviation of 52.72. The maximum count of 809 heart attacks within a single group highlights the presence of high-risk subpopulations. Interestingly, while the average percentage of heart attack prevalence in groups is 43.39%, the median is 0%, suggesting that many groups report no confirmed heart attack cases. This indicates that heart attack occurrences are concentrated within specific high-risk subgroups, which merit further investigation.

In terms of population representation, no single group dominates the dataset, as the maximum representation of a single group accounts for only 2.22% of the total population. This suggests a relatively diverse dataset that captures a wide range of demographic and health profiles.

We can also look into the occurrence of heart attacks, across different age groups to identify correlations, if any, there are with smoking and heart attacks.

```
SELECT 
    age_group,
    Heart_Attack AS 'Heart Attack',
    Gender,
    Smoking,
    COUNT(*) AS 'Number of Patients',
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) AS 'Percentage of Total Patients',
        ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM heart_attack_china_youth_vs_adult), 2) AS '% of Total Population'
FROM (
    SELECT 
        Gender,
        Heart_Attack,
        Smoking,
        CASE
            WHEN Age <= 15 THEN 'Adolescent'
            WHEN Age BETWEEN 16 AND 24 THEN 'Young Adult'
            WHEN Age BETWEEN 25 AND 39 THEN 'Middle Age Adult'
            WHEN Age BETWEEN 40 AND 59 THEN 'Adult'
            ELSE 'Senior'
        END AS age_group
    FROM heart_attack_china_youth_vs_adult
) AS smoking_and_age_group
GROUP BY age_group, Heart_Attack, Gender, Smoking
ORDER BY age_group, Heart_Attack, Gender, Smoking;
```

Below is the distribution of patients who have had heart attacks based on their smoking status, age group, and gender, and calculates the percentage of these individuals compared to the overall dataset.

| Age Group        | Heart Attack | Gender | Smoking | Number of Patients | Percentage of Total Patients | % of Total Population |
|------------------|--------------|--------|---------|---------------------|------------------------------|-----------------------|
| Adolescent       | No           | Female | No      | 6464                | 2.39                         | 2.39                  |
| Adolescent       | No           | Female | Yes     | 2808                | 1.04                         | 1.04                  |
| Adolescent       | No           | Male   | No      | 6731                | 2.49                         | 2.49                  |
| Adolescent       | No           | Male   | Yes     | 2944                | 1.09                         | 1.09                  |
| Adolescent       | No           | Other  | No      | 540                 | 0.20                         | 0.20                  |
| Adolescent       | No           | Other  | Yes     | 219                 | 0.08                         | 0.08                  |
| Adolescent       | Yes          | Female | No      | 888                 | 0.33                         | 0.33                  |
| Adolescent       | Yes          | Female | Yes     | 434                 | 0.16                         | 0.16                  |
| Adolescent       | Yes          | Male   | No      | 873                 | 0.32                         | 0.32                  |
| Adolescent       | Yes          | Male   | Yes     | 377                 | 0.14                         | 0.14                  |
| Adolescent       | Yes          | Other  | No      | 93                  | 0.03                         | 0.03                  |
| Adolescent       | Yes          | Other  | Yes     | 26                  | 0.01                         | 0.01                  |
| Adult            | No           | Female | No      | 33152               | 12.28                        | 12.28                 |
| Adult            | No           | Female | Yes     | 14320               | 5.30                         | 5.30                  |
| Adult            | No           | Male   | No      | 33245               | 12.31                        | 12.31                 |
| Adult            | No           | Male   | Yes     | 14402               | 5.33                         | 5.33                  |
| Adult            | No           | Other  | No      | 2805                | 1.04                         | 1.04                  |
| Adult            | No           | Other  | Yes     | 1193                | 0.44                         | 0.44                  |
| Adult            | Yes          | Female | No      | 4589                | 1.70                         | 1.70                  |
| Adult            | Yes          | Female | Yes     | 1954                | 0.72                         | 0.72                  |
| Adult            | Yes          | Male   | No      | 4547                | 1.68                         | 1.68                  |
| Adult            | Yes          | Male   | Yes     | 1915                | 0.71                         | 0.71                  |
| Adult            | Yes          | Other  | No      | 356                 | 0.13                         | 0.13                  |
| Adult            | Yes          | Other  | Yes     | 147                 | 0.05                         | 0.05                  |
| Middle Age Adult | No           | Female | No      | 25311               | 9.37                         | 9.37                  |
| Middle Age Adult | No           | Female | Yes     | 10748               | 3.98                         | 3.98                  |
| Middle Age Adult | No           | Male   | No      | 25111               | 9.30                         | 9.30                  |
| Middle Age Adult | No           | Male   | Yes     | 10738               | 3.98                         | 3.98                  |
| Middle Age Adult | No           | Other  | No      | 2060                | 0.76                         | 0.76                  |
| Middle Age Adult | No           | Other  | Yes     | 901                 | 0.33                         | 0.33                  |
| Middle Age Adult | Yes          | Female | No      | 3282                | 1.22                         | 1.22                  |
| Middle Age Adult | Yes          | Female | Yes     | 1396                | 0.52                         | 0.52                  |
| Middle Age Adult | Yes          | Male   | No      | 3299                | 1.22                         | 1.22                  |
| Middle Age Adult | Yes          | Male   | Yes     | 1388                | 0.51                         | 0.51                  |
| Middle Age Adult | Yes          | Other  | No      | 309                 | 0.11                         | 0.11                  |
| Middle Age Adult | Yes          | Other  | Yes     | 127                 | 0.05                         | 0.05                  |
| Young Adult      | No           | Female | No      | 14887               | 5.51                         | 5.51                  |
| Young Adult      | No           | Female | Yes     | 6515                | 2.41                         | 2.41                  |
| Young Adult      | No           | Male   | No      | 14809               | 5.48                         | 5.48                  |
| Young Adult      | No           | Male   | Yes     | 6334                | 2.35                         | 2.35                  |
| Young Adult      | No           | Other  | No      | 1240                | 0.46                         | 0.46                  |
| Young Adult      | No           | Other  | Yes     | 564                 | 0.21                         | 0.21                  |
| Young Adult      | Yes          | Female | No      | 1965                | 0.73                         | 0.73                  |
| Young Adult      | Yes          | Female | Yes     | 878                 | 0.33                         | 0.33                  |
| Young Adult      | Yes          | Male   | No      | 2061                | 0.76                         | 0.76                  |
| Young Adult      | Yes          | Male   | Yes     | 807                 | 0.30                         | 0.30                  |
| Young Adult      | Yes          | Other  | No      | 175                 | 0.06                         | 0.06                  |
| Young Adult      | Yes          | Other  | Yes     | 73                  | 0.03                         | 0.03                  |



Based on this data, the percentage of patients experiencing a heart attack is higher among those who smoke, particularly for both genders and age groups. **Adults and Middle-Aged Adults** group show a significantly higher number of heart attack cases, with smoking as a contributing factor. Although young adults show slightly lower number of heart attacks in comparison to older age groups, smoking still plays a notable role.

## Does Smoking Impact? 

**Yes.** Smoking increases the likelihood of a heart attack across all age groups and genders. The "Yes" smoking category has significantly fewer "No" heart attack cases. Overall, males tend to have higher patient numbers, but the percentage of patients with heart attacks is somewhat comparable between genders, particularly for adults and middle-aged adults. Smoking is consistently linked with higher heart attack occurrences.

We will now look the Avg BMI, Avg age based on gender, heart attack status, and smoking status. The query will represent unique combinations for individuals in that group. 

| Gender | Heart Attack | Smoking | Avg BMI | Avg Age |
|--------|--------------|---------|---------|---------|
| Female | No           | No      | 24      | 35.55   |
| Female | No           | Yes     | 24.01   | 35.51   |
| Male   | No           | No      | 24      | 35.51   |
| Male   | No           | Yes     | 24      | 35.51   |
| Male   | Yes          | Yes     | 23.99   | 35.75   |
| Female | Yes          | No      | 24      | 35.69   |
| Male   | Yes          | No      | 24.01   | 35.56   |
| Other  | No           | No      | 23.95   | 35.64   |
| Female | Yes          | Yes     | 24.06   | 35.35   |
| Other  | No           | Yes     | 23.95   | 35.46   |
| Other  | Yes          | No      | 24.12   | 34.72   |
| Other  | Yes          | Yes     | 23.97   | 35.41   |

**Males** with heart attacks who smoke have a slightly lower BMI (23.99), but their age is higher on average compared to non-smokers (35.75 vs. 35.51).

**Female** patients with a heart attack have a slightly higher average BMI (24.06) compared to those without (24), but their average age is fairly consistent across both groups (35.35 vs. 35.55).

This dataset suggests that while smoking impacts health negatively, other factors like BMI and age may not be significantly altered in this specific dataset.

---

# Conclusion

The data reveals that heart attack rates are highest among adults and middle-aged adults, with significant variability in the occurrence of heart attacks across different groups. Smoking is strongly associated with increased heart attack risk, with those who smoke showing notably fewer non-heart attack cases. While the majority of participants report healthy blood pressure and normal BMI, further analysis is needed to understand how these metrics correlate with heart attack prevalence. The dataset also highlights high-risk subgroups, with one group reporting as many as 809 heart attack cases, suggesting that heart attacks are concentrated in specific populations. Despite a health-conscious demographic with a high proportion of heart-healthy cholesterol levels, smoking remains a key factor in heart attack occurrences across genders and age groups. Further investigation into these high-risk subgroups, along with targeted health interventions, is necessary to mitigate cardiovascular risks effectively.