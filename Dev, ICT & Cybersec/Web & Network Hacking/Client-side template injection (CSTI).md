>[!question] What is Client-side template injection?
>Client-side template injection vulnerabilities arise when applications using a client-side template framework dynamically embed user input in web pages. When rendering a page, the framework scans it for template expressions and executes any that it encounters. This issue can be exploited to conduct [Cross-Site Scripting (XSS)](Cross-Site%20Scripting%20(XSS).md) attacks, even if user input is encoded.

# AngularJS CSTI

Examples using [AngularJS](../Dev,%20scripting%20&%20OS/AngularJS.md)

Identified the application uses angular and angular templates:
```html
...
<body ng-app="labApp" class="ng-scope">
...
<section class="blog-header">
	<script>angular.module('labApp', []).controller('vulnCtrl',function($scope, $parse) {
		$scope.query = {};
		var key = 'search';
		$scope.query[key] = '{{1+1}}';
		$scope.value = $parse(key)($scope.query);
	});</script>
	<h1 ng-controller="vulnCtrl" class="ng-scope ng-binding">0 search results for {{1+1}}</h1>
	<hr>
</section>
```


# External References
-  [XSS without HTML: Client-Side Template Injection with AngularJS](https://portswigger.net/research/xss-without-html-client-side-template-injection-with-angularjs)