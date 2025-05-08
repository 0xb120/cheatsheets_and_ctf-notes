---
author: "elttam.com"
aliases: "New Method to Leverage Unsafe Reflection and Deserialisation to RCE on Rails"
tags: RW_inbox, readwise/articles
url: ?__readwiseLocation=
date: 2025-04-24
summary: Researchers discovered a new method to exploit unsafe reflection and deserialization in Ruby on Rails, specifically using the sqlite3 gem. This vulnerability can lead to arbitrary code execution if user inputs are not properly controlled. The findings highlight the risks of using certain Ruby methods and the potential for new security issues in Rails applications.
---
# New Method to Leverage Unsafe Reflection and Deserialisation to RCE on Rails

![rw-book-cover](https://readwise-assets.s3.amazonaws.com/static/images/article0.00998d930354.png)


unsafe reflection in a minimal Rails application and the discovery of a new RCE reflection and deserialisation gadget in the sqlite3 gem. [](https://read.readwise.io/read/01jsf1c2p4qfdp89cyxf3jp58p)



Introduction
 It is a well known security issue that allowing user controlled inputs in calls to the Ruby `String#constantize`, `String#safe_constantize`, `Module#const_get` and `Module#qualified_const_get` methods are insecure, since these methods allow the arbitrary loading of Ruby classes. [](https://read.readwise.io/read/01jsf1cpvtw7n2whrnf66w82ph)



the following code snippet would allow an attacker to initialise an object of any class:
 vuln_params[:type].constantize.new(*vuln_params[:args]) [](https://read.readwise.io/read/01jsf1d45n5s9dxwy6q1vyzyvx)



This unsafe reflection has resulted in code execution vulnerabilities in the past, by initialising a new `Logger` object since the class previously used the Ruby [`Kernel#open`](https://ruby-doc.org/3.4.1/Kernel.html#method-i-open) method for opening the log file, which allowed the execution of terminal commands if the input begun with `|`. [](https://read.readwise.io/read/01jsf1fk7aw46ss4f12yq0hv5x)



It did pique our interest if the above dangerous construct could still result in arbitrary code execution on a minimal installation of Ruby on Rails, which resulted in the discovery of a new method by loading SQLite extension libraries via the `SQLite3::Database` class in the [`sqlite3` gem](https://rubygems.org/gems/sqlite3). [](https://read.readwise.io/read/01jsf1h68hqrx7d5myzx8fgekj)



`SQLite3::Database` reflection gadget could also be exploited in a new deserialisation gadget chain if user inputs were deserialised using `Marshal.load` on a Rails application with the [`sqlite3`](https://rubygems.org/gems/sqlite3), [`activerecord`](https://rubygems.org/gems/activerecord) and [`activesupport`](https://rubygems.org/gems/activesupport) gems installed. [](https://read.readwise.io/read/01jsf1j2cmr9rws3rb14vve339)



Past Research [](https://read.readwise.io/read/01jsf1mphp5v7gem4v53h1yvj1)



• [Conviso’s research into exploiting unsafe reflection in Rails applications](https://web.archive.org/web/20201030010411/https://blog.convisoappsec.com/en/exploiting-unsafe-reflection-in-rubyrails-applications/) highlighted several reflection gadgets, including the `Logger` command execution method that has been patched since 2017.
 • [Justin Collins’s research into `constantize`](https://blog.presidentbeef.com/blog/2020/09/14/another-reason-to-avoid-constantize-in-rails/) documented Gems that had an unsafe `const_missing` method that could be abused to cause memory a leak with only just `params[:class].classify.constantize` as the dangerous construct. [](https://read.readwise.io/read/01jsf1mwxhfeh2nnb5eyywcxjq)



Research Scope and Testing Environment Setup [](https://read.readwise.io/read/01jsf1pa5vk1wbpa41sfgffppb)



the following `Dockerfile` (adapted from [Luke Jahnke](https://nastystereo.com/security/rails-_json-juggling-attack.html)) was used to build the Docker image.
 • `/reflection`: Unsafe reflection of user controlled values to construct a new object.
 • `/marshal`: Unsafe deserialisation of a user controllable value using `Marshal.load`. [](https://read.readwise.io/read/01jsf1pkcwjyncnbpmqtxck145)



Potential RCE Reflection Gadgets
 The first task was to discover classes within installed gems and Ruby that could result in arbitrary code execution when an object is initialised. [](https://read.readwise.io/read/01jsf1qgy17f207g3eb82a81vm)



over 1,000+ potential sinks were raised that were then manually investigated if input parameters for a constructor reached the sink. [](https://read.readwise.io/read/01jsf1rn2p3sffqcg770qj1zcf)



This analysis resulted in the discovery of the following classes that were considered suitable candidates to investigate further.
 • [`RDoc::RI::Driver`](https://github.com/ruby/rdoc/blob/master/lib/rdoc/ri/driver.rb#L402-L436) [](https://read.readwise.io/read/01jskedcne94dcyc5kakxzd5rx)



SQLite3::Database [](https://read.readwise.io/read/01jskedm9rgz2r78hxajthmqm3)



A few other interesting constructor methods in the following classes were also discovered, but since the impact was not code execution they were not analysed further [](https://read.readwise.io/read/01jskedwnn7tjjh3c7zebyekk5)



Writing Files to the Filesystem on Ruby on Rails
 Both the `RDoc::RI::Driver` and `SQLite3::Database` reflection gadgets depend on reading a user controllable file on the filesystem to leverage the arbitrary code execution sinks. [](https://read.readwise.io/read/01jskeeefdsbx47pep1jjwb2fg)



The next option was to look into how Ruby on Rails handles `multipart/form-data` requests with uploaded files. Underneath the hood, Ruby on Rails uses [`Rack`](https://github.com/rack/rack) for handling the processing of web requests and responses. The following code snippet from [`rack/blob/main/lib/rack/multipart.rb`](https://github.com/rack/rack/blob/main/lib/rack/multipart.rb#L48-L66) shows the `parse_multipart` method that processes `multipart/form-data` requests with uploaded files. [](https://read.readwise.io/read/01jskegk2v77v1wfcenn893pgs)



Digging into [`Rack::Multipart::Parser`](https://github.com/rack/rack/blob/main/lib/rack/multipart/parser.rb) class, it shows that a new `TempFile` is created with the contents of the uploaded file with the default prefix of `RackMultipart`. [](https://read.readwise.io/read/01jskegwsw2868bdfchkd1pcz9)



This can be confirmed by sending a file in a `POST` request to a target Rails application, and then observing the temporary file in the `/tmp` folder (default location for temporary files).
 ***NOTE:*** The `/` endpoint does not exist on the test application, but the file is still saved to the filesystem. [](https://read.readwise.io/read/01jskeh34wknac0e6gwhc60ekw)



One problem with this method of file upload is that although the extension value can be controlled by the user, the filename cannot be controlled so the `RDoc::RI::Driver` gadget is no longer a valid candidate for this scenario. [](https://read.readwise.io/read/01jskehphyrjx0zxdn5f3ejaa3)



Exploiting the `SQLite3::Database` Reflection Gadget
 To confirm if `/proc/self/fd/{num}` paths could be loaded as a SQLite extension, first a basic POC library was compiled [](https://read.readwise.io/read/01jskejegjzz6x1fsep967684n)



Uploading the `payload.so` to the application, Rack opened a new file descriptor to the temporary file at `/proc/self/fd/13`. [](https://read.readwise.io/read/01jskejwq84gapkme8x3bbasrr)



Finally, the below `curl` command would load `/proc/self/fd/13` as a SQLite extension when the `SQLite3::Database` object is initialised.
 curl \
 -H 'Content-Type: application/json' \
 -d '{"type": "SQLite3::Database", "args": ["/tmp/rce.db", {"extensions": ["/proc/self/fd/13"]}]}' \
 http://127.0.0.1:3000/reflection [](https://read.readwise.io/read/01jskek5v4k4pj56s1w9fdw5bw)



This confirms that any Rails application with the `sqlite3` gem installed (which is installed by default) that allow the unsafe reflection of user inputs could result in RCE. `Rack` enables an attacker to upload a malicious SQLite extension to the filesystem that is loaded during the construction of a `SQLite3::Database` object by using `/proc/self/fd/x` filepaths, which an external attacker could easily enumerate. [](https://read.readwise.io/read/01jskema3dp579c1h57s2efmng)



Adapting the `SQLite3::Database` Reflection Gadget into a Deserialisation Gadget
 The next challenge was to investigate if `SQLite3::Database` objects could be leveraged in a deserialisation gadget chain to achieve RCE on a Rails application. [](https://read.readwise.io/read/01jskemph29vbj5hwkkzwyt866)



it was discovered that the [`ActiveRecord::ConnectionAdapters::SQLite3Adapter` class](https://github.com/rails/rails/blob/9f16995fb9556228ad88a3799b0e349ef6f6e0c7/activerecord/lib/active_record/connection_adapters/sqlite3_adapter.rb) initialises a new `SQLite3::Database` object when the `connect!` method is invoked [](https://read.readwise.io/read/01jskenay3x0qn3cvm1s5hk011)



Using the infamous [`ActiveSupport::Deprecation::DeprecatedInstanceVariableProxy`](https://github.com/rails/rails/blob/9f16995fb9556228ad88a3799b0e349ef6f6e0c7/activesupport/lib/active_support/deprecation/proxy_wrappers.rb#L97) gadget, a deserialised `DeprecatedInstanceVariableProxy` object could be used to invoke the `connect!` method of a deserialised `ActiveRecord::ConnectionAdapters::SQLite3Adapter` object, which was first [utilised back in 2013 by Hailey Somerville to exploit CVE-2013-0156](https://github.com/haileys/old-website/blob/master/posts/rails-3.2.10-remote-code-execution.md). [](https://read.readwise.io/read/01jskep4p0pbca3qepq10fqvt5)



The following `curl` command then exploits the `Marshal.load` vulnerability on the `/marshal` endpoint on the test application that results in the initialisation of the `SQLite3::Database` object that loads the SQLite extension.
 curl -H 'Content-Type: application/json' \
 -d '{"deserialise":"'$(ruby sqlite-marshal-poc.rb 12 | tr -d '\n')'"}' \
 http://127.0.0.1:3000/marshal [](https://read.readwise.io/read/01jskeq1kbcbyngxprptpsvz0s)

