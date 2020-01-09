---
title: FHIR Power Query Folding
description: FHIR Power Query connector query folding
author: hansenms
ms.service: powerquery
ms.topic: conceptual
ms.date: 01/08/2020
ms.author: mihansen
LocalizationGroup: reference
---

# FHIR Query Folding

[Power Query folding](https://docs.microsoft.com/power-query/power-query-folding) is the mechanism used by a Power Query connector to turn data transformations into queries that are sent to the data source. This allows Power Query to off-load as much of the data selection as possible to the data source rather than retrieving large amounts of unneeded data only to discard it in the client. The FHIR Power Query connector includes query folding capabilities, but due to the nature of [FHIR search](https://www.hl7.org/fhir/search.html), special attention must be given to the Power Query expressions to ensure that query folding is performed when possible. This article explains the basics of FHIR Power Query folding and provides guidelines and examples.

## FHIR and Query Folding

Suppose you are constructing a query to retrieve "Patient" resources from a FHIR server and you are interested in patients born before the year 1980. Such a query could look like:

```
let
    Source = Fhir.Contents("https://myfhirserver.azurehealthcareapis.com", null),
    Patient1 = Source{[Name="Patient"]}[Data],
    #"Filtered Rows" = Table.SelectRows(Patient1, each [birthDate] < #date(1980, 1, 1))
in
    #"Filtered Rows"
```

Instead of retrieving all Patient resources from the FHIR server and filtering them in the client (Power BI), it would is more efficient to send a query with a search parameter to the FHIR server:

```
GET https://myfhirserver.azurehealthcareapis.com/Patient?birthdate=lt1980-01-01
```

With such a query the client would only receive the patients of interest and would not need to discard data in the client.

In the example of birth date, the query folding is straightforward, but in general it is challenging in FHIR because the search parameter names don't always correspond to the data field names and frequently multiple data fields will contribute to a single search parameter. 

For example let's consider the Observation resource and the "category" field. The `Observation.category` field is a `CodeableConcept` in FHIR, which has a `coding` field, which has a `system` and `code` fields (among other fields). Suppose you were interested in vital-signs only, you would be interested in Observations where `Observation.category.coding.code = "vital-signs"`, but the FHIR search would look something like `https://myfhirserver.azurehealthcareapis.com/Observation?category=vital-signs`.

To be able to achieve query folding in the more complicated cases, the FHIR Power Query connector matches Power Query expressions with a list of expression patterns and translates them into appropriate search parameters. This matching with expression patterns works best when the selection expression is done as early as possible data transformation steps.

> [!Note]
> To give the Power Query engine the best chance of performing query folding, you should do all data selection expressions before any shaping of the data.

## Query Folding Example

To illustrate efficient query folding, we will walk through the example of getting all vital signs from the Observation resource. The intuitive way to do this would be to first expand the `Observation.category` field and then expand `Observation.category.coding` and then filter. The query would look something like this:

```
let
    Source = Fhir.Contents("https://myfhirserver.azurehealthcareapis.com", null),
    Observation = Source{[Name="Observation"]}[Data],
    ExpandCategory = Table.ExpandTableColumn(Observation, "category", {"coding"}, {"category.coding"}),
    ExpandCoding = Table.ExpandTableColumn(ExpandCategory, "category.coding", {"system", "code"}, {"category.coding.system", "category.coding.code"}),
    FilteredRows = Table.SelectRows(ExpandCoding, each ([category.coding.code] = "vital-signs"))
in
    FilteredRows
```

Unfortunately, the Power Query engine no longer recognized that as a selection pattern that maps to the `category` search parameter, but if we restructure the query to:

```
let
    Source = Fhir.Contents("https://myfhirserver.azurehealthcareapis.com", null),
    Observation = Source{[Name="Observation"]}[Data],
    FilteredObservations = Table.SelectRows(Observation, each Table.MatchesAnyRows([category], each Table.MatchesAnyRows([coding], each [code] = "vital-signs"))),
    ExpandCategory = Table.ExpandTableColumn(FilteredObservations, "category", {"coding"}, {"category.coding"}),
    ExpandCoding = Table.ExpandTableColumn(ExpandCategory, "category.coding", {"system", "code"}, {"category.coding.system", "category.coding.code"})
in
    ExpandCoding
```

The search query `/Observation?category=vital-signs` will be sent to the FHIR server, which will reduce the amount of data that the client will receive from the server.

While the first and the second Power Query expressions will result in the same data set, the latter will, in general, result in better query performance. It is important to note that the second, more efficient, version of the query cannot be obtained purely through data shaping with the graphical user interface (GUI). It is necessary to write the query in the "Advanced Editor".

It is recommended that initial data exploration can be done with the GUI query editor, but it is recommended that the query be refactored with query folding in mind. Specifically, selective queries should be performed as early as possible.

## Summary

Query folding is an important mechanism to leverage in efficient Power Query expressions. A properly crafted Power Query will enable query folding and thus off-load much of the data filtering burden to the data source.

## Next steps

In this article, you have learned how to use query folding in the FHIR Power Query connector. Next explore the list of FHIR Power Query folding patterns.

>[!div class="nextstepaction"]
>[FHIR Power Query folding patterns](FHIR-QueryFoldingPatterns.md)