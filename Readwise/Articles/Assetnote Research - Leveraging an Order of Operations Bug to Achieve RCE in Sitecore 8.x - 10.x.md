---
author: Assetnote Research
aliases:
  - Leveraging an Order of Operations Bug to Achieve RCE in Sitecore 8.x - 10.x
tags:
  - readwise/articles
  - dotnet
url: https://www.assetnote.io/resources/research/leveraging-an-order-of-operations-bug-to-achieve-rce-in-sitecore-8-x---10-x?__readwiseLocation=
date: 2025-04-24
---
# Leveraging an Order of Operations Bug to Achieve RCE in Sitecore 8.x - 10.x

![rw-book-cover](https://cdn.prod.website-files.com/6422e507d5004f85d107063a/649c2686dd142039d6d5ea8e_Frame%201428.png)


## What is an Order of Operations bug?

Before we get into the details of the vulnerability itself, we wanted to spend some time exploring the concept of **order of operation** bugs. When writing code, especially code that is responsible for security boundaries, the order in which these security boundaries are applied is extremely important. Often logic flaws within these flows can lead to devastating security impact. [](https://read.readwise.io/read/01js4smx56t4ns0cw62w6mmc7b)

Let's consider the following code snippet: [](https://read.readwise.io/read/01js4sq9y6jwv7jget75m96a3d)
```python
from werkzeug.utils import secure_filename
from enterprise.utils import decrypt_str

def decrypt_value(encrypted_str):
	return decrypt_str(encrypted_str)

@app.route('/upload', methods=['POST'])
def upload_file():
	file = request.files['file_upload']
	encrypted_file_path = request.form['file_path']
	file_path = decrypt_value(secure_filename(encrypted_file_path))
	file.save(f"/var/www/images/{file_path}")
```

In the case above, we can see that there is some logic that is supposed to prevent path traversal through the use of `secure_filename`, however, the order of operations is incorrect. [](https://read.readwise.io/read/01js4srn4aqh0dbj98b92w00d9)

The `secure_filename` operation will not prevent path traversal as it is occurring on an encrypted string that is then being decrypted, after being sanitized. The sanitization would have no impact on an encrypted string. [](https://read.readwise.io/read/01jskesajyde4d52g750fsf0mk)

### Finding an Order of Operations bug in Sitecore

The `/-/speak/v1/bundles/bundle.js` endpoint allows for arbitrary file read if an absolute path is used. This is possible because the query parameter specifying the file is not properly normalized before it is verified. The input is also modified after verification resulting in file extension checks being bypassable. [](https://read.readwise.io/read/01jsketnhs1afy2eb7y0rffyd4)

The following diagram represents the flow of this bug:
![](https://cdn.prod.website-files.com/64233a8baf1eba1d72a641d4/6740047cd904d89c7a59a75c_674004762d51e7789c28c021_diagram%25203%2520(2).png) [](https://read.readwise.io/read/01jskexcdayj2we9yh7snbtaj6)

### Impact

Local file disclosure vulnerabilities on .NET systems can typically lead to command execution through deserializing a [ViewState](../../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/dotNET.md#ViewState) crafted after obtaining the `machineKey` value from the `web.config` file.

In the case of Sitecore, Telerik can also be exploited if the keys are present within the `web.config` file. An attacker can also use this vulnerability to download Sitecore backups. Both the Telerik RCE and backup download attack vectors were disclosed in a [previous blog post](https://www.assetnote.io/resources/research/bypass-iis-authorisation-with-this-one-weird-trick-three-rces-and-two-auth-bypasses-in-sitecore-9-3). [](https://read.readwise.io/read/01jskf1x8j9fnvhq13j7n7z2t1)