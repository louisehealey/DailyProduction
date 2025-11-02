# 📈 Daily Production Report

This Power BI report provides a dynamic daily overview of completed units, offering clear insights into production trends across the calendar month. Users can interactively adjust the **Daily Production Goal** to support parameter testing and scenario planning.

The report calculates and visualizes:
- **Month-to-Date (MTD) Goals**
- **Actual MTD Performance**

These metrics help teams monitor progress and stay aligned with production targets.

## 🎯 Interactive Goal Setting

If today were **August 15, 2022** and the **Product Line Nova Core** was selected, the report would appear as follows:

<p align="center">
  <img src="https://github.com/louisehealey/DailyProduction/blob/main/CompletedUnitsByDay.gif">
</p>

Users can adjust the **Daily Production Goal** using the slider in the **Adjustable Goal (Daily)** visual. This value is represented by the yellow dotted goal line on the bar chart. Bars update in real time:
- ✅ **Green** when the goal is met or exceeded  
- ❌ **Red** when production falls short

This immediate visual feedback allows users to quickly assess daily performance.

<p align="center">
  <img src="https://github.com/louisehealey/DailyProduction/blob/main/AdjustableGoal%20(daily).png" width="300">
</p>

## 📊 KPI Snapshot: Completed Units Over Goal

The **radial gauge** visual (**Completed Units over Goal**) updates automatically alongside other KPIs:
- **MTD Actual**
- **MTD Goal**

These visuals offer a high-level snapshot of monthly production progress. The radial gauge calculates the ratio of actual units produced MTD to the MTD goal, providing a quick visual cue of performance status.

<p align="center">
  <img src="https://github.com/louisehealey/DailyProduction/blob/main/CompletedOverGoal.png" width="200" height="400">
</p>


## 🎨 Conditional Formatting Logic

To apply dynamic color formatting to columns based on goal achievement, use the following DAX measure in the Format panel:

```DAX
ColumnDeterminate = 
IF([AdjustableProductionGoal(m)] > [TotalUnits], "R", "G")
```
---
## Data Modeling:

### 📊 Generating a Date Table: 
Adding a date table is essential when working with time-based data. In our case, it ensures continuity by preventing gaps or inconsistencies caused by business days when no units are completed. Without a date table, the report would simply skip over those inactive days, making it difficult to accurately analyze trends or performance over time.
``` 
FiscalCalendar =

VAR EndDate = TODAY()
VAR StartDate = DATE(YEAR(EndDate) - 1, MONTH(EndDate), DAY(EndDate))
RETURN
ADDCOLUMNS (
    CALENDAR(StartDate, EndDate),
    "Year", YEAR([Date]),
    "Month", FORMAT([Date], "MMMM"),
    "Month Number", MONTH([Date]),
    "Quarter", "Q" & FORMAT(QUARTER([Date]), "0")
)
```
#### Dates in Scope
Accurately calculating the number of business days in a month requires excluding non-business days such as weekends and holidays. Below is a simple formula to exclude weekends . 
``` 
IsWorkDay =
VAR DayNumber = WEEKDAY([Date], 2)
RETURN
    IF(DayNumber <= 5, 1, 0)
``` 
Excluding holidays is also necessary when determing the days in scope. To capture holidays, you would need to load your companys fisical calendar into Power BI and add a `RELATED` or `LOOKUPVALUE` variable to pull in the additional dates you want to exclude. The calculated column below excludes both weekends and the holidays.
```
IsWorkday =
VAR DayNumber = WEEKDAY([Date], 2)
VAR IsWeekend = IF(DayNumber > 5, 1, 0)
VAR IsHoliday = IF([Date] IN VALUES(HolidayDates[HolidayDate]), 1, 0)
RETURN
    IF(IsWeekend = 0 && IsHoliday = 0, 1, 0)
```
### 📊 Generating a Table that Counts Completed Units
Loading tables that capture data related to the closure of jobs associated with finished units is a reliable method for tracking production output. This process typically involves importing transactional records, applying filters to isolate relevant action types, constraining the data to a specific date range, and removing unnecessary columns to optimize the data model's size and performance.

Once the data is extracted, loaded, and transformed, the following measures can be created to visualize completed units effectively. For refrence, the table that holds this data is named `CLOSED_JOBS`

#### Measures: 

The **Total Units** measure sums the total units completed. The `IF(ISBLANK())` statement ensures consistent numerical output across all dates, even when nothing is produced. Without it, the days where nothing is produced would return "Blank".
```
TotalUnits= 
VAR TotalQuantity =
    CALCULATE(
        ABS(SUM('CLOSED_JOBS'[quantity]))
    )
RETURN
    IF(ISBLANK(TotalQuantity), 0, TotalQuantity)
```


### 📊 Generating a Paremeter for the What-if Analysis
Generating an Adjustable Paremeter, in our case it's the **Adjusted Daily Goal**, is simple. Go to Modeling > New Paremeter > Numeric Range. This will return the values below

**The Table:**
```
AdjustedProductionGoal = GENERATESERIES(0, 300, 25)
```
**The Measure:**
```
AdjustedProductionGoal(m) = SELECTEDVALUE('Adjustable Production Goal'[Parameter], 75)

````
#### Measures:

The **MTD Goal** is a measure that multiplies the MTD buisness days by the what-if parameter `AdjustedProductionGoal(m)`
```
MTD Goal = 
VAR DaysInScope= COUNT(FiscalCalendar[Date])
VAR AdjGoal= [AdjustedProductionGoal(m)]
RETURN
DaysInScope * AdjGoal
```

The measure for the Radical Gauage is below:
```
TotalUnits =
VAR DailyT= [DailyTotals]
VAR MTDGoal= [MTD Goal]
RETURN
DailyT/[MTD Goal]
```
