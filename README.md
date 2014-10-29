OCDS API
========

**Open Contracting Data Standard** aims to allow information to be shared as structured data and to provide a transparent way to analyze all the phases of a contract from planning to completion. By creating the *OCDS API* we are offering to developers the possibility to create easy to interpret visual reports and application that can identify trends in public contracts.

This document provides a description of the APIs and examples of their use.


1. API conventions
2. Records API
3. Releases API
4. Pagination
5. Metadata API
6. Versioning
7. Errors


## 1. API Conventions

**Authentication**
Using the OCDS API doesn’t require any kind of authentication, users can send as many request as they want, the only limitation will be the number of records returned; in this way we encourage developers to use pagination in order to iterate throw all of the data.

**JSON Callbacks**

The **OCDS API** is developer-friendly so it’s explorable via a browser address bar for testing purposes, but also in a real application you can use AJAX or JSON-P callbacks in order to query the system and *consume* the response:

```
$.ajax({
	type: 'GET',
	url:  'http://www.contractawards.eu/open-contracting/api/v1/years',
	success: function(response) {
		console.log(response.hits);
	}
});
```

All the request should have the type **GET**.

## 2. Records API


**Limiting release content that is returned by the API**

**Open Contracting Data Standard** allows a release to be displayed in 2 ways in a record package:
* as a URI having a unique releaseID that contains the actual information about the release
* embed the actual release contend (the same content that can be obtain using the first method, but with this method the response  size will be much smaller

Having this in consideration and the fact that an API consumer doesn’t always need the full representation of a release the user can use the *embed=true* parameter in order to minimize network traffic and speed up the usage of the **Records API**

For example, the following request would retrieve just the basic information about a record package but it will also have the releases URI that can be further used to retrieve the releases content:

```
GET http://www.contractawards.eu/open-contracting/api/v1/record-package?year=2012

{
    "uri": "http://www.contractawards.eu/open-contracting/api/v1/record-package?year=2012”,
    "publisher": {
        "name": "Development Gateway"
    },
    "license": "http://opensource.org/licenses/MIT",
    "publishedDate": "2012-09-14T04:28:58-0400",
    "packages": [
        "http://www.contractawards.eu/open-contracting/api/v1/release-package/releaseID1"
    ],
    "records": [
        {
            "ocid": "1234-5678",
            "releases": [
                {
                    "name": "releaseID1",
                    "uid": "releaseID",
                    "uri": "http://www.contractawards.eu/open-contracting/api/v1/release/releaseID1"
                }
            ]
        }
    ]
}
```

## 3. Releases API

This is the API that can be used to obtain information about OCDS releases. It’s build to be flexible and with the help of the filters users can slice the data and drill down the results.

## 4. Pagination

## 5. Metadata API

**Getting filters metadata**

This API is used to get information about the *filters* that can be used with the **Records API**

**Basic call and parameters**

```
GET /open-contracting/api/v1/<dataset>
```

Calls will return information about the *dataset* and all valid values that can be used. If a value is not returned, for example year 2000, this means that we don’t have contracts published in 2000.

Possible values for *dataset*:
* **years** - returns an array of valid years
* **buyerCountries** - returns all buyer countries
* **supplierCountries** - returns supplier countries
* **sectors** - returns the main sector of the contract
* **procedures** - returns the procedures (Service contract, Works)
* **awardCriteria** - returns the awarding criteria (Lowest price, The most economic tender)

**Response example**

```
GET http://www.contractawards.eu/open-contracting/api/v1/years

{
  hits: [2006, 2007, 2008, 2009, 2010, 2011, 2012, 2013]
}
```

The above response will be available in a web browser, but if the user uses ajax, then a possible way to get all the years that can be used to filter the data will be:

```
$.ajax({
	type: 'GET',
	url: 'http://www.contractawards.eu/open-contracting/api/v1/years',
	success: function(response) {
		console.log(response.hits);
	}
});
```


## 6. Versioning

Change is inevitable so we decided to implement a versioning API that will help us to iterate faster and prevent invalid requests. This will also allow us to easily adopt new version of the OCDS standard and continue to offer old API versions for a period of time. The version are identified as /**api/v1**/, **api/v2**/ and so on.

## 7. Errors
