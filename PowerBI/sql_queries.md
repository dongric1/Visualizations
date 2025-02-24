### Google BigQuery - SQL Queries

#### 1. Region Codes Query

```sql
SELECT 
  TerritoryID,
  CONCAT(name, ', ', CountryRegionCode) AS Territoryname,
  Name,
  CountryRegionCode,
  `Group`
FROM `tc-da-1.adwentureworks_db.salesterritory`
```

#### 2. Pareto Query

```sql
SELECT
  SUM(TotalDue) AS totalsales,
  e.EmployeeId,
  CASE
    WHEN e.EmployeeId IS NULL THEN 'Online'
    ELSE CONCAT(c.Firstname, IF(c.MiddleName IS NOT NULL AND c.MiddleName != '', CONCAT(' ', c.MiddleName), ''), ' ', c.LastName)
  END AS full_name,
  RANK() OVER (ORDER BY SUM(TotalDue) DESC) AS sales_rank
FROM
  `tc-da-1.adwentureworks_db.salesorderheader` soh
LEFT JOIN
  `tc-da-1.adwentureworks_db.employee` e ON e.EmployeeId = soh.SalesPersonID
LEFT JOIN
  `tc-da-1.adwentureworks_db.contact` c ON c.ContactId = e.ContactID
GROUP BY
  ALL
ORDER BY
  EmployeeId
```


#### 3. Regional Sales

```sql
SELECT
      salesorderheader.*,
      province.stateprovincecode as ship_province,
      province.CountryRegionCode as country_code,
      province.name as country_state_name
FROM `tc-da-1.adwentureworks_db.salesorderheader` as salesorderheader
INNER JOIN
     `tc-da-1.adwentureworks_db.address` as address
    ON salesorderheader.ShipToAddressID = address.AddressID
INNER JOIN
     `tc-da-1.adwentureworks_db.stateprovince` as province
    ON address.stateprovinceid = province.stateprovinceid
```

#### 4. Sales Reason

```sql
WITH sales_per_reason AS (
 SELECT
   DATE_TRUNC(OrderDate, MONTH) AS year_month,
   sales_reason.SalesReasonID,
   SUM(sales.TotalDue) AS sales_amount
 FROM
   `tc-da-1.adwentureworks_db.salesorderheader` AS sales
 INNER JOIN
   `tc-da-1.adwentureworks_db.salesorderheadersalesreason` AS sales_reason
 ON
   sales.SalesOrderID = sales_reason.salesOrderID
 GROUP BY 1,2
)
SELECT
 sales_per_reason.year_month,
 reason.Name AS sales_reason,
 sales_per_reason.sales_amount
FROM
 sales_per_reason
LEFT JOIN
 `tc-da-1.adwentureworks_db.salesreason` AS reason
ON
 sales_per_reason.SalesReasonID = reason.SalesReasonID
```