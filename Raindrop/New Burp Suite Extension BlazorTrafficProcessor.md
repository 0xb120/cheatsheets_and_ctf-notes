---
raindrop_id: 1348087134
raindrop_highlights:
  68cbb91c44e259183dcaeaa6: b0b797e349cdefbd142af2d798283604
  68cbb92810b79ee38aa8b279: 249b9051d734df1cbbf1ff1fcca050ec
  68cbba18608b24f31074af4f: a95789c3f35e8e117e3b7f029a039b50
  68cbba1f626887e93e6aac8e: 976e805bfb3c50cfe196647cd01a76eb
  68cbba31e82f05cf5c171d34: 25639d4ae3cbe549b42805ecfcffef97
  68cbba3a608b24f31074b4d0: 10082d5192bfe55abd87445db9f869a1
  68cbba4b979dc9a08387c8bf: e866086058af965cf5051cb740b14005
  68cbba5929170cd85d4103a8: c390a023bab510a85774fdecd66c6ef9
  68cbba5d29170cd85d410436: 147a3b1d1f3b15cab66cdafe9a9d01ca
  68cbba66e903562de7f9096e: fb849134930b0a9b0afa8257021c3780
  68cbba79ef709ba17ad0a119: b625a6a2b886aff98a6d762370b7aff6
  68cbba86979dc9a08387d260: adf3da6d5d3b5c0a434101c09517dfdd
  68cbbade626887e93e6ac9a3: b48c8c248c38ee25819fc0ac622e44fd
  68cbbaedef709ba17ad0b346: 18e423d537685db859208062f5deb2ed
  68cbbb03608b24f31074d27f: 653be1302f76b60f9600ef12a7ecdd9c
  68cbbb23471e198d606a5122: 1f562a605653dee255c40dd2f1ce4ff6
  68cbbb33e903562de7f92b81: 5bda4b98388be118ea82d8cd4d963c92
  68cbbbaf638a1a09288dd43b: 91fffdd9402f9105e92054e3d1b370f3
  68cbbbedb0cc210e2a9c0f96: a5606bfa4b71de82734aec48f3c3446e
  68cbbbef10b79ee38aa92c85: 9a7a969784e93e3a0cdd2f7757f283a8
  68cbbbf429170cd85d41475e: de02f6e6a861f1859c4e9686de89cfb6
  68cbbbfdef709ba17ad0df45: be6f008481c60ffb009398697d6bb7ba
  68cbbc06608b24f31074fe50: dc50c0e5dff346ffb2427f934b440a80
  68cbbc09e82f05cf5c17693e: 9ae71b17827ecaffdae8faad33787fbb
  68cbbc0b29170cd85d414aeb: dda2f64e9b8c588a7a277b85423cde01
  68cbbc0d10b79ee38aa930fd: 811a39837402a6e20cc25d85aa3e6ea8
  68cbbcb19f97e78b18fa28e9: bf9f87e5dec65b5b48954873708fb011
  68cbbcbd9f97e78b18fa2b46: 8805c643f1dbbc5db2c7331a5e72e1b1
  68cbbcc1608b24f310751ca6: 429bc88a8e260442cb91f0a46cffb7fe
  68cbbcca608b24f310751e1e: fc912065bbe9ee91c6b629073bddda15
  68cbbcd3471e198d606a96b2: a0cbf11ddd72d7b16b079e188f39d7e2
  68cbbce6ef709ba17ad103d5: b1c4664505ed638743a6d637b9aa5edc
  68cbbd69638a1a09288e1b0a: c5b832813f1269a03febd7883d514ac2
  68cbbd6b638a1a09288e1b93: 379bce5f00f834b90d1acb5a64253717
  68cbbd787db6aa2fb315a8d0: 34293dbade16eac7aa510021ec95f1ff
  68cbbd7a626887e93e6b396d: 5367f5060483b432105e935919132e4b
  68cbbd94471e198d606ab436: 47590709ccaf6ad3ecfc8ed5dca2f694
  68cbbda7abb21f4b009c1830: 716c9b32a5169de63011eac73703e53b
  68cbbdb8979dc9a0838856b2: 248ce20766047456d71ff479bf691f15
  68cbbdc37db6aa2fb315b4a6: 576ddc8c51103b81482f1124f168bdc3
  68cbbdd210b79ee38aa97933: 103b5c345e0a8a34da26faadc2c1fefc
title: "New Burp Suite Extension: BlazorTrafficProcessor"

description: |-
  Pentesting web applications that use Blazor server comes with unique challenges, especially without tooling. In this post, we discuss why such challenges exist and provide a Burp Suite Extension to address them.

source: https://www.aon.com/en/insights/cyber-labs/new-burp-suite-extension---blazortrafficprocessor

created: 1758181627873
type: article
tags:
  - "_index"

 
  - "tech-blog" 
  - "blazor"

---
# New Burp Suite Extension: BlazorTrafficProcessor

