---
raindrop_id: 1521221751
raindrop_highlights:
  6963f37ad50cb47fc41abcdd: 10b4d61ad062da4938e31952e146d1ad
  6963f390ce769efdaa426875: fae609183811a7078bcae26ef0ce54a0
  6963f3a57d028986911f1ccf: b0cb15284524cd6838f241885a06f286
  6963f3de0c216ba50d6f390e: 46f4ac483cd8a21e5be1f023649ac5f0
  6963f404d399b53b992d7d3b: fe81afd5a8947f8893095c48ce70288c
  6963f40ad815af5e327f15f7: cc343898e6ac00f4679865647ce45728
  6963f42c4214af98e9d682ae: 8100252b3d17ecb934269555c2d76071
  6963f477ef1ec5dd4c93e56f: c54ee4baf9f7f73ddf46ffe882509a3e
  6963f482ef1ec5dd4c93e753: 1ecd856fc3118f60f9c08534af6dbd89
  6963f487ef1ec5dd4c93e833: 74a68eb2e3ad9b3c4b4b476d13fba7b0
  6963f4a086ce7738d8e2d419: 66eb280596b2707f693bf3c14d361e3a
title: "Turning List-Unsubscribe into an SSRF/XSS Gadget"

description: |-
  The List-Unsubscribe SMTP header is standardized but often overlooked during security assessments. It allows email clients to provide an easy way for end-users to unsubscribe from mailing lists.
  This post discusses how this header can be abused to perform Cross-Site Scripting (XSS) and Server-Side Request Forgery (SSRF) attacks in certain scenarios. Real-world examples involving Horde Webmail (CVE-2025-68673) and Nextcloud Mail App are provided to illustrate the risks.

source: https://security.lauritz-holtmann.de/post/xss-ssrf-list-unsubscribe/

created: 2026-01-02
sync-date: 1769114383421
tags:
  - "_index"

 
  - "tech-blog"

---
# Turning List-Unsubscribe into an SSRF/XSS Gadget

![](https://security.lauritz-holtmann.de/images/advisories/nextcloud-list-unsubscribe-1.png)

> [!summary]
> The List-Unsubscribe SMTP header is standardized but often overlooked during security assessments. It allows email clients to provide an easy way for end-users to unsubscribe from mailing lists.
This post discusses how this header can be abused to perform Cross-Site Scripting (XSS) and Server-Side Request Forgery (SSRF) attacks in certain scenarios. Real-world examples involving Horde Webmail (CVE-2025-68673) and Nextcloud Mail App are provided to illustrate the risks.





The List-Unsubscribe SMTP header is standardized but often overlooked during security assessments. It allows email clients to provide an easy way for end-users to unsubscribe from mailing lists.

This post discusses how this header can be abused to perform Cross-Site Scripting (XSS) and Server-Side Request Forgery (SSRF) attacks in certain scenarios. Real-world examples involving Horde Webmail (CVE-2025-68673) and Nextcloud Mail App are provided to illustrate the risks.
Foundations

The List-Unsubscribe SMTP header is defined in RFC 23691 and allows email clients to provide an easy way for end-users to unsubscribe from mailing lists.
The following examples were taken from RFC 2369, section 3.2:

The List-Unsubscribe field describes the command (preferably using mail) to directly unsubscribe the user (removing them from the list).

Examples:

  List-Unsubscribe: <mailto:list@host.com?subject=unsubscribe>
  List-Unsubscribe: (Use this command to get off the list)
    <mailto:list-manager@host.com?body=unsubscribe%20list>
  List-Unsubscribe: <mailto:list-off@host.com>
  List-Unsubscribe: <http://www.host.com/list.cgi?cmd=unsub&lst=list>,
    <mailto:list-request@host.com?subject=unsubscribe>
Many modern email clients and webmail applications have implemented support for this header to improve user experience. For example, when an email includes a List-Unsubscribe header, the client may render a button or link that allows the user to unsubscribe with a single click.
Easy to miss, but the most interesting example was the very last one, which includes both an HTTP URI and a mailto link. Can we simply add arbitrary http(s):// URIs here? What about other schemes?
Stored XSS via JavaScript URI: Horde Webmail (CVE-2025-68673)
When an email includes a List-Unsubscribe SMTP header, a button is rendered within the message detail view that allows users to unsubscribe directly from mailing lists.

A malicious actor can exploit this behavior by including a JavaScript URI (javascript:), which allows them to execute JavaScript in the origin of the Horde installation when an end-user clicks the link:

<table class="horde-table mailinglistinfo">
 <tbody>
  <tr>
   <td>Unsubscribe</td>
   <td><a href="javascript://lhq.at/%0aconfirm(document.domain)" target="_blank">javascript://lhq.at/%0aconfirm(document.domain)</a></td>
  </tr>
 </tbody>
</table>
Blind SSRF: Nextcloud Mail App
Nextcloud’s Mail app supports the List-Unsubscribe SMTP header to unsubscribe from mailing lists
When the end-user unsubscribes, the Nextcloud instance issues a server-side request
During my research, it looked like Nextcloud would allow forging SSRF requests via the List-Unsubscribe header to arbitrary internal destinations. However, this seems to be possible only if the development configuration flag 'allow_local_remote_servers' => true is set or other supporting factors are present that allow exploitation.