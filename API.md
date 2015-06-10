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

The endpoint for a beacon resource is the following:
 
``` html
/beacons
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
	"title" = "Control room - ACME Corp - ", // optional, if white-labeled
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

