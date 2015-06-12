#API

ProximitySense offers a RESTful API. The root url for accessing the API is: 

``` html
https://platform.proximitysense.com/api/v1
```
Access is only available through **HTTPS** - HTTP is not allowed.

**_Important!_**

Don't forget to include the **Content-Type: application/json;** HTTP header in your request

``` html
Content-Type:application/json; charset=utf-8
```

## Return codes

Usually HTTP response codes in the **2xx** range mean success. The ones in the **4xx** range usually are errors. 
If you receive a code in the **5xx** range, that's usually a bug on our side - please contact us to make sure we fix it.
 

## Authentication

ProximitySense API implements a simple mechanism for authentication, based on sets of public/private token pairs. 
In order to obtain an API authentication pair you need to create an **Application** in ProximitySense.
The system will generate an authentication pair for you after successfully creating an **Application**.

Every call to an API endpoint needs to be authenticated. An authenticated call needs to contain two HTTP headers - 
**X-Authorization-ClientId** and **X-Authorization-Signature**

``` html
X-Authorization-ClientId: (Application id)
X-Authorization-Signature: (Message signature)
```

The **X-Authorization-ClientId** HTTP header needs to contain the Application id of the ProximitySense Application you are making the call on behalf of.
You can obtain that from the details dialog, after successfully creating an **Application**. The id is a hexadecimal string, looking like this: _8e83ddb95b954fe9beb1b6cba9880e32_

The **X-Authorization-Signature** HTTP header contains the SHA256 signature of the message. It is calculated using the following algorithm:

1. Concatenate the following in order: (full url) + (message content - if any) + (Application Private Key)

 _For GET requests, the message content component is skipped, so you need to concatenate only the URL and the Application Private Key_
 
2. Calculate the **SHA256** hash of the resulting string.

The signature is checked using the same algorithm on the server. If the provided signature does not match the server signature, error is returned - 
usually it is *401 Unauthorized*.


## Ownership

ProximitySense has an internal ownership system that manages entities' ownership and scope. There are two scopes - Organisation Scope and Apllication Scope.
Objects such as Beacons and Applications can be owned by individual users or by Organisations.
Entities such as Venues, Zones and Campaigns are owned only by Applications, hence they have only Application Scope.
  
When accessing the API and requesting information for entities, outside of application scope please consider that ProximitySense will infer
the owning entity from the application used to make the call.

Consider the following situation:

1. You own beacons with serial numbers 1 and 2. 

2. You are also member of an Organisation, that ownes beacons with serial numbers 3, 4 and 5.

3. Both you and the organisation have Applications created - application "Test App" owned by you and application "Organisation Test app", owned by the organisation.

**Scenario 1:**

* You make a request to retrieve all beacons, using credentials from your own application - "Test App"

* The result is: beacons with serial numbers 1, 2, 3, 4 and 5 - the first two because they are directly owned by you, the rest because they are owned by
an Organisation you are member of.

**Scenario 2:**

* You make a request to retrieve all beacons, using the organisation app credentials - "Organisation Test App"
* The result is: beacons with serial numbers 3, 4 and 5 - because the application used to make the call can only access beacons that are owned by it's parent Organisation.


## API Entities


### Analytics

The endpoint for an analytics resource is the following:
 
``` html
/analytics
```


### Applications

The endpoint for an application resource is the following:
 
``` html
/applications
```


### Audience

The endpoint for an audience resource is the following:
 
``` html
/audience
```


### Beacons

The beacons endpoint allows users to retrieve beacon data. A successfull response consists of an **array** of **Beacon** objects. 
Only beacons that are owned by the owner of the calling Application are returned. 
 
``` html
API Endpoint: /beacons{/serialNumber}
Allowed methods: GET, POST
```

A **Beacon** object has the following fields:

