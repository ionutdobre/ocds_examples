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
7. Examples

## 1. API Conventions

### Authentication and Usage Limits
OCDS API doesn’t require any kind of authentication.Users can send as many requests as they want, the only limitation will be the number of records returned; in this way we encourage developers to use pagination in order to iterate through all of the requested data.

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

*You should be aware that http://www.contractawards.eu web application supports the OCDS standard but it is not part of the standard*.

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
* ```item``` - Sector, should be an object, e.g., *{"classificationScheme": "CPV", "classificationID": "45000000", "classificationDescription": "Construction work"}*
* ```year``` - Year, e.g., *2012*
* ```contractType``` - Contract Type, e.g., *Service contract*
* ```awardCriteria``` - Award criteria, e.g, *Lowest price*
* ```buyerCountry``` - 2 digits country code, e.g., *de* for Germany 
* ```supplierCountry``` - 2 digits country code, e.g., *it* for Italy

Please note that the last 6 criteria (```sector```, ```year```, ```contractType ```, ```awardCriteria```, ```buyerCountry``` and ```supplierCountry```) need to be valid. You can check the validity by invoking the **Metadata API**.

**Request pagination**

Each request accepts the following parameters:
* ```page``` – page number
* ```pageSize``` – number of records per page; the default value is 100 and it cannot be greater than 1000

For example,
```
http://www.contractawards.eu/open-contracting/api/v1/record-package?year=2008&buyerCountry=Germany&page=10&pageSize=20
```

The example above returns the 10th 20-record page.

```
http://www.contractawards.eu/open-contracting/api/v1/record-package?year=2008&supplierCountry=France&page=1&pageSize=50
```

```
http://www.contractawards.eu/open-contracting/api/v1/record-package?sector=Construction work&procedure=Service contract&year=2012&page=1&pageSize=1000
```

### Limitation of the Filter API implementation

1.	Supplier name and buyer name reset all other parameters, in other words if one of the ```supplier``` or ```buyer``` parameter is present all other parameters will be ignored.
2.	All other parameters (```sector```, ```year```, ```procedure```, ```awardCriteria```, ```buyerCountry```, ```supplierCountry```) can be combined in any order with the condition that at least two criteria from the aforementioned list be used.

### Release content returned by the API
There two ways to include a release into a record package:
* as a URI pointing to the release
* as the actual release content by inserting ```embed=true``` parameter.

For example, the following request would retrieve just basic information about a record package, with release URIs that can be called to obtain the actual release information:

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

### Obtaining a single record
Each record has a unique **Open Contracting ID** called ```ocid``` (http://ocds.open-contracting.org/standard/r/0__3__3/#conceptual-model). A particular record can be retrieves via the following API:

```
http://www.contractawards.eu/open-contracting/api/v1/record?recordId=<ocid>
```

### Pretty print format
By default, the API provides pretty printing of the JSON output. In a production application a user can reduce the cost of  data transfer by using the parameter ```pretty=false``` in order to remove all the white spaces from the response.

### Records API Response
Records contain the following information:

* ```uri``` - URI of the records package
* ```publisher``` - information that uniquely identifies the publisher of this package
* ```publishedDate``` -package publication date
* ```packages``` - a list of URIs of release packages included in the record package
* ```records``` - records included in this records package

## 3. Releases API

This API provides information about a package release or a single release. It can be used to obtain information related to a particular stage in the contracting process.

### Basic call and parameters

Obtain a release package:
```
http://www.contractawards.eu/open-contracting/api/v1/release-package?releaseId=<id>
```

Obtain an individual releaze:
```
http://www.contractawards.eu/open-contracting/api/v1/release?releaseId=<id>
```

**Pretty format** - the **Release API** supports the same ```pretty=false``` parameter as **Records API**; it triggers white-space compression of the response.

### Release API Response
Releases contain the following information:

* ```ocid``` - A unique identifier of an Open Contracting Process (the same ID can be used for ```noticeID``` and ```TenderID``` should always have the same value as OCID)
* ```releaseID``` - A unique release identifier
* ```releaseDate``` - The date this information was released
* ```releaseTag``` - A tag that helps to identify the type of data in the dataset (e.e, *awardNotice*)
* ```language``` - The default language of the data returned
* ```formationType``` - String specifying the type of formation process used for this contract(should be *tender*)
* ```buyer``` - An entity that procures goods, works or services.
	* **id**
		* *name*
		* *uid*
		* *uri*
	* **address** - An Address following the convention of http://microformats.org/wiki/hcard
		* *locality*
		* *region*
		* *country-name*
* ```tender``` - Activities undertaken in order to enter into a contract.
	* **tenderID** - TenderID should always be the same as the OCID.
	* **notice** - A notice is a public document that notifies the public about various stages of the contracting process
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
* ```awards``` - Information related to the award phase of the contracting process
	* **awardID**
	* **notice**
		* *id*
		* *uri*
		* *publishedDate*
	* **awardValue**
		* *amount*
		* *currency*
	* **suppliers** - The awarded suppliers
		* *id*
			* *name*
			* *uid*
			* *uri*
		* *address*
			* *locality*
			* *region*
			* *country-name*
	* **itemsAwarded** - The goods and services included in the awarded contract, broken into line items wherever possible
		* *description*
		* *classificationScheme*
		* *classificationID*
		* *classificationDescription*
* ```planning``` - Information from the planning phase of the contracting process.
	* **budgetID**
	* **budgetAmount**
	* **publicHearingNotice**
	* **strategicJustificiation**
	* **anticipatedMilestones**
* ```contracts``` - Information from the contract creation phase of the procurement process.
	* **contractID**
	* **awardID**
	* **contractPeriod**
	* **contractValue**
	* **signatureDate**
	* **itemsContracted**
	* **attachments**
* ```performance``` - Information related to the implementation of the contract in accordance with the obligations laid out therein.
	* **transactionDataPackageURI**
	* **transactionID**
	* **transactionAmount**

## 4. Metadata API

**Getting filters metadata**

This API returns information about ```filters``` that can be used to query **Records API**

### Basic call and parameters
```
GET /open-contracting/api/v1/<dataset>
```

This call returns information about a ```dataset``` and all valid values that can be used to query it. If a value is not returned, for example, year 2001, then it means that there are no contracts published in 2001.

Possible ```dataset``` values include:
* ```years``` - returns an array of valid years
* ```buyerCountries``` - returns all buyer countries
* ```supplierCountries``` - returns all supplier countries
* ```sectors``` - returns all sectors
* ```contractType``` - returns all  contract types (e.g., Service contract, Works)
* ```awardCriteria``` - returns all award criteria (Lowest price, The most economic tender, etc.)

#### Response example
```
GET http://www.contractawards.eu/open-contracting/api/v1/years

{
hits: [2006, 2007, 2008, 2009, 2010, 2011, 2012, 2013]
}
```

The above response can be viewed in a web browser. When using Ajax, one can obtain a list of years in a dataset as follows:

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

Change is inevitable, and the versioning API helps iterate faster and prevent invalid requests. This also allows to adopt new version of OCDS standard while continuing to support old API versions for a some time. Versions are identified as ```/api/v1/```, ```api/v2/``` and so on.

## 6. Errors

In case of an error (on client or server side) a response will contain JSON error body that provides:  a useful error message, an error code, and the URL that caused the error. For example:

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

## 7. Examples

The following is an example of **Records API** response:

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
