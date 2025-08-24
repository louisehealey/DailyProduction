# DailyProduction
This Power BI report provides a daily breakdown of completed units, delivering a clear view of total production across the calendar month. Users can adjust the daily production goal to support parameter testing and scenario planning. The report also calculates and compares the Month-to-Date (MTD) goal with actual MTD performance, enabling teams to track progress and stay aligned with production targets.
<p align="center">
  <img src="https://github.com/louisehealey/DailyProduction/blob/main/XXX.png"
      >
</p>


## Generating a Date Table 
Adding a date table is essential when working with time-based data. In our case, it ensures continuity by preventing gaps or inconsistencies caused by business days when no units are completed. Without a date table, the report would simply skip over those inactive days, making it difficult to accurately analyze trends or performance over time.

If today were August 15, 2025, the following DAX expression would generate a fiscal calendar covering the past year:
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
### Dates in Scope
--Total Quantity >0 || Weekday 
 'CLOSED_JOBS'[DateFilter] 

## Generating a Paremeter for the What-if Analysis
Generating an Adjustable Paremeter, in our case it's the **Adjusted Daily Goal**, is simple. Go to Modeling > New Paremeter > Numeric Range. This will return the values below

**The Table:**
```
AdjustedProductionGoal = GENERATESERIES(0, 300, 25)
```
**The Measure:**
```
AdjustedProductionGoal(m) = SELECTEDVALUE('Adjustable Production Goal'[Parameter], 75)
````
## Calculating the Total Units Produced 

**Daily Total**
```
DailyTotals = 
VAR SUMM =
    CALCULATE(
        ABS(SUM('CLOSED_JOBS'[quantity])),
        'CLOSED_JOBS'[DateFilter] = "TRUE"
    )
RETURN
    IF(ISBLANK(SUMM), 0, SUMM)
```
## MTD Goal
```
MTD Goal = 
VAR DaysInScope= COUNT(FiscalCalendar[Date])
VAR AdjGoal= [AdjustedProductionGoal(m)]
RETURN
DaysInScope * AdjGoal
```

## Average Daily Total
```
Average Daily Total = 
AVERAGEX(FiscalCalendar,[DailyTotals])
```

## Changing Column Columns
The **Adjusted Daily Goal** is a dynamic what-if parameter that enables dashboard users to modify the input value using a slicer. This allows them to explore various production output scenarios for the current calendar month.

Once a goal is selected, the Adjusted Daily Goal amount is multiplied by the total number of business days in the current calendar month. The resulting value is visually represented by the dotted yellow line on the bar chart.

After the Adjusted Daily Goal is calculated, it is compared against the actual daily production figures to determine whether the goal has been met. If the goal is met, the bar for that day turns green; otherwise, it remains red. This color-coding enables quick and intuitive visual analysis of daily performance.


<p align="center">
  <img src="https://github.com/louisehealey/DailyProduction/blob/main/CompletedUnitsByDay.gif" width="600"/>
</p>

