---
title: "Examples: Using AUTO Mode"
description: View examples of queries that use FOR XML AUTO mode.
ms.custom: ""
ms.date: 05/05/2022
ms.prod: sql
ms.prod_service: "database-engine"
ms.reviewer: randolphwest
ms.technology: xml
ms.topic: conceptual
helpviewer_keywords:
  - "AUTO FOR XML mode, examples"
author: MikeRayMSFT
ms.author: mikeray
---
# Examples: Using AUTO mode

[!INCLUDE [SQL Server Azure SQL Database](../../includes/applies-to-version/sql-asdb.md)]

The following examples illustrate the use of AUTO mode. Many of these queries are specified against bicycle manufacturing instructions XML documents that are stored in the Instructions column of the ProductModel table in the [!INCLUDE[ssSampleDBnormal](../../includes/sssampledbnormal-md.md)] sample database.

## Example: Retrieve customer, order, and order detail information

This query retrieves customer, order, and order detail information for a specific customer.

```sql
USE AdventureWorks2012;
GO
SELECT Cust.CustomerID,
       OrderHeader.CustomerID,
       OrderHeader.SalesOrderID,
       Detail.SalesOrderID, Detail.LineTotal, Detail.ProductID,
       Product.Name,
       Detail.OrderQty
FROM Sales.Customer AS Cust
INNER JOIN Sales.SalesOrderHeader AS OrderHeader
    ON Cust.CustomerID = OrderHeader.CustomerID
INNER JOIN Sales.SalesOrderDetail AS Detail
    ON OrderHeader.SalesOrderID = Detail.SalesOrderID
INNER JOIN Production.Product AS Product
    ON Product.ProductID = Detail.ProductID
WHERE Cust.CustomerID IN (29672, 29734)
ORDER BY OrderHeader.CustomerID,
         OrderHeader.SalesOrderID
FOR XML AUTO;
```

Because the query identifies, `Cust`, `OrderHeader`, `Detail`, and `Product` table aliases, corresponding elements are generated by the `AUTO` mode. Again, the order in which tables are identified by the columns specified in the `SELECT` clause determine the hierarchy of these elements.

This is the partial result.

```xml
<Cust CustomerID="29672">
  <OrderHeader CustomerID="29672" SalesOrderID="43660">
    <Detail SalesOrderID="43660" LineTotal="874.794000" ProductID="758" OrderQty="1">
      <Product Name="Road-450 Red, 52" />
    </Detail>
    <Detail SalesOrderID="43660" LineTotal="419.458900" ProductID="762" OrderQty="1">
      <Product Name="Road-650 Red, 44" />
    </Detail>
  </OrderHeader>
  <OrderHeader CustomerID="29672" SalesOrderID="47660">
    <Detail SalesOrderID="47660" LineTotal="469.794000" ProductID="765" OrderQty="1">
      <Product Name="Road-650 Black, 58" />
    </Detail>
  </OrderHeader>
  <OrderHeader CustomerID="29672" SalesOrderID="49857">
    <Detail SalesOrderID="49857" LineTotal="44.994000" ProductID="852" OrderQty="1">
      <Product Name="Women's Tights, S" />
    </Detail>
  </OrderHeader>
...
</Cust>
```

## Example: Specify GROUP BY and aggregate functions

The following query returns individual customer IDs and the number of orders that the customer has requested.

```sql
USE AdventureWorks2012;
GO
SELECT C.CustomerID, COUNT(*) AS NoOfOrders
FROM Sales.Customer AS C
INNER JOIN Sales.SalesOrderHeader AS SOH
On C.CustomerID = SOH.CustomerID
GROUP BY C.CustomerID
FOR XML AUTO;
```

This is the partial result:

```xml
<I CustomerID="11000" NoOfOrders="3" />
<I CustomerID="11001" NoOfOrders="3" />
...
```

## Example: Specify computed columns in AUTO mode

This query returns concatenated individual customer names and the order information. Because the computed column is assigned to the innermost level encountered at that point, the `<SOH>` element in this example. The concatenated customer names are added as attributes of the `<SOH>` element in the result.

```sql
USE AdventureWorks2012;
GO
SELECT P.FirstName + ' ' + P.LastName AS Name,
       SOH.SalesOrderID
FROM Sales.Customer AS C
INNER JOIN Sales.SalesOrderHeader AS SOH
    ON  C.CustomerID = SOH.CustomerID
INNER JOIN Person.Person AS P
    ON P.BusinessEntityID = C.PersonID
FOR XML AUTO;
```

This is the partial result:

```xml
<SOH Name="Jon Yang" SalesOrderID="43793" />
<SOH Name="Eugene Huang" SalesOrderID="43767" />
```

To retrieve the `<IndividualCustomer>` elements having the `Name` attribute that contains each sales order header information as a subelement, the query is rewritten using a sub select. The inner select creates a temporary `IndividualCustomer` table with the computed column that contains the names of the individual customers. This table is then joined to the `SalesOrderHeader` table to obtain the result.

The `Sales.Customer` table stores individual customer information, including the `PersonID` value for that customer. This `PersonID` is then used to find the contact name from the `Person.Person` table.

