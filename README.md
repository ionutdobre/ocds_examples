OCDS API
========

**Open Contracting Data Standard** aims to allow information to be shared as structured data and to provide a transparent way to analyze all the phases of a contract from planning to completion. By creating the *OCDS API* we are offering developers the possibility to create with open contracting data reports and applications that can identify trends in public contracts.

This document provides a description of the APIs and examples of their use.

1. API conventions
2. Records API
3. Releases API
4. Metadata API
5. Versioning
6. Errors
7. Possible response

## 1. API Conventions

### Authentication and Usage Limits
OCDS API doesn’t require any kind of authentication. 
Users can send as many requests as they want, the only limitation will be the number of records returned; in this way we encourage developers to use pagination in order to iterate through all of the requested data.

### JSON Callbacks
**OCDS API** is explorable via a browser address bar for testing purposes. In a real application you can use AJAX or JSON-P callbacks in order to query the system and *consume* the response:

```
$.ajax({
	type: 'GET',
	url:  'http://www.contractawards.eu/open-contracting/api/v1/years',
	success: function(response) {
		console.log(response.hits);
	}
});
```

All requests should have the type **GET**.

### OCDS Definitions
Information about an Open Contracting Process may accumulate over time. As a result, the Open Contracting Data Standard provides for two kinds of data:
1. **Contracting release** - Information pertaining to a particular stage in the contracting process - such as tender notices, award notices, or details of a finalized contract.
2. **Contracting record** - A snapshot of all the key elements of a unique contracting process, including its planning, formation, performance and completion.

Both **contracting releases** and **contracting records** are provided within data packages, containing meta-data about the publisher, publication data and licensing information.

## 2. Records API

This API can be used to generate record packages or to retrieve individual *record* information. With the help of *filters* users can slice the data and drill down the results.

### Basic call and parameters

```
http://www.contractawards.eu/open-contracting/api/v1/record-package?<filters>
```
The following parameters (```filters```) are supported:
* ```supplier``` - Supplier name, e.g., *Acciona infraestructuras*
* ```buyer``` - Buyer name, e.g., *ADMINISTRADOR DE INFRAESTRUCTURAS FERROVIARIAS*
* ```sector``` - Sector, e.g., *Transport services*(!)
* ```year``` - Year, e.g., *2012*
* ```procedure``` - Tendering procedure, e.g., *Service contract* (!)
* ```awardCriteria``` - Award criteria, e.g, *Lowest price*
* ```buyerCountry``` - e.g., *Germany* (!)
* ```supplierCountry``` - e.g., *Italy* (!)

Please note that the last 6 criteria (```sector```, ```year```, ```procedure```, ```awardCriteria```, ```buyerCountry``` and ```supplierCountry```) need to be valid. You can check the validity by invoking the **Metadata API**.

**Request pagination**

Each request accepts the following parameters:
* ```page``` – page number
* ```pageSize``` – number of records per page; the default value is 100 and it can not be greater that 1000

For example, 
```
http://www.contractawards.eu/open-contracting/api/v1/record-package?year=2008&buyerCountry=Germany&page=10&pageSize=20
```

In the above example you will get the 10th 20-record page.

```
http://www.contractawards.eu/open-contracting/api/v1/record-package?year=2008&supplierCountry=France&page=1&pageSize=50
```

```
http://www.contractawards.eu/open-contracting/api/v1/record-package?sector=Construction work&procedure=Service contract&year=2012&page=1&pageSize=1000
```

### Rules for invoking the Filter API
1.	Supplier name and buyer name reset all other parameters, in other words if one of the ```supplier``` or ```buyer``` parameter is present all other parameters will be ignored.
2.	All other parameters (```sector```, ```year```, ```procedure```, ```awardCriteria```, ```buyerCountry```, ```supplierCountry```) can be combined in any order with the condition that at least two criteria from the aforementioned list be used
3.	The *HTTP Method* for all request should be **GET**

### Limiting release content that is returned by the API
**Open Contracting Data Standard** allows a release to be displayed in 2 ways in a record package:
* as a URI having a unique *releaseID* that contains the actual information about the release
* embed the actual release contend(the same content that can be obtain using the first method), but with this method the response size will be much larger

Having this in consideration and the fact that an API consumer doesn’t always need the full representation of a release the user can use the ```embed=true``` parameter in order to minimize network traffic and speed up the usage of the **Records API**.

