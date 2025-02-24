
1. **AOV (Average Order Value):**
   - 
     ```
     AOV = AVERAGEX(salesorderheader, 'salesorderheader'[TotalDue])
     ```

2. **Avg. Sales Per Customer:**
   - 
     ```
     Average Sales Per Customer = 
     AVERAGEX(
         VALUES(salesorderheader[CustomerID]),
         CALCULATE(
             SUM(salesorderheader[TotalDue])
         )
     )
     ```

3. **Cumulative Percentage:**
   - 
     ```dax
     Cumulative_Perc. = [total cumulative] / [Sales in Total]
     ```

4. **Offline Orders:**
   - 
     ```
     Offline Orders = 
     CALCULATE(
         COUNT(salesorderheader[SalesOrderID]),
         NOT(ISBLANK(salesorderheader[SalesPersonID]))
     )
     ```

5. **Offline Orders %:**
   - 
     ```
     Offline Orders % = 
     DIVIDE([Offline Orders], [Total Orders], 0)
     ```

6. **Offline Sales:**
   - 
     ```
     Offline Sales = 
     CALCULATE(
         SUM(salesorderheader[TotalDue]),
         NOT(ISBLANK(salesorderheader[SalesPersonID]))
     )
     ```
   - **Explanation:** This measure calculates the sum of TotalDue for sales orders where a SalesPersonID is not blank, indicating offline sales.

7. **Offline Sales Percent:**
     ```
     Offline Sales % = 
     DIVIDE([Offline Sales], [Total Sales], 0)
     ```

8. **Online Orders:**
   - 
     ```
     Online Orders = 
     CALCULATE(
         COUNT(salesorderheader[SalesOrderID]),
         ISBLANK(salesorderheader[SalesPersonID])
     )
     ```
   - **Explanation:** This measure counts the number of sales orders where SalesPersonID **is blank**, indicating online orders.

9. **Online Sales:**
   - 
     ```
     Online Sales = 
     CALCULATE(
         SUM(salesorderheader[TotalDue]),
         ISBLANK(salesorderheader[SalesPersonID])
     )
     ```

10. **Online Sales %:**
    - 
      ```
      Online Sales % = 
      DIVIDE([Online Sales], [Total Sales], 0)
      ```

11. **Previous Year Total Sales:**
    - **Definition:** Total sales amount from the previous year, used for Year-over-Year (YoY) growth.
    - 
      ```
      Previous Year Total Sales = 
      CALCULATE(
          SUM(salesorderheader[TotalDue]),
          SAMEPERIODLASTYEAR(salesorderheader[OrderDate])
      )
      ```
      
12. **Sales in Total:**
    -
      ```
      Sales in Total = 
      SUM(salesorderheader[TotalDue])
      ```

13. **Total B2B Customers:**
    - 
      ```
      Total B2B Customers = 
      CALCULATE(
          DISTINCTCOUNT(salesorderheader[CustomerID]),
          customer[CustomerType] = "B2B"
      )
      ```

14. **Total B2B Sales:**
    - 
      ```
      Total B2B Sales = 
      CALCULATE(
          [Total Sales],
          customer[CustomerType] = "B2B"
      )
      ```

15. **Total B2C Customers:**
    - 
      ```
      Total B2C Customers = 
      CALCULATE(
          DISTINCTCOUNT(salesorderheader[CustomerID]),
          customer[CustomerType] = "B2C"
      )
      ```

16. **Total B2C Sales:**
    - 
      ```
      Total B2C Sales = 
      CALCULATE(
          [Total Sales],
          customer[CustomerType] = "B2C"
      )
      ```

17. **Total Cumulative:**
    - 
      ```
      Total Cumulative = 
      CALCULATE(
          SUM(Pareto[totalsales]),
          FILTER(
              ALLSELECTED(Pareto),
              Pareto[sales_rank] <= MAX(Pareto[sales_rank])
          )
      )
      ```

18. **Total Customers:**
    - 
      ```
      Total Customers = 
      DISTINCTCOUNT(salesorderheader[CustomerID])
      ```

19. **Total Orders:**
    - 
      ```
      Total Orders = 
      COUNT(salesorderheader[SalesOrderID])
      ```

20. **Total Sales Amount:**
    - 
      ```
      Total Sales Amount = 
      SUM('salesorderheader'[TotalDue])
      ```

21. **YoY Growth %:**
    -  
      ```
      YoY Growth % = 
      VAR CurrentYearSales = SUM(salesorderheader[TotalDue])
      VAR PreviousYearSales = [Previous Year Total Sales]
      RETURN
          IF(
              NOT(ISBLANK(PreviousYearSales)),
              (CurrentYearSales - PreviousYearSales) / PreviousYearSales,
              BLANK()
          )
      ```

#### 22. Calendar Table

```dax
Calendar = 
VAR MinDate = MIN(salesorderheader[OrderDate])
VAR MaxDate = MAX(salesorderheader[OrderDate])
RETURN
ADDCOLUMNS (
    CALENDAR (MinDate, MaxDate),
    "Year", YEAR ( [Date] ),
    "Month Number", MONTH ( [Date] ),
    "Month Name", FORMAT ( [Date], "MMMM" ),
    "Day", DAY ( [Date] ),
    "Quarter", QUARTER ( [Date] ),
    "Weekday", WEEKDAY ( [Date], 2 ),
    "Weekday Name", FORMAT ( [Date], "dddd" ),
    "Week Number", WEEKNUM ( [Date], 2 ),
    "Is Weekend", IF ( WEEKDAY ( [Date], 2 ) >= 6, TRUE, FALSE ),
    "Year-Month", FORMAT ( [Date], "YYYY-MM" )
)
```

#### 23. Geography Column

'Regional Sales' table:

```dax
Geography = 'Regional Sales'[ship_province]& " - " & 'Regional Sales'[country_state_name] & ", " & 'Regional Sales'[country_code]
```
