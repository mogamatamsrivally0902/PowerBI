# Loan Performance Data Dashboard

### Dashboard Link

[https://app.powerbi.com/links/f8dc8MnA-1?ctid=20f3b949-2903-4d24-9d01-ad8df676e397&pbi_source=linkShare](https://app.powerbi.com/links/f8dc8MnA-1?ctid=20f3b949-2903-4d24-9d01-ad8df676e397&pbi_source=linkShare)

---

## Problem Statement

This dashboard was developed to analyze **loan portfolio performance and default behavior** using historical loan data. The objective is to help stakeholders understand how loan defaults and loan exposure vary across **borrower demographics, employment types, credit score segments, and time**.

The dashboard enables data-driven decision-making by:

* Monitoring overall loan portfolio health
* Identifying high-risk borrower segments
* Tracking default trends over time
* Supporting credit risk assessment and financial analysis


---

## Steps Followed

### Step 1: Data Ingestion

* Loaded loan performance data into Power BI Desktop from a CSV file.

### Step 2: Data Profiling & Validation

* Opened **Power Query Editor**.
* Enabled **Column Distribution**, **Column Quality**, and **Column Profile** under the View tab.
* Switched column profiling to analyze the **entire dataset** instead of the default 1,000 rows.

### Step 3: Data Cleaning & Preparation

* Validated data types for numerical, categorical, and date fields.
* Checked for missing or null values in key financial fields such as loan amount, income, and default flag.
* Blank or non-applicable values were excluded implicitly during aggregations and averages.

### Step 4: Feature Engineering

* Created derived fields such as:

  * **Age Groups** (for demographic segmentation)
  * **Credit Score Bins** (for credit risk analysis)
  * **Year** (for time-based analysis)

### Step 5: Data Modeling

* Built a single fact-style loan table suitable for analytical reporting.
* Created **three dedicated measure tables** to logically organize KPIs and calculations.

### Step 6: DAX Measures Creation

* Developed measures to calculate portfolio exposure, default rates, averages, medians, and time intelligence metrics (YoY, YTD).

### Step 7: Report Design

* Designed an interactive single-page dashboard.
* Added KPI cards, charts, and slicers for dynamic analysis.
* Applied a consistent theme for professional presentation.

### Step 8: Publishing

* Published the report to **Power BI Service** for sharing and access.

---

## DAX Measures Overview

All key performance indicators (KPIs) and analytical metrics were created using **DAX measures**, organized into three dedicated measure tables for clarity and maintainability.

---

## Measure Table 1 – Portfolio & Risk Segmentation (DAX)

**1. Average Loan by Age Group**

```DAX
Average Loan by Age Group =  
AVERAGEX(
    VALUES('Loan_default'[Age Groups]),
    AVERAGE('Loan_default'[LoanAmount])
)
```

**2. Default Rate by Employment Type (%)**

```DAX
Default rate by employement type =  
VAR totalrecords = COUNTROWS(ALL('Loan_default'))
VAR Defaultcases = 
    COUNTROWS(
        FILTER('Loan_default', 'Loan_default'[Default] = TRUE())
    )
RETURN
CALCULATE(
    DIVIDE(Defaultcases, totalrecords),
    ALLEXCEPT('Loan_default', 'Loan_default'[EmploymentType])
) * 100
```

**3. Default Rate by Year (%)**

```DAX
Default Rate by Year = 
VAR totalloans = 
    CALCULATE(
        COUNTROWS('Loan_default'),
        ALLEXCEPT('Loan_default', 'Loan_default'[Year])
    )
VAR default = 
    CALCULATE(
        COUNTROWS(
            FILTER('Loan_default', 'Loan_default'[Default] = TRUE())
        ),
        ALLEXCEPT('Loan_default', 'Loan_default'[Year])
    )
RETURN
DIVIDE(default, totalloans) * 100
```

**4. Loan Amount by Purpose**

```DAX
Loan Amount by Purpose = 
SUMX(
    FILTER('Loan_default', NOT(ISBLANK('Loan_default'[LoanAmount]))),
    'Loan_default'[LoanAmount]
)
```

**5. Average Income by Employment Type**

```DAX
Average Income by Employement Type = 
CALCULATE(
    AVERAGE('Loan_default'[Income]),
    ALLEXCEPT('Loan_default', 'Loan_default'[EmploymentType])
)
```

---

## Measure Table 2 – Credit & Demographic Analysis (DAX)

**1. Average Loan Amount (High Credit Score)**

```DAX
Average Loan Amount (High Credit Score) = 
AVERAGEX(
    FILTER('Loan_default', 'Loan_default'[credit Score Bins] = "High"),
    'Loan_default'[LoanAmount]
)
```

**2. Loans by Education Type**

```DAX
Loans By Education Type = 
COUNTROWS(
    FILTER('Loan_default', NOT(ISBLANK('Loan_default'[LoanID])))
)
```

**3. Median Loan Amount by Credit Score Bins**

```DAX
Median By Credit Score Bins = 
MEDIANX('Loan_default', 'Loan_default'[LoanAmount])
```

**4. Total Loan Amount by Credit Bins (Adults)**

```DAX
Total Loan (Credit Bins) = 
CALCULATE(
    SUM('Loan_default'[LoanAmount]),
    'Loan_default'[Age Groups] = "Adults",
    ALLEXCEPT(
        'Loan_default',
        'Loan_default'[Age],
        'Loan_default'[Age Groups],
        'Loan_default'[CreditScore],
        'Loan_default'[credit Score Bins]
    )
)
```

**5. Total Loan Amount (Middle Age Adults)**

```DAX
Total Loan(Middle Age Adults) = 
SUMX(
    FILTER('Loan_default', 'Loan_default'[Age Groups] = "Middle Age Adults"),
    'Loan_default'[LoanAmount]
)
```

---

## Measure Table 3 – Time Intelligence Metrics (DAX)

**1. YoY Default Loans Change (%)**

```DAX
YOY Default Loans Change = 
DIVIDE(
    CALCULATE(
        COUNTROWS(FILTER('Loan_default', 'Loan_default'[Default] = TRUE())),
        'Loan_default'[Year] = YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))
    ) - 
    CALCULATE(
        COUNTROWS(FILTER('Loan_default', 'Loan_default'[Default] = TRUE())),
        'Loan_default'[Year] = YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY])) - 1
    ),
    CALCULATE(
        COUNTROWS(FILTER('Loan_default', 'Loan_default'[Default] = TRUE())),
        'Loan_default'[Year] = YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY])) - 1
    ),
    0
) * 100
```

**2. YoY Loan Amount Change (%)**

```DAX
YOY Loan Amount Change = 
DIVIDE(
    CALCULATE(
        SUM('Loan_default'[LoanAmount]),
        'Loan_default'[Year] = YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY]))
    ) - 
    CALCULATE(
        SUM('Loan_default'[LoanAmount]),
        'Loan_default'[Year] = YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY])) - 1
    ),
    CALCULATE(
        SUM('Loan_default'[LoanAmount]),
        'Loan_default'[Year] = YEAR(MAX('Loan_default'[Loan_Date_DD_MM_YYYY])) - 1
    ),
    0
) * 100
```

**3. YTD Loan Amount**

```DAX
YTD Loan Amount = 
CALCULATE(
    SUM('Loan_default'[LoanAmount]),
    DATESYTD('Loan_default'[Loan_Date_DD_MM_YYYY].[Date]),
    ALLEXCEPT(
        'Loan_default',
        'Loan_default'[credit Score Bins],
        'Loan_default'[MaritalStatus]
    )
)
```

---

### Measure Table 2 – Credit & Demographic Analysis

* **Average Loan Amount (High Credit Score)**
* **Loans by Education Type**
* **Median Loan Amount by Credit Score Bins**
* **Total Loan Amount by Credit Bins (Adults)**
* **Total Loan Amount (Middle Age Adults)**

These KPIs enable analysis of borrowing behavior and exposure across creditworthiness and age segments.

---

### Measure Table 3 – Time Intelligence Metrics

* **YoY Default Loans Change (%)**
* **YoY Loan Amount Change (%)**
* **YTD Loan Amount**

These measures provide trend-based insights into portfolio growth and default risk over time.

---

## Visuals Used

* **KPI Cards**: Total Loan Amount, Default Rate, YTD Loan Amount
* **Bar Charts**:

  * Loan Amount by Purpose
  * Default Rate by Employment Type
* **Line Charts**:

  * Default Rate by Year
  * YoY Loan Amount Change
* **Slicers**:

  * Age Groups
  * Employment Type
  * Credit Score Bins
  * Marital Status
  * Year

---

## Key Insights

### 1. Portfolio & Exposure

* Loan exposure varies significantly across age groups, with adults and middle-aged borrowers accounting for the largest share of total loan amounts.
* Loan purpose analysis highlights concentration in specific borrowing categories, indicating potential exposure clustering.

### 2. Default Risk Analysis

* Default rates differ notably by employment type, indicating that employment stability plays a critical role in credit risk.
* Year-over-year default trends help identify periods of increased portfolio stress.

### 3. Credit Score Behavior

* Borrowers with high credit scores generally show different average and median loan amounts compared to lower credit score segments.
* Median-based analysis provides a more stable view of exposure by reducing the effect of outliers.

### 4. Time-Based Trends

* YoY loan amount changes reflect shifts in lending activity and portfolio growth.
* YTD loan amount provides a near real-time view of current-year lending performance.

---

## Business Recommendations

* Strengthen risk monitoring for employment types exhibiting higher default rates.
* Use credit score bin analysis to optimize lending strategies and pricing.
* Track YoY default trends as early warning indicators for portfolio deterioration.
* Apply demographic segmentation to tailor credit policies and marketing strategies.

---

## Conclusion

This Power BI dashboard demonstrates strong analytical capabilities in data modeling, DAX calculations,
segmentation analysis, and business insight generation.The project effectively supports decision-making across data analytics,
credit risk, and financial analysis functions and is suitable for portfolio presentation, interviews, and real-world business scenarios.
