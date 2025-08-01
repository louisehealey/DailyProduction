# DailyProduction
This Power BI report provides a daily breakdown of completed units, delivering a clear view of total production across the calendar month. Users can adjust the daily production goal to support parameter testing and scenario planning. The report also calculates and compares the Month-to-Date (MTD) goal with actual MTD performance, enabling teams to track progress and stay aligned with production targets.
<p align="center">
  <img src="https://github.com/louisehealey/DailyProduction/blob/main/CompletedUnitsByDayDash.png"
      >
</p>

## Changing Column Columns
The **Adjusted Daily Goal** is a dynamic what-if parameter that enables dashboard users to modify the input value using a slicer. This allows them to explore various production output scenarios for the current calendar month.

Once a goal is selected, the Adjusted Daily Goal amount is multiplied by the total number of business days in the current calendar month. The resulting value is visually represented by the dotted yellow line on the bar chart.

After the Adjusted Daily Goal is calculated, it is compared against the actual daily production figures to determine whether the goal has been met. If the goal is met, the bar for that day turns green; otherwise, it remains red. This color-coding enables quick and intuitive visual analysis of daily performance.


<p align="center">
  <img src="https://github.com/louisehealey/DailyProduction/blob/main/CompletedUnitsByDay.gif" width="600"/>
</p>

## Parameters
##Meassues