For example, the following request would retrieve just the basic information about a record package but it will also have the releases URI that can be further used to retrieve the releases content:

```
GET http://www.contractawards.eu/open-contracting/api/v1/record-package?year=2012&embed=true

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

### Getting a single record

Each record has a unique **Open Contracting ID** called ```ocid``` (http://ocds.open-contracting.org/standard/r/0__3__3/#conceptual-model) and sometimes it is useful not to see the big picture but only the informations about a particular record so the user can do that by using the following API:

```
http://www.contractawards.eu/open-contracting/api/v1/record?recordId=<ocid>
```

### Pretty print format
By default the API provides pretty printing of the JSON output because it’s more approachable but in a production application the user can reduce the cost of the data transfer by using the parameter ```pretty=false``` in order to remove all the white spaces from the response.

### Records API Response

The response fields for the package records are explained in more detailed below

* ```uri``` - the URI of this records package
* ```publisher``` - information to uniquely identify the publisher of this package
* ```publishedDate``` - the date that this package was published.
* ```packages``` - a list of URIs of all the release packages that were used to create this record package
* ```records``` - the records for this data package

## 3. Releases API

This API can be used to obtain informations about a package release or a particular release. It can be useful to obtain more  information about a particular stage in the contracting process or if the user is using the ```embed=true``` parameter in the **Records API** and then he wants to get the content of the releases.

### Basic call and parameters

```
http://www.contractawards.eu/open-contracting/api/v1/release-package?releaseId=<id>
```

It can be used to get the release package with the given ```id```.

```
http://www.contractawards.eu/open-contracting/api/v1/release?releaseId=<id>
```

It can be used to get only one release content with the given ```id```.

**Pretty format** - the **Release API** supports the same ```pretty=false``` parameter as **Records API** in order to get a white-space compressed response.

### Release API Response

The response fields for a release are explained in more detailed below

* ```ocid``` - A unique identifier that identifies the unique Open Contracting Process
* ```releaseID``` - A unique identifier that identifies this release
* ```releaseDate``` - The date this information is released, it may well be the same as the parent publishedDate
* ```releaseTag``` - A tag that helps to identify the type of data in the dataset (it should be *awardNotice*)
* ```language``` - Specifies the default language of the data
* ```formationType``` - String specifying the type of formation process used for this contract(it should be *tender*)
* ```buyer``` - The buyer is the entity whose budget will be used to purchase the goods.
	* **id**
		* *name*
		* *uid*
		* *uri*
	* **address** - An Address following the convention of http://microformats.org/wiki/hcard
		* *locality*
		* *region*
		* *country-name*
* ```tender``` - The activities undertaken in order to enter into a contract.
	* **tenderID** - TenderID should always be the same as the OCID.
	* **notice** - The notice is a published document that notifies the public at various stages of the contracting process
		* *id*
		* *uri*
		* *publishedDate*
	* **itemsToBeProcured** - The goods and services to be purchased
		* *description*
		* *classificationScheme*
		* *classificationID*
		* *classificationDescription*
	* **totalValue** - The total estimated value of the procurement
		* *amount*
		* *currency*
	* **numberOfBids** - The number of unique bidders who participated in the tender
* ```awards``` - Information from the award phase of the contracting process
	* **awardID** - The identifier for this award
	* **notice**
		* *id*
		* *uri*
		* *publishedDate*
	* **awardValue** - The total value of this award
		* *amount*
		* *currency*
	* **suppliers** - The suppliers awarded this award
		* *id*
			* *name*
			* *uid*
			* *uri*
		* *address*
			* *locality*
			* *region*
			* *country-name*
	* **itemsAwarded** - The goods and services awarded in this award, broken into line items wherever possible
		* *description*
		* *classificationScheme*
		* *classificationID*
		* *classificationDescription*

## 4. Metadata API

**Getting filters metadata**

This API is used to get information about the ```filters``` that can be used with the **Records API**

### Basic call and parameters
```
GET /open-contracting/api/v1/<dataset>
```

Calls will return information about the ```dataset``` and all valid values that can be used. If a value is not returned, for example year 2000, this means that we don’t have contracts published in 2000.

Possible values for ```dataset```:
* ```years``` - returns an array of valid years
* ```buyerCountries``` - returns all buyer countries
* ```supplierCountries``` - returns supplier countries
* ```sectors``` - returns the main sector of the contract
* ```procedures``` - returns the procedures (Service contract, Works)
* ```awardCriteria``` - returns the awarding criteria (Lowest price, The most economic tender)

#### Response example
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

## 5. Versioning

Change is inevitable so we decided to implement a versioning API that will help us to iterate faster and prevent invalid requests. This will also allow us to easily adopt new version of the OCDS standard and continue to offer old API versions for a period of time. The version are identified as ```/api/v1/```, ```api/v2/``` and so on.

## 6. Errors

In case of an error(client issue of server issue) we will return a response with a JSON error body that will provide:  a useful error message, a unique error code that can be looked up for a detailed description in the documentation and the URL that cause the error. For example:

```
{
  "errors": [
   {
    "code" : 1024,
    "message": "Sorry, the requested resource does not exist"
    "url": "http://www.contractawards.eu/open-contracting/api/v1/release?id=156809-2006"
   }
  ]
}
```
or

```
{
  "errors": [
   {
    "code" : 1020,
    "message": " You should use at least 2 criteria from the following list of filters: sector, year, procedure, awardCriteria, buyerCountry, supplierCountry"
    "url": "http://www.contractawards.eu/open-contracting/api/v1/record-package?year=2012"
   }
  ]
} 
```

## 7. Possible response

In order to have a complete picture on the response that you can get by calling the **Records API** you can take a look at the following example.

```
{
    "uri": "http://www.contractawards.eu/open-contracting/api/v1/record-package/{filters}",
    "publisher": {
        "name": "Development Gateway"
    },
    "license": "http://opensource.org/licenses/MIT",
    "publishedDate": "2006-09-14T04:28:58-0400",
    "packages": [
        "http://www.contractawards.eu/open-contracting/api/v1/release-package/releaseID1",
        "http://www.contractawards.eu/open-contracting/api/v1/release-package/releaseID2",
        "http://www.contractawards.eu/open-contracting/api/v1/release-package/releaseID3"
    ],
    "records": [
        {
            "ocid": "1234-5678",
            "releases": [
                {
                    "ocid": "1234-5678",
                    "releaseID": "releaseID-1234-5678",
                    "releaseDate": "2006-09-14T04:28:58-0400",
                    "releaseTag": "awardNotice",
                    "language": "en",
                    "formationType": "tender",
                    "buyer": {
                        "id": {
                            "name": "CONSEIL GÉNÉRAL DE L'YONNE",
                            "uid": "6490828",
                            "uri": "http://www.dgmarket.com/tenders/adminShowBuyer.do?buyerId=6490828"
                        },
                        "address": {
                            "locality": "Paris",
                            "region": "Paris",
                            "country-name": "France"
                        }
                    },
                    "tender": {
                        "tenderID": "1219083",
                        "notice": {
                            "id": "1234-5678",
                            "uri": "http://www.dgmarket.com/tenders/np-notice.do?noticeId=1219083",
                            "publishedDate": "2006-03-21T00:00:00-0500"
                        },
                        "itemsToBeProcured": [{
                            "description": "Construction work",
                            "classificationScheme": "CPV",
                            "classificationID": "45000000",
                            "classificationDescription": "Construction work"
                        }],
                        "totalValue": {
                            "amount": 100000,
                            "currency": "EUR"
                        },
                        "numberOfBids": 3
                    },
                    "awards": [{
                        "awardID": "1129358",
                        "notice": {
                            "id": "1234-5678",
                            "uri": "http://www.dgmarket.com/tenders/np-notice.do?noticeId=1219083",
                            "publishedDate": "2006-03-21T00:00:00-0500"
                        },
                        "awardValue": {
                            "amount": 100000,
                            "currency": "EUR"
                        },
                        "suppliers": [{
                            "id": {
                                "name": "Divers organismes",
                                "uid": "123",
                                "uri": "http://www.contractawards.eu/#!supplier=Divers+organismes"
                            },
                            "address": {
                                "locality": "Bucharest",
                                "region": "Ilfov",
                                "country-name": "Romania"
                            }
                        }],
                        "itemsAwarded": [{
                            "description": "Construction work",
                            "classificationScheme": "CPV",
                            "classificationID": "45000000",
                            "classificationDescription": "Construction work"
                        }]
                    }]
                }
            ]
        }
    ]
}
```
