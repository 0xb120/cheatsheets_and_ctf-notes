---
author: "Antoine Gicquel"
aliases: "Web Architect - An Introduction"
tags: RW_inbox, readwise/articles
url: https://www.synacktiv.com/publications/web-architect-an-introduction.html
date: 2025-01-14
---
# Web Architect - An Introduction

![rw-book-cover](https://www.synacktiv.com/sites/default/files/styles/blog_grid_view/public/2023-09/intro.jpg)

## Highlights


Code paths and Routing
 When a request is received by a CMS, the code execution typically goes in the following order. The entry point file (`index.php` for a PHP CMS as an example) is loaded, which includes the necessary code and configuration files to bootstrap the CMS environment, variables, classes and functions. The routing system then determines the appropriate controller or action based on the requested URL, using a mapping configuration. The controller is then invoked to handle the request, performing operations and interacting with models, databases or other components as needed. Once the necessary data is obtained, the controller prepares it to be rendered and passes it to the view layer. Views, which are usually templates containing HTML, CSS, and dynamic placeholders, are then rendered using a templating engine. The engine replaces the placeholders with the actual data, resulting in the generation of HTML output.
 ![Code path of an usual request](https://www.synacktiv.com/publications/web-architect-an-introduction.html/../sites/default/files/inline-images/codepath_2.png)
 Code path of a usual request
 Sessions and authentication data is usually handled during the bootstrapping phase, setting the right variables in the CMS environment. It is then to the responsibility of the controller to check whether the user is authenticated (through the previously set CMS variables) and to enforce ACL rules.
> [View Highlight](https://read.readwise.io/read/01jhj52a6s61qkaxp2h64ryvbg)



