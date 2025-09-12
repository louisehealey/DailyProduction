# DailyProduction
This Power BI report delivers a comprehensive daily overview of completed units, offering clear insight into production across the calendar month. Users can interactively adjust the daily production goal to support parameter testing and scenario planning. The report dynamically calculates and visualizes Month-to-Date (MTD) goals alongside actual MTD performance, enabling teams to monitor progress and stay aligned with production targets.

If today were August 15, 2022, the dashboard might appear as follows:

<p align="center">
  <img src="https://github.com/louisehealey/DailyProduction/blob/main/CompletedUnitsByDayDash.png"
      >
</p>
The chart features a dynamic goal line that reflects the user-defined daily production target, adjustable via a slider or by entering a value between 25 and 300. Once a target is set, the bars respond in real time—turning green when the goal is met or exceeded, and red when production falls short. This immediate visual feedback allows users to quickly assess daily performance.

The Radical Gauge provides a high-level visual indicator of monthly production progress relative to the Month-to-Date (MTD) goal. Together, these visuals offer a clear and intuitive way for users to monitor output, and stay aligned with production targets throughout the month.

## Data Modeling:

### Generating a Date Table 
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
### Dates in Scope
Accurately calculating the number of business days in a month requires excluding non-business days such as weekends and holidays. Below is a simple formula to exclude weekends . 
``` 
IsWeekday =
VAR DayNumber = WEEKDAY([Date], 2)
RETURN
    IF(DayNumber <= 5, 1, 0)
``` 
Excluding holidays is also necessary when determiing the worked days in scope. To capture holidays, you would need to load your companys fisical calendar into Power BI and add a `RELATED` or `LOOKUPVALUE` variable to pull in the additional dates you want to exclude. The below example :
```
IsWorkday =
VAR DayNumber = WEEKDAY([Date], 2)
VAR IsWeekend = IF(DayNumber > 5, 1, 0)
VAR IsHoliday = IF([Date] IN VALUES(HolidayDates[HolidayDate]), 1, 0)
RETURN
    IF(IsWeekend = 0 && IsHoliday = 0, 1, 0)
```

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
  <img src="https://github.com/louisehealey/DailyProduction/blob/main/CompletedUnitsByDay.gif" width="800"/>
</p>

