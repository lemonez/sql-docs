---
title: "getNClob Method (int)"
description: "getNClob Method (int)"
author: David-Engel
ms.author: v-davidengel
ms.date: "01/19/2017"
ms.prod: sql
ms.technology: connectivity
ms.topic: reference
---
# getNClob Method (int)
[!INCLUDE[Driver_JDBC_Download](../../../includes/driver_jdbc_download.md)]

  Retrieves the value of the designated JDBC **NCLOB** parameter as a NClob object in the Java programming language.  
  
## Syntax  
  
```  
  
public java.sql.NClob getNClob(int parameterIndex)  
```  
  
#### Parameters  
 *parameterIndex*  
  
 An **int** that indicates the parameter index.  
  
## Return Value  
 ANClobobject.  
  
## Exceptions  
 [SQLServerException](../../../connect/jdbc/reference/sqlserverexception-class.md)  
  
## Remarks  
 This getNClob method is specified by the getNClob method in the java.sql.CallableStatement interface.  
  
 This method only supports retrieving **NCHAR**, **NVARCHAR**, **NTEXT**, and **XML** parameters. Calling these methods on other data type parameters will cause an exception.  
  
## See Also  
 [getNClob Method &#40;SQLServerCallableStatement&#41;](../../../connect/jdbc/reference/getnclob-method-sqlservercallablestatement.md)   
 [SQLServerCallableStatement Members](../../../connect/jdbc/reference/sqlservercallablestatement-members.md)  
  
  