```sql
SELECT IndividualCustomer.Name, SOH.SalesOrderID
FROM (SELECT FirstName+ ' '+LastName AS Name, C.PersonID, C.CustomerID
      FROM Sales.Customer AS C, Person.Person AS P
      WHERE C.PersonID = P.BusinessEntityID) AS IndividualCustomer
LEFT OUTER JOIN  Sales.SalesOrderHeader AS SOH
   ON IndividualCustomer.CustomerID = SOH.CustomerID
ORDER BY IndividualCustomer.CustomerID, SOH.CustomerIDFOR XML AUTO;
```

This is the partial result:

```xml
<IndividualCustomer Name="Jon Yang">
  <SOH SalesOrderID="43793" />
  <SOH SalesOrderID="51522" />
  <SOH SalesOrderID="57418" />
</IndividualCustomer>
...
```

## Example: Return binary data

This query returns a product photo from the `ProductPhoto` table. `ThumbNailPhoto` is an **varbinary(max)** column in the `ProductPhoto` table. By default, `AUTO` mode returns to the binary data a reference that is a relative URL to the virtual root of the database where the query is executed. The `ProductPhotoID` key attribute must be specified to identify the image. When retrieving an image reference as illustrated in this example, the primary key of the table must also be specified in the `SELECT` clause to uniquely identify a row.

```sql
SELECT ProductPhotoID, ThumbNailPhoto
FROM   Production.ProductPhoto
WHERE ProductPhotoID = 70
FOR XML AUTO;
```

This is the result:

```xml
<Production.ProductPhoto
  ProductPhotoID="70"
  ThumbNailPhoto= "dbobject/Production.ProductPhoto[@ProductPhotoID='70']/@ThumbNailPhoto" />
```

The same query is executed with the `BINARY BASE64` option. The query returns the binary data in base64-encoded format.

```sql
SELECT ProductPhotoID, ThumbNailPhoto
FROM   Production.ProductPhoto
WHERE ProductPhotoID = 70
FOR XML AUTO, BINARY BASE64;
```

This is the result:

```xml
<Production.ProductPhoto ProductPhotoID="70" ThumbNailPhoto="Base64 encoded photo" />
```

By default, when you use AUTO mode to retrieve binary data, a reference to a relative URL to the virtual root of the database where the query was executed will be returned instead of the binary data. This will occur if the BINARY BASE64 option isn't specified.

When AUTO mode returns a URL reference to the binary data in case-insensitive databases where a table or column name specified in the query doesn't match the table or column name in the database, the query executes. However, the case returned in the reference won't be consistent. For example:

```sql
SELECT ProductPhotoID, ThumbnailPhoto
FROM   Production.ProductPhoto
WHERE  ProductPhotoID=70
FOR XML AUTO;
```

This is the result:

```xml
<Production.PRODUCTPHOTO
  PRODUCTPHOTOID="70"
  THUMBNAILPHOTO= "dbobject/Production.PRODUCTPHOTO[@ProductPhotoID='70']/@ThumbNailPhoto" />
```

This can be a problem particularly when `dbobject` queries are executed against a case sensitive database. To avoid this, the case of the table or column name that is specified in the queries should match the case of the table or column name in the database.

## Example: Understand the encoding

This example shows the various encoding that occurs in the result.

Create this table:

```sql
CREATE TABLE [Special Chars] (Col1 char(1) primary key, [Col#&2] varbinary(50));
```

Add the following data to the table:

```sql
INSERT INTO [Special Chars] VALUES ('&', 0x20), ('#', 0x20);
```

This query returns the data from the table. The FOR XML AUTO mode is specified. Binary data is returned as a reference.

```sql
SELECT * FROM [Special Chars] FOR XML AUTO;
```

This is the result:

```xml
<Special_x0020_Chars Col1="#"
Col_x0023__x0026_2="dbobject/Special_x0020_Chars[@Col1='#']/@Col_x0023__x0026_2"
/>
<Special_x0020_Chars Col1="&"
Col_x0023__x0026_2="dbobject/Special_x0020_Chars[@Col1='&']/@Col_x0023__x0026_2"
/>
```

This is the process for encoding special characters in the result:

- In the query result, the special XML and URL characters in the element and attribute names that are returned are encoded by using the hexadecimal value of the corresponding Unicode character. In the previous result, the element name `<Special Chars>` is returned as `<Special_x0020_Chars>`. The attribute name `<Col#&2>` is returned as `<Col_x0023__x0026_2>`. Both XML and URL special characters are encoded.

- If the values of the elements or attribute contain any of the five standard XML character entities (', "", \<, >, and &), these special XML characters are always encoded using XML character encoding. In the previous result, the value `&` in the value of attribute `<Col1>` is encoded as `&`. However, the # character remains #, because it's a valid XML character and not a special XML character.

- If the values of the elements or attributes contain any special URL characters that have special meaning in the URL, they're encoded only in the DBOBJECT URL value and are encoded only when the special character is part of a table or column name. In the result, the character `#` that is part of table name `Col#&2` is encoded as `_x0023_ in the DBOJBECT URL`.

## See also

- [Use AUTO Mode with FOR XML](../../relational-databases/xml/use-auto-mode-with-for-xml.md)