| Field name  				| Type  		| Sample value 			| Read/Write	| Note 		|
| :------------------------ |:--------- 	| -----:				| :-----------:	||
| id      					| integer 		| 1 					| R			||
| serialNumber      		| string 		| "123-test" 			| R			||
| name      				| string 		| "Sample beacon 1"		| R/W			||
| description      			| string 		| "description" 		| R/W			||
| manufacturer      		| string 		| "Blue Sense Networks" | R			||
| isInSync      			| boolean 		| false					| R|true if latest configuration changes were applied successfully 	|
| pincode      				| string 		| 0ABCDEF1				| R|Hexadecimal security code, used to authenticate when applying beacon configuration|
| uuid      				| uuid / guid 	| "A0B13730-3A9A-11E3-AA6E-0800200C9A66" 					| R/W|iBeacon UUID|
| major      				| integer 		| 100 					| R/W|iBeacon Major (0-65535)		|
| minor      				| integer 		| 101 					| R/W|iBeacon Minor (0-65535) 		|
| advertisingInterval      	| integer 		| 300 					| R/W|In miliseconds				|
| broadcastPower      		| integer 		| 8 					| R/W|Valid range is 0-9 for long range, and 0-15 for standard models |
| batteryLevel      		| integer 		| 80											| R|values are in percent, valid range is 0-100	|
| createdOn      			| date 			| "2015-06-11T10:57:50.0650585+01:00"			|R||
| userMetadata      		| dictionary 	| { "key1":"value1", "key2":"value2" } 			|R/W|||

``` json
{
	"id":"1", // integer number
	"serialNumber":"123", // can be used to query for this beacon
	"name":"Sample Beacon 1",
	"description":null,
	"manufacturer":"Blue Sense Networks",
	"isInSync":false, // true if latest configuration changes were applied successfully
	"pincode":"A0BCDEF1",
	"uuid":"A0B13730-3A9A-11E3-AA6E-0800200C9A66",
	"major":100,
	"minor":200,
	"advertisingInterval":360, // in miliseconds
	"broadcastPower":8,
	"batteryLevel":90, // in percent
	"createdOn":"2015-06-11T10:57:50.0650585+01:00",
	"userMetadata": {
		"key1":"value1",
		"key2":"value2",
	}
}
```

####Getting all beacons

Calling the endpoint without specifying a serial number retrieves all beacons, owned by the owner of the calling Application. 
The order they are returned in is indeterminate at the moment, so don't rely on specific ordering. 

Sample response looks like this:

``` json
[
	{
		"id":"1",
		"name":"Sample Beacon 1",
		"serialNumber":"123",
		...
	},
	...
	{
		"id":"22",
		"name":"Sample Beacon 22",
		"serialNumber":"234",
		...
	}

]
```

####Getting a single beacon

When you need to retrieve data for only a specific beacon, append the beacon serial number to the request url, like this:

``` html
GET: /beacons/123abc
```

####Updating beacons' details

Updating beacons' details is done by making a POST request to the same endpoint. The passed in array of one or more **Beacon** objects
must have their respective serial number fields correctly filled. Only fields marked with R/W are updated, if you pass in fields that are read-only, or don't exist
in the object definition they will be ignored.


``` html
POST: /beacons
Content: [{Beacon}, {Beacon} ... {Beacon}]
```


### Campaigns

The endpoint for a campaign resource is the following:
 
``` html
/campaigns
```


### Profile

The profile endpoint allows users to retrieve profile data for the owner of the calling Application. A successfull response is either
a **User** object, or an **Organisation** object.
 
``` html
API Endpoint: /profile
Allowed methods: GET
```

Sample response for a **User** object:
 
``` json
{
	"id"="1234", // integer value
	"isActive" = true,
	"screenName" = "user123",
	"profileImageUrl" = "https://url-to-profile-image.com/profile.jpg",
	"email" = "someone@domain.com",
	"address" = "1st Street, Somewhereville",
	"country" = "UK",
	"phone" = "00123456789",
	"company" = "ACME Corporation ltd",
	"website" = "https://acmecorporation.com",
	"createdOn" = "2015-06-10T18:25:43.511Z",
	"plan" = "pro" // optional
}
```


Sample response for an **Organisation** object:
 
``` json
{
	"id"="1234", // integer value
	"name" = "ACME Corporation",
	"description" = "beep - beep",
	"accessUrl" = "https://acme.proximitysense.com/", // optional, if white-labeled
	"title" = "Control room - ACME Corp", // optional, if white-labeled
	"logoUrl" = "http://acme.proximitysense.com/logo.png", // optional, if white-labeled
	"createdOn" = "2015-06-10T18:25:43.511Z",
}
```
 

### Supernodes

The endpoint for a supernode resource is the following:
 
``` html
/supernodes
```


### Venues

The endpoint for a venue resource is the following:
 
``` html
/venues
```


### Zones

The endpoint for a zone resource is the following:
 
``` html
/zones
```