![](https://res.aon.com/image/fetch/c_fill,f_auto,g_auto/https://assets.aon.com//-/media/images/photos/infrastructure/infrastructure-loop-roundabout-data-822649514.jpg)

> [!summary]
> Pentesting web applications that use Blazor server comes with unique challenges, especially without tooling. In this post, we discuss why such challenges exist and provide a Burp Suite Extension to address them.





During a web application assessment, we encountered ASP.NET’s “Blazor” server for the first time.
For starters, all the messages transmitted by the application included seemingly random binary characters. You can make out some strings within these messages, but most of the data is not easily readable
Additionally, any attempts we made to tamper with or replay requests resulted in an error and the connection being renegotiated
Blazor Basics
Blazor is a framework that comes with a variety of hosting options: WebAssembly (WASM), Server-Side ASP.NET, and Native Client. For the purposes of this blog post, we’ll focus on server-side Blazor, which integrates with SignalR to send browser events and receive pre-rendered page updates.
> [!info]
> By default, Blazor server applications communicate via WebSockets, though other transports such as Long Polling over HTTP are available as well.

Forcing a Blazor application to use Long Polling is possible within the negotiation process.
POST /_blazor/negotiate?negotiateVersion=1 HTTP/1.1
Content-Length: 0
[...]
HTTP/1.1 200 OK
Content-Length: 316
Content-Type: application/json
[...]

{"negotiateVersion":1,"connectionId":"******","connectionToken"
:"******","availableTransports":[{"transport":"WebSockets","transferFormats":["Text","Binary"]},{"transport":"ServerSentEvents","transferFormats":["Text"]},{"transport":"LongPolling","transferFormats":["Text","Binary"]}]}
Using a Burp match and replace rule, you can remove the WebSockets transport from the response, forcing the browser to fall back to use Long Polling over HTTP.
By forcing the application to use Long Polling, all Blazor traffic will now occur over HTTP which makes Blazor data more accessible to prospective Burp Suite extensions.
If we turn to the documentation, Microsoft identifies this format as MessagePack and outlines how it can be used in ASP.NET applications.
MessagePack
MessagePack is another serialization format used to package structured data, like JSON, XML, etc. The key difference with MessagePack is that it is binary in nature, meaning specific bytes are used to indicate the types and length of serialized data.

While Blazor server uses MessagePack, the traffic is specifically formatted according to Blazor’s own Hub Protocol specification.
Blazor messages are formatted according to the Hub Protocol specification.

([Length] [Body])([Length] [Body])...

Length is a variable-size integer representing the size of Body, which is the actual message.
If you modify a string to be a different length than the original message, you’d need to update the Length variable-size integer as well.
For the Body field, there are different types of messages supported by Blazor (i.e., Invocation, StreamInvocation, Ping, etc.). However, while proxying traffic and testing a sample Blazor application, we rarely saw any other types of messages being used other than Invocation.
InvocationMessage Analysis

The predominant message type observed while testing Blazor applications is an InvocationMessage, used to render page updates and submit data.
Taking a look at the specification, we see that there is the following structure:
[1, Headers, InvocationId, Target, [Arguments], [StreamIds]]
1 – the message type, InvocationMessage types are 1.
Headers – a map containing string key-value pairs. During testing of a sample Blazor app, this was observed to only be equal to a null/empty map.
InvocationId – this can either be NIL to indicate a lack of an invocation ID or a string that holds the value. Again, this was always NIL during testing.
Target – a string representing the backend function to call.
Arguments – an array of arguments to pass to that backend function.
StreamIds – an array of strings representing unique stream identifiers. Again, this was always NIL or non-existent in the messages observed whilst testing.
Tampering
Within Blazor server applications, the JSInvokable attribute allows developers to expose DotNet functions to the client-side JavaScript.
[JSInvokable("CallMe")]
public static void hiddenFunc(String var)
{
    Console.WriteLine("Hidden function called!");
    Console.WriteLine(var);
}
This is a simple example that just logs some user input to the console, but there are no web pages that allow the user to call this function.
Instead, the DotNet.invokeMethodAsync JavaScript function can be used (as outlined here).
Figure 8 – Deserialized Invocation Request
The input of foo is contained within a 1-element array, so let’s try tampering and replaying the request. For demo purposes, we’ll change the JSON payload in the BTP tab to include a newline to see if newlines get written to the console output:
[{"Target":"BeginInvokeDotNetFromJS","Headers":0,"Arguments":["3","BlazorServerTestApp","CallMe",0,["Line1\r\nLine2"]],"MessageType":1}]
Upon sending the request, the response is a simple 200 OK:
HTTP/1.1 200 OK
Content-Length: 0
Connection: close
Content-Type: text/plain
Server: Kestrel
> [!danger]
> Important note about Blazor server Long Polling: sending any type of invocation POST request will not return data in the response. Instead, Long Polling keeps an open GET request that receives server updates as they become available. As such, if you send a request with malformed input that causes an error, you won’t see that error unless you look at subsequent GET requests.

What are these “Target” values I keep seeing?
While testing Blazor server applications, you’re likely to run into various Target values such as: BeginInvokeDotNetFromJS, OnRenderCompleted, OnLocationChanged, etc. These functions themselves are not specific to the app that you’re testing. Rather, they are built-in to ASP.NET Core as part of ComponentHub and are used to facilitate communications between the frontend JavaScript and the backend .NET application.
The implementation of these functions can be found in the “aspnetcore” repository on GitHub here.
It is important to distinguish between what’s native and what’s custom in Blazor server applications, since your goal will likely be to find vulnerabilities in the app that you’re testing, not in Blazor itself. As such, focus your testing efforts on fields that contain your input (i.e., arguments to BeginInvokeDotNetFromJS) as opposed to normal Blazor operations (i.e., OnRenderCompleted or EndInvokeJSFromDotNet).