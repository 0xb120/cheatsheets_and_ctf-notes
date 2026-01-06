---
author: Antoine Gicquel
aliases:
  - Magento for Security Audit
tags:
  - RW_inbox
  - readwise/articles
url: https://www.synacktiv.com/publications/magento-for-security-audit.html?__readwiseLocation=
created: 2025-04-30
---
# Magento for Security Audit

![rw-book-cover](https://www.synacktiv.com/sites/default/files/styles/blog_grid_view/public/2023-09/magento_1_1.jpg)

## Highlights


This blog post tries to answer most of the questions one would have when encountering Magento for the fist time in a security context [](https://read.readwise.io/read/01jt3b31323rd35e640dkbpczc)



• How is authentication handled ? How are authorization checked on routes / actions ?
 • How are user inputs handled ?
 • Does the framework use serialized objects ? Are they passed to clients ?
 • Is there a templating engine in use ? How does it work ?
 • What can you do once you have acquired admin access to the CMS ? How can you persist ? What can you loot ? Can you execute arbitrary code on the underlying server ? [](https://read.readwise.io/read/01jt3b3m9t5qz7n37554kqatnm)



Magento 101Application structure [](https://read.readwise.io/read/01jt3b5713cayjwk4kk0y09b8e)



• `/app/` : contains the components of the CMS
 • modules in `/app/code`
 • themes in `/app/design`
 • language packs in `/app/i18n`
 • `/lib/` : contains the libraries used by the application. In particular, `/lib/Magento/Framework` contains the core classes of the CMS (app engine, class factories, database connections, …)
 • `/pub/` : this is the folder exposed by the web server. Contains mostly `index.php`, the entry point for every request received [](https://read.readwise.io/read/01jt3b6ytv1936vj59dy5wqfae)



When auditing a custom Magento application, the specific code will most likely be located under the `/app/code` or `/app/design` folder, with maybe some of it in `/lib` for lower-level generic code. [](https://read.readwise.io/read/01jt3b7gsem8rswfmbh0a36np5)



You can easily find the version of Magento your custom application is based on by looking in the `composer.json` at the root of the project. [](https://read.readwise.io/read/01jt3b82qfse5kz4q05sepw3kj)



Modules, stored in `/app/code`, are key components of Magento and are made of:
 • `/app/code/<Module_Vendor>/<Module_Name>/registration.php` : this file is required at the root of the module, and is used by Magento to register the module classes to the core engine when the application starts
 • `/app/code/<Module_Vendor>/<Module_Name>/etc/` : contains XML configuration files. Note that these can be overridden by area in the corresponding `etc/[area]/` folder [](https://read.readwise.io/read/01jt3b9gyqkw10kfrcrmvqpv0j)



• `module.xml` : defines the module and its dependency on other modules
 • `routes.xml` : registers the URI paths that this module can handle
 • `acl.xml` : defines custom resources in the “Role Resources” menu (which define what actions a given role can execute, see ACL)
 • `di.xml` : specifies the arguments and attributes of objects for the Object Manager (see Dependency Injection)
 • `config.xml` : contains the configuration of the module (like the module options) [](https://read.readwise.io/read/01jt3bahvvdrs9t92cd5vs24mh)



Different “areas”
 In order to improve its efficiency and security, Magento splits the applicative part in multiple sections called areas [](https://read.readwise.io/read/01jt3bbagv2xszv1p6ed5kxefv)



`frontend` : customer-facing website, with the products view, the payment, the account settings, … This area is accessible by anyone. [](https://read.readwise.io/read/01jt3bbwk8mt09y6y0v8yyex0k)



`adminhtml` : administration panel, with menus and options aimed at website administrators. This is accessible by administrators only, after having logged in on the admin panel. [](https://read.readwise.io/read/01jt3bcmd56m94a8q9nj2cjnfa)



`crontab` : things which must run in cron tasks either via the `/pub/cron.php` file or via the `bin/magento cron:run` shell command [](https://read.readwise.io/read/01jt3bd2pdw1hja5ezecm1466q)



PIs (`graphql`, `webapi_rest`, `webapi_soap`) : APIs, which are accessible via routes defined in a module’s `etc/webapi.xml` Indeed, some modules act on multiple of these areas, and may have different requirements depending on it. For example, the same module called in the context of `webapi_rest` may not load the HTML rendering classes, while it would in the context of `frontend`. As mentioned before, modules can override the general configuration files in `[module_path]/etc/` with area-specific configuration files in `[module_path]/etc/[area]/`. [](https://read.readwise.io/read/01jt3be5j4jned80rnvycr1jte)



Admin panel login page [](https://read.readwise.io/read/01jt3bh7k6d88masn4rqnjj59n)



By default, the URI of the admin login page is randomized and displayed on the terminal when starting the Magento application. [](https://read.readwise.io/read/01jt3bhebj78y2cx8anvx5vv1t)



The random URI might look something like [](https://read.readwise.io/read/01jt3bhv9chd3jjnq64q7n9c9s)



/admin_[0-9a-z]{6} [](https://read.readwise.io/read/01jt3bj0n7va3tx20nx02c8d79)



and is stored in `/app/etc/env.php`. [](https://read.readwise.io/read/01jt3bja0mjzsqm1c7e89s0jm9)



this file also contains some juicy data such as the main encryption key, the database credentials and other stuff. You might want to exfiltrate it if you found a way to read arbitrary files on the server. [](https://read.readwise.io/read/01jt3bjsczwv89ab6z9rc3dywg)



Also note that in the case it isn’t changed, the generated random URL does not follow a uniform probability distribution. Indeed, due to a flawed random generation, the first digit has a >50% chance of being a `1`, which drastically reduces the average number of requests needed to find the correct URL. [](https://read.readwise.io/read/01jt3bm6385k6bdj7dwf8s21d2)



Authorizations and ACL
 The ACL system of the Magento backend is pretty standard [](https://read.readwise.io/read/01jt3bmsv3qsdwqpj1k76trmy5)



: users are assigned roles, and roles are assigned “resources”, which are atomic permissions. A module can define its resources in `etc/acl.xml`[7](https://www.synacktiv.com/publications/magento-for-security-audit.html/magento-for-security-audit#footnote7_ez24dua), and base its authorizations on these resources. [](https://read.readwise.io/read/01jt3bnkr37nq141hgy482g9cf)



In order to properly apply a permission restriction on a controller, a number of distinct actions are needed:
 • first, the button / link to the controller must be hidden to the users which do not have the permission to use this controller,
 • then, the controller itself must restrict its execution to authorized users only, by setting the `const ADMIN_RESOURCE` at the top of the controller’s file. This is handled properly if the controller inherits from the `Magento\Backend\App\Action` class,
 • if the controller is also accessible via an API area, its access must be restricted in the `etc/webapi.xml` configuration file. [](https://read.readwise.io/read/01jt3bpvjym29z2q1j0e3ht86y)



some of these actions are less obvious than others, as a developer could do only the first one and think it is sufficient. In the case you encounter a custom module, you should definitely check all these elements to properly validate ACL rules. [](https://read.readwise.io/read/01jt3bqj782jf6ymjtvvb4sxp8)



When assigning resources, be sure to disable access to the Permissions tool if you are limiting access for a given role. Otherwise, users are able to modify their own permissions. [](https://read.readwise.io/read/01jt3brayjkpxvjvpd2aty0f4e)



Routing engine
 The routing engine parses URI in the following way :
 [area]/[frontName]/[controller folder]/[controller class]
 The `frontName` refers to a module’s name (as defined in modules `routes.xml`), and the mapping is done by parsing every module’s `etc/[area/]routes.xml` file. [](https://read.readwise.io/read/01jt3bseg5k339tkp5gykh3t27)



The next URI parts are optional and refer to the path of the file that will be invoked inside the `Controller/` folder of the matching module. If they are not defined, the `Index/Index.php` file is called. [](https://read.readwise.io/read/01jt3bszh6kpz2tm43axc1ha6k)



Plugins
 The plugins mechanism[8](https://www.synacktiv.com/publications/magento-for-security-audit.html/magento-for-security-audit#footnote8_j1i8c61) in Magento is not as straightforward as one could think. Indeed, plugins act on modules and hook specific functions to run code before, after and around the original function. Plugins can sometimes be referred to as *Interceptors* in the Magento documentation and code, and you can think of them as a form of instrumentation of the original code. [](https://read.readwise.io/read/01jt3btxba295ph5jqe2dxynh9)



Plugins register their hooks inside the `etc/di.xml` file of the module they live in.
 <config>
 <type name="{ObservedType}">
 <plugin name="{pluginName}" type="{PluginClassName}" sortOrder="1" disabled="false" />
 </type>
 </config> [](https://read.readwise.io/read/01jt3bvfd49gqe41dzz4hcangt)



only the hooked *class* is specified and not the hooked *methods*. [](https://read.readwise.io/read/01jt3bx4bbw6zn11kjn7ybahhc)



The hooked methods are determined at runtime from a naming convention: when calling a method `myMethod` from a hooked class, the app engine looks for functions named `beforeMyMethod`, `aroundMyMethod` and `afterMyMethod` [](https://read.readwise.io/read/01jt3by38tgk2ay4txm94ggr2p)



Serializers
 Magento possesses a couple of custom Serializer classes[9](https://www.synacktiv.com/publications/magento-for-security-audit.html/magento-for-security-audit#footnote9_mki86l4). They all implement the `Magento\Framework\Serialize\SerializerInterface` interface (which declares the `serialize($data)` and `unserialize($data)` methods) [](https://read.readwise.io/read/01jt3byxrynzewmwq0v8pecd5p)



The default serializer implementation is the JSON serializer, however the `Serialize` serializer (which uses the infamous native PHP `serialize` and `unserialize` functions) is also available to modules and used in certain cases in core Magento classes, mostly in deprecated methods and legacy code. [](https://read.readwise.io/read/01jt3c07hmsg2c04x4rrrtv6ab)



Storage
 Magento documentation[10](https://www.synacktiv.com/publications/magento-for-security-audit.html/magento-for-security-audit#footnote10_2s2lta4) advises developers not to store files unless it is strictly necessary. [](https://read.readwise.io/read/01jt3c0qw8faww41jxkds5wnz9)



Operations on the filesystem are done through the `Magento\Framework\Filesystem` core library [](https://read.readwise.io/read/01jt3c17mjv3m6d1rypmbs4266)



publicly accessible files [](https://read.readwise.io/read/01jt3c1b4bw2ahdw4dxtmpsjeh)



are stored in subdirectories of `pub/media/`. [](https://read.readwise.io/read/01jt3c1jj3dbxmdry2k87n0sg0)



storage of secrets and configuration options [](https://read.readwise.io/read/01jt3c2v27fh3w67a41k8a5dnq)



in the `app/etc` folder [](https://read.readwise.io/read/01jt3c352jrthcsf606e6nyfjb)



database credentials [](https://read.readwise.io/read/01jt3c39pgd84e24nccaw0r7k2)



in `app/etc/env.php` [](https://read.readwise.io/read/01jt3c3d7zfh5f9qgx6a6y5w3v)



Other configuration options might be overwritten by the `core_config_data` table in the database. [](https://read.readwise.io/read/01jt3c4nckmtbrcdb6rvg6ttrt)



For the database, Magento uses the Entity Attribute Value (EAV) model[11](https://www.synacktiv.com/publications/magento-for-security-audit.html/magento-for-security-audit#footnote11_3dntxps), which is described in details [here](https://en.wikipedia.org/wiki/Entity%E2%80%93attribute%E2%80%93value_model). [](https://read.readwise.io/read/01jt3c54knre9mtzn908y32qe4)



data objects (like customers, products, UI blocks, …) are stored as rows in a table named `xxxx_entity` with `xxxx` being the object name, e.g. `catalog_product`, with a set of common attributes. [](https://read.readwise.io/read/01jt3c5sr0q30y3nvc84w1xdys)



Then, [](https://read.readwise.io/read/01jt3c67sdh31v0mk5rtmkxdvn)



custom attributes specific to one entity [](https://read.readwise.io/read/01jt3c6h77f4e21tyfh1y8d0qa)



are stored in other tables depending on the type of the attribute [](https://read.readwise.io/read/01jt3c6vqaj8ftkszbeqtaapp7)



For instance, if you wanted to add a warranty to a product, you could have a table `catalog_product_entity_int` with a content like:
 Entity_ID
 Attribute
 Value
 4
 ...
 ...
 5
 years_of_warranty
 3
 8
 ...
 ... [](https://read.readwise.io/read/01jt3c7x8hhakp016gjxgnkm88)



tables containing potentially sensitive data [](https://read.readwise.io/read/01jt3c8613qswrdf6bkcr6bnx6)



• `admin_user`: list of users registered on the backend. If their passwords are not there, they are stored in the`admin_passwords` table
 • `admin_user_session`: last few sessions on the backend. If you are trying to be stealth, clean this table after your actions
 • `authorization_role`: to check on ACL roles
 • `authorization_rule`: links ACL resources to rules
 • `customer_log`: login/logout logs of customers
 • `customer_entity`: user data (login, password_hash)
 • `eav_attribute`: A few store attributes
 • `cms_page`: content and metadata of blog pages
 • `cron_schedule`: list of future cron jobs scheduled as well as past cron jobs with their status
 • `core_config_data`: previously mentioned
 • `login_as_customer`: used for assistance, an admin can log in as a customer to help him with actions on his account
 • `magento_login_as_customer_log`: logs of login_as_customer actions
 • `persistent_session`: users can persist their cart data in the application, and the data will get stored here, with a hashed identifier to get it back the next time they come
 • `quote_payment`: contains credit card data
 • `session`: customer session data is stored here, JSON-serialized
 • `url_rewrite`: lists URL-rewriting rules [](https://read.readwise.io/read/01jt3c8g9d1ezzjk0t4wsqmwtz)



Post-exploitation [](https://read.readwise.io/read/01jt3c8yd205hp2h9wj6srb4gv)



Is there an easy *RCE-as-a-service* like the plugins mechanism on WordPress ? Sadly, the answer is **no**. [](https://read.readwise.io/read/01jt3c9cfa9ent1jt77qsdjcej)



Every sensitive maintenance operation, like the plugin installation procedure or crontab actions is done via the command line interface using the `bin/magento` utility. [](https://read.readwise.io/read/01jt3caa04gk3rkpq3mkqmpk65)



However, the attack surface of the admin panel is still pretty large, with some templating available as well as multiple sensitive features. [](https://read.readwise.io/read/01jt3cbt2sjv68c59fdp3p8a3a)



Component analysis methodology
 Once you have picked a target component, the first thing you can do is to check its configuration files, with a focus on `etc/[*/]di.xml` and `etc/[*/]routes.xml`. These files will point you to the URL paths that are handled by this module, as well as the underlying objects and classes it uses. [](https://read.readwise.io/read/01jt3ccs8786tyvrjag67dz242)

