---
title: "Right outer join"
description: An article on how to do a merge operation in Power Query using the right outer join kind. 
author: ptyx507
ms.service: powerquery
ms.reviewer: 
ms.date: 07/22/2020
ms.author: v-miesco
---

# Right outer join

A right outer join is one of the join kinds available inside the **Merge queries** window in Power Query. To read more about the merge operations in Power Query, see [Merge operations overview](merge-queries-overview.md).

A right outer join keeps all the rows from the right table, and brings any matching rows from the left table.

This article demonstrates, with a practical example, how to do a merge operation using the right outer join as the join kind.

![Sample right outer join](images/right-outer-join-operation.png)

>[!Note]
>Samples used in this article are only to showcase the concepts. The concepts showcased here apply to all queries in Power Query.

## Sample input and output tables

The sample source tables for this example are:

* **Sales**&mdash;with the fields **Date**, **CountryID**, and **Units**. The *CountryID* is a whole number value that represents the unique identifier from the **Countries** table.

   ![Sales table](images/me-merge-operations-left-outer-join-sales-table.png)

* **Countries**&mdash;this table is a reference table with the fields **id** and **Country**. The *id* represents the unique identifier of each record.

   ![Countries table](images/me-merge-operations-right-outer-join-countries-table.png)

The goal is to merge both tables, where the **Sales** table will be the left table and the **Countries** table the right one. The join will be made between the following columns:

|Field from Sales table| Field from Countries table|
|-----------|------------------|
|CountryID|id|

The goal is to reach the following table where the name of the country appears as a new **Country** column in the **Sales** table. Because of how the right outer join works, all rows from the right table will be brought in, but only matching rows from the left table will be kept.

![Right outer join final table](images/me-merge-operations-right-outer-final-table.png)

## Right outer join

To do a right outer join:

1. Select the **Sales** query, and then select **Merge queries** to create a new step inside the Sales query that will merge the **Sales** query with the **Countries** query.
2. Select **Countries** as the **Right table for merge**.
3. Select the **CountryID** column from the **Sales** table.
4. Select the **id** column from the **Countries** table.
5. From the **Join Kind** section, select the **Right outer** option.
6. Select **OK**.

![Merge window for right outer join](images/me-merge-operations-right-outer-merge-window.png)

From the newly created **Countries** column after the merge operation, expand the **Country** field without using the original column name as prefix.

![Expand table column for Country](images/me-merge-operations-right-outer-expand-field.png)

After doing this operation, the desired result is reached.

![Right outer join final table](images/me-merge-operations-right-outer-final-table.png)
