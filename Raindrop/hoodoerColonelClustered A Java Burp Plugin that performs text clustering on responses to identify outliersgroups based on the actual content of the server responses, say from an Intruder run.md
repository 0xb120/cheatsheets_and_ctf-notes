---
raindrop_id: 1557121800
raindrop_highlights:
  69726e6b8b491ac5e5672cdb: 9a8c81fa90a1e60c2efea8d457e713a8
  69726e6fc106a2635bd8b1cc: a1793a7995a80bb7777fb1d574a0258e
  69726ea32e688b1ea40d6cc8: 000dea8378ed9f84d64e78d083bd5ae3
  69726eb0c3112f5f67172b47: 23472a6dad952da34762b23c38edb911
  69726ebec106a2635bd8cded: 94061aa96ac29e5a56a409bc5c287189
  69726ecf5d124f706664cc47: 09a316410af124c7355c54b134ddcf48
  69726ed39df944ba95aebdae: f311846d39b5ab52220e72b104320568
  69726edd8b491ac5e567524f: 43550a64a3a75c23a3739bf665749c5f
  69726eeb4d6ed8961865792f: e4a813e277445ab7d804a1b9fa16a74b
  69726eef918443e4507815db: 1c34c326ac2201fed4922440acfaea9a
  69726ef81c1f44584befdbd6: f07d9da245fd1f430e6b61bfc22931e1
  69726f12f019b5f3ae50b902: addce7f556963ec5d3c9868a5b5a8168
title: "hoodoer/ColonelClustered: A Java Burp Plugin that performs text clustering on responses to identify outliers/groups based on the actual content of the server responses, say from an Intruder run."

description: |-
  A Java Burp Plugin that performs text clustering on responses to identify outliers/groups based on the actual content of the server responses, say from an Intruder run.  - hoodoer/ColonelClustered:...

source: https://github.com/hoodoer/ColonelClustered

created: 2026-01-22
sync-date: 1769114384367
tags:
  - "_index"

 
  - "Tools"

---
# hoodoer/ColonelClustered: A Java Burp Plugin that performs text clustering on responses to identify outliers/groups based on the actual content of the server responses, say from an Intruder run.

![](https://opengraph.githubassets.com/43fd32da57e7398cd5b8c63a4bb08bc52c8779565ea3eb901a44abd9e13931ac/hoodoer/ColonelClustered)

> [!summary]
> A Java Burp Plugin that performs text clustering on responses to identify outliers/groups based on the actual content of the server responses, say from an Intruder run.  - hoodoer/ColonelClustered:...





A Burp Suite extension for clustering HTTP responses to find outliers.
A Java Burp Plugin that performs text clustering on responses to identify outliers/groups based on the actual content of the server responses, say from an Intruder run.
we look for those differences by looking at status codes, response time, and response sizes. All of these are indirect measures of the content of the response. Of course as pentesters we don't have time to read the content of thousands of server to look for differences in the response content. But certainly we can have algorithms do this for us.
That's where this plugin comes in. It uses text clustering techniques to analyze the content of the responses and put them into clusters based on their similarity.
It gives you another way to view the results of your intruder fuzzing attacks, and quickly identifying those server responses that are different.
How to Use
Load the Extension:
Send Responses for Analysis:
Go to any tool in Burp, such as Intruder results or Proxy history.
Select one or more request/response items.
Right-click and select "Send to Colonel Clustered".
A default Fast Scan will automatically begin, and a progress bar will appear to monitor the analysis.
Perform Deep Analysis (Optional):
If a more detailed, hierarchical clustering is desired, click the "Deep Analysis" button.
Note: If you are analyzing a large number of items, a warning will appear about potential performance issues before the scan begins.
A progress bar will appear, allowing you to monitor the analysis.
Analyze the Results in the Quad-Pane UI:

The "Colonel Clustered" tab uses a four-pane layout to help you quickly navigate results.
Top-Left (Clusters): This pane shows the clusters found, including a special "Outliers" group. Each entry shows the number of items in that cluster.
Bottom-Left (Cluster members): Select a cluster in the pane above to see all of its members displayed in this table. The table features several columns:
Request/Response Pair: The original index of the item.
Status Code: The HTTP response status code.
Length: The length of the response body in bytes.
Content-Type: The Content-Type header of the response.
Sorting: Click on any column header in the table to sort the items within that cluster, allowing you to easily find the largest/smallest responses, or group by status code.
Top-Right & Bottom-Right (Viewers): Select any row in the table to view its full request and response in the viewers on the right.