# STD Health Analysis

An interactive Power BI dashboard analysing four sexually transmitted diseases (HIV/AIDS, Hepatitis B, HPV and Syphilis) across 14 countries and 5 world regions from 2015 to 2025.

![Sexually Transmitted Diseases Dashboard showing Syphilis](Powerbi_Screenshot/Syphilis.png)

## Project overview

The dashboard is a single report page built around one disease at a time. A dropdown at the top right switches between the four diseases, and every visual on the page updates to match. It is designed to answer four questions:

* How have new cases and deaths changed between 2015 and 2025?
* How does the average prevalence rate compare across regions?
* How does the average incidence rate compare across regions?
* Which countries does the data cover for each disease?

## Repository structure

```
std-health-analysis/
├── STD_dashboard.pbix              Power BI report
├── dataset/
│   ├── Health Dataset.csv          Main dataset (819 rows, 15 columns)
│   └── Health Image Dataset.csv    Disease image lookup table (4 rows)
├── Powerbi_Screenshot/
│   ├── HIV_AIDS.png                Dashboard filtered to HIV/AIDS
│   ├── HepatitisB.png              Dashboard filtered to Hepatitis B
│   ├── HPV.png                     Dashboard filtered to HPV
│   ├── Syphilis.png                Dashboard filtered to Syphilis
│   └── model.png                   Data model view
└── README.md
```

## Data source

The dataset was sourced from [Isaac Emmanuel](https://www.youtube.com/watch?v=pqScQQua0ls) on YouTube. It is a practice dataset, so the figures are for learning and portfolio purposes and should not be read as real epidemiological statistics.

## The data

### [Health Dataset.csv](dataset/Health%20Dataset.csv)

Each row is one combination of year, country, disease, age group and gender. The file has 819 rows, with no missing values and no duplicate rows.

| Column | Type in Power BI | Description |
| --- | --- | --- |
| `Year` | Whole number | 2015 to 2025 |
| `Country` | Text | Australia, Brazil, Canada, China, France, Germany, Ghana, India, Japan, Kenya, Nigeria, South Africa, UK, USA |
| `Region` | Text | Africa, Americas, Asia, Europe, Oceania |
| `Disease` | Text | HIV/AIDS, Hepatitis B, HPV, Syphilis |
| `Age_Group` | Text | 15 to 24, 25 to 49, 50+ |
| `Gender` | Text | Female, Male |
| `Population` | Whole number | Population of the group |
| `Total_Cases` | Whole number | Total recorded cases |
| `New_Cases` | Whole number | New cases in that year |
| `Deaths` | Whole number | Recorded deaths |
| `Prevalence_Rate` | Percentage | Share of the population living with the disease |
| `Incidence_Rate` | Whole number | Rate of new cases |
| `Tested_Population` | Whole number | Number of people tested |
| `Vaccinated_Population` | Whole number | Number of people vaccinated (0 where no vaccine applies) |
| `Treatment_Coverage` | Percentage | Share of cases receiving treatment |

### [Health Image Dataset.csv](dataset/Health%20Image%20Dataset.csv)

A lookup table with one row per disease (`Disease`, `Image URL`). Each URL points to a custom disease image hosted on Google Drive, which the dashboard displays when that disease is selected.

## Data preparation (Power Query)

Both CSV files were imported into Power Query and prepared with the following steps.

**Health Dataset**
1. Promoted the first row to column headers.
2. Set data types for all 15 columns: whole number for counts, rates and year; text for categories; percentage for `Prevalence_Rate` and `Treatment_Coverage`.
3. Standardised the percentage columns. In the raw CSV, 96 rows store `Prevalence_Rate` and `Treatment_Coverage` without a percent sign (for example `3` instead of `3.00%`). Power Query reads `3.00%` as 0.03 but `3` as 3, so those rows loaded 100 times too large and inflated the regional averages. A transform step divides any value of 1 or more by 100, so every row uses the same scale.

```powerquery
#"Fixed Percentage Scale" = Table.TransformColumns(#"Changed Type", {
    {"Prevalence_Rate", each if _ <> null and _ >= 1 then _ / 100 else _, Percentage.Type},
    {"Treatment_Coverage", each if _ <> null and _ >= 1 then _ / 100 else _, Percentage.Type}
})
```

**Health Image Dataset**
1. Promoted the first row to column headers.
2. Set `Disease` and `Image URL` as text.

No rows were removed or filtered, as the data contained no blanks or duplicates. The raw CSV in this repository is left unchanged, so the cleaning step can be seen and repeated in Power Query.

## Data model

![Data model](Powerbi_Screenshot/model.png)

The model has two tables joined by a single relationship:

* `Health Image Dataset[Disease]` (one) to `Health Dataset[Disease]` (many)
* Single direction filtering, from the image table to the main table

This lets the disease slicer, which uses the `Disease` column from the image table, filter both the image visual and every chart built on the main dataset.

## Calculations

The report does not use DAX measures or calculated columns. All values come from Power BI's built-in **Average** aggregation applied directly in each visual:

| Visual | Calculation |
| --- | --- |
| New cases and deaths trend | Average of `New_Cases` (shown as Avg NC) and average of `Deaths` (shown as Avg DC) by `Year` |
| Prevalence rate by region | Average of `Prevalence_Rate` by `Region` |
| Incidence rate by region | Average of `Incidence_Rate` by `Region` |

## Dashboard

| Visual | Type | What it shows |
| --- | --- | --- |
| Disease image | Image custom visual (CloudScope) | A custom image and short description of the selected disease |
| New Cases and Death Trends Over Time | Line chart | Average new cases and deaths for each year, in year order |
| Prevalence Rate Across Regions | Bar chart | Average prevalence rate for each region, sorted highest first |
| Global Disease Distribution | Shape map (orthographic globe) | The countries with data for the selected disease |
| Incidence Rate Across Regions | Column chart styled as a lollipop chart | Average incidence rate for each region, sorted highest first |
| Disease selector | Dropdown slicer | Filters the whole page by disease |

The page uses a custom background image and a muted grey and teal colour palette.

### Screenshots

| HIV/AIDS | Hepatitis B |
| --- | --- |
| ![HIV/AIDS view](Powerbi_Screenshot/HIV_AIDS.png) | ![Hepatitis B view](Powerbi_Screenshot/HepatitisB.png) |
| **HPV** | **Syphilis** |
| ![HPV view](Powerbi_Screenshot/HPV.png) | ![Syphilis view](Powerbi_Screenshot/Syphilis.png) |

## Limitations and next steps

* **Explicit measures.** Replacing the implicit averages with named DAX measures would make the calculations easier to reuse and document.
* **Uneven coverage.** The 50+ age group has only 61 rows, and the number of rows per country and per year varies, so averages for smaller groups should be read with care.

## Tools used

* Power BI Desktop
* Power Query
* Image custom visual by CloudScope (AppSource)
* Shape map visual

## How to use

1. Clone or download this repository.
2. Open `STD_dashboard.pbix` in Power BI Desktop.
3. The report points to the CSV files at a local path. If you want to refresh it, go to **Transform data > Data source settings** and change the source to the files in the `dataset` folder.
4. Use the dropdown at the top right to switch between diseases.

The disease images load from Google Drive, so an internet connection is needed for them to display.

## Author

**Orhmo**
GitHub: [Orhmo](https://github.com/Orhmo)
