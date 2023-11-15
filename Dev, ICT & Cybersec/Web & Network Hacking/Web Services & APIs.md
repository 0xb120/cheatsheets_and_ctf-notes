# Web Service & API 101

The terms `web service` and `application programming interface (API)` should not be used interchangeably:
- Web Service are a subset of APIs
- Web services usually utilize **SOAP** for security reasons. APIs can be found using different designs, such as XML-RPC, JSON-RPC, SOAP, and REST.
- Web services usually utilize the XML format for data encoding. APIs can be found using different formats to store data, but mostly JSON

## XML-RPC

>[!summary]- Documentation
>XML-RPC specification: http://xmlrpc.com/spec.md

XML-RPC uses XML for encoding/decoding the remote procedure call (RPC) and the respective parameter(s). HTTP is usually the transport of choice.
The payload in XML is essentially a single `<methodCall>` structure. `<methodCall>` should contain a `<methodName>` sub-item, that is related to the method to be called. If the call requires parameters, then `<methodCall>` must contain a `<params>` sub-item.

```http
POST /RPC2 HTTP/1.0
User-Agent: Frontier/5.1.2 (WinNT)
Host: betty.userland.com
Content-Type: text/xml
Content-length: 181

<?xml version="1.0"?>
<methodCall>
<methodName>examples.getStateName</methodName>
<params>
   <param>
		 <value><i4>41</i4></value>
		 </param>
	  </params>
</methodCall>

--- RESPONSE ---

```http
HTTP/1.1 200 OK
Connection: close
Content-Length: 158
Content-Type: text/xml
Date: Fri, 17 Jul 1998 19:55:08 GMT
Server: UserLand Frontier/5.1.2-WinNT

<?xml version="1.0"?>
<methodResponse>
 <params>
	<param>
		  <value><string>South Dakota</string></value>
		  </param>
	</params>
</methodResponse>
```

## JSON-RPC

>[!summary]- Documentation
>JSON-RPC specification: https://www.jsonrpc.org/specification

JSON-RPC uses JSON to invoke functionality. HTTP is usually the transport of choice. It works pretty much like XML-RPC, but supports also an `id` parameter, sent by the client, whose parameter returned by the server must match.

```http
POST /ENDPOINT HTTP/1.1
Host: ...
Content-Type: application/json-rpc
Content-Length: ...

{"method": "sum", "params": {"a":3, "b":4}, "id":0}

--- RESPONSE ---

HTTP/1.1 200 OK
...
Content-Type: application/json-rpc

{"result": 7, "error": null, "id": 0}
```

## SOAP (Simple Object Access Protocol)

SOAP uses XML like XML-RPC but provides more functionalities. It defines both a **header structure** and a **payload structure**. The former identifies the actions that SOAP nodes are expected to take on the message, while the latter deals with the carried information. A Web Service Definition Language (WSDL) declaration can also be included, but it's optional.

- Anatomy of a SOAP Message
    - `soap:Envelope`: (Required block) Tag to differentiate SOAP from normal XML. This tag requires a `namespace` attribute.
    - `soap:Header`: (Optional block) Enables SOAP’s extensibility through SOAP modules.
    - `soap:Body`: (Required block) Contains the procedure, parameters, and data.
    - `soap:Fault`: (Optional block) Used within `soap:Body` for error messages upon a failed API call.

```http
POST /Quotation HTTP/1.0
Host: www.xyz.org
Content-Type: text/xml; charset = utf-8
Content-Length: nnn

<?xml version = "1.0"?>
<SOAP-ENV:Envelope
xmlns:SOAP-ENV = "http://www.w3.org/2001/12/soap-envelope"
 SOAP-ENV:encodingStyle = "http://www.w3.org/2001/12/soap-encoding">

<SOAP-ENV:Body xmlns:m = "http://www.xyz.org/quotations">
   <m:GetQuotation>
	 <m:QuotationsName>MiscroSoft</m:QuotationsName>
  </m:GetQuotation>
</SOAP-ENV:Body>
</SOAP-ENV:Envelope>

--- RESPONSE ---

HTTP/1.0 200 OK
Content-Type: text/xml; charset = utf-8
Content-Length: nnn

<?xml version = "1.0"?>
<SOAP-ENV:Envelope
xmlns:SOAP-ENV = "http://www.w3.org/2001/12/soap-envelope"
SOAP-ENV:encodingStyle = "http://www.w3.org/2001/12/soap-encoding">

<SOAP-ENV:Body xmlns:m = "http://www.xyz.org/quotation">
  <m:GetQuotationResponse>
	 <m:Quotation>Here is the quotation</m:Quotation>
 </m:GetQuotationResponse>
</SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

## RESTful (Representational State Transfer)

REST web services usually use XML or JSON. WSDL declarations are supported but uncommon. HTTP is the transport of choice, and HTTP verbs are used to access/change/delete resources and use data.

XML:
```http
POST /api/2.2/auth/signin HTTP/1.1
HOST: my-server
Content-Type:text/xml

<tsRequest>
<credentials name="administrator" password="passw0rd">
  <site contentUrl="" />
</credentials>
</tsRequest>
```

JSON:
```http
POST /api/2.2/auth/signin HTTP/1.1
HOST: my-server
Content-Type:application/json
Accept:application/json

{
"credentials": {
 "name": "administrator",
"password": "passw0rd",
"site": {
  "contentUrl": ""
 }
}
}
```

## WSDL

WSDL stands for Web Service Description Language. WSDL is an XML-based file exposed by web services that informs clients of the provided services/methods, including where they reside and the method-calling convention.

Usually they can be found at `/wsdl` but arbitrary location (or per-service definitions) can also be applied:
- `/wsdl`
- `/wsdl?wsdl`
- `/api/wsdl`
- `/wsdl?FUZZ`
- `/example.wsdl`
- `/?disco` or `/example.disco` [^disco]

[^disco]: [DISCO](https://learn.microsoft.com/en-us/archive/msdn-magazine/2002/february/xml-files-publishing-and-discovering-web-services-with-disco-and-uddi) by Microsoft

### WSDL file breakdown

- `Definition`: The root element of all WSDL files. Inside the definition, the name of the web service is specified, all namespaces used across the WSDL document are declared, and all other service elements are defined.
```xml
<wsdl:definitions targetNamespace="http://tempuri.org/" 
    <wsdl:types></wsdl:types>
    <wsdl:message name="LoginSoapIn"></wsdl:message>
    <wsdl:portType name="HacktheBoxSoapPort">
  	  <wsdl:operation name="Login"></wsdl:operation>
    </wsdl:portType>
    <wsdl:binding name="HacktheboxServiceSoapBinding" type="tns:HacktheBoxSoapPort">
  	  <wsdl:operation name="Login">
  		  <soap:operation soapAction="Login" style="document"/>
  		  <wsdl:input></wsdl:input>
  		  <wsdl:output></wsdl:output>
  	  </wsdl:operation>
    </wsdl:binding>
    <wsdl:service name="HacktheboxService"></wsdl:service>
</wsdl:definitions>
```
- `Data Types`: The data types to be used in the exchanged messages.
```xml
<wsdl:types>
    <s:schema elementFormDefault="qualified" targetNamespace="http://tempuri.org/">
  	  <s:element name="LoginRequest">
  		  <s:complexType>
  			  <s:sequence>
  				  <s:element minOccurs="1" maxOccurs="1" name="username" type="s:string"/>
  				  <s:element minOccurs="1" maxOccurs="1" name="password" type="s:string"/>
  			  </s:sequence>
  		  </s:complexType>
  	  </s:element>
  	  <s:element name="LoginResponse">
  		  <s:complexType>
  			  <s:sequence>
  				  <s:element minOccurs="1" maxOccurs="unbounded" name="result" type="s:string"/>
  			  </s:sequence>
  		  </s:complexType>
  	  </s:element>
  	  <s:element name="ExecuteCommandRequest">
  		  <s:complexType>
  			  <s:sequence>
  				  <s:element minOccurs="1" maxOccurs="1" name="cmd" type="s:string"/>
  			  </s:sequence>
  		  </s:complexType>
  	  </s:element>
  	  <s:element name="ExecuteCommandResponse">
  		  <s:complexType>
  			  <s:sequence>
  				  <s:element minOccurs="1" maxOccurs="unbounded" name="result" type="s:string"/>
  			  </s:sequence>
  		  </s:complexType>
  	  </s:element>
    </s:schema>
</wsdl:types>
```
- `Messages`: Defines input and output operations that the web service supports. In other words, through the _messages_ element, the messages to be exchanged, are defined and presented either as an entire document or as arguments to be mapped to a method invocation.
```xml
<!-- Login Messages -->
<wsdl:message name="LoginSoapIn">
    <wsdl:part name="parameters" element="tns:LoginRequest"/>
</wsdl:message>
<wsdl:message name="LoginSoapOut">
    <wsdl:part name="parameters" element="tns:LoginResponse"/>
</wsdl:message>
<!-- ExecuteCommand Messages -->
<wsdl:message name="ExecuteCommandSoapIn">
    <wsdl:part name="parameters" element="tns:ExecuteCommandRequest"/>
</wsdl:message>
<wsdl:message name="ExecuteCommandSoapOut">
    <wsdl:part name="parameters" element="tns:ExecuteCommandResponse"/>
</wsdl:message>
```
- `Operation`: Defines the available SOAP actions alongside the encoding of each message.
- `Port Type`: Encapsulates every possible input and output message into an operation. More specifically, it defines the web service, the available operations and the exchanged messages. Please note that in WSDL version 2.0, the _interface_ element is tasked with defining the available operations and when it comes to messages the (data) types element handles defining them.
```xml
<wsdl:portType name="HacktheBoxSoapPort">
    <!-- Login Operaion | PORT -->
    <wsdl:operation name="Login">
  	  <wsdl:input message="tns:LoginSoapIn"/>
  	  <wsdl:output message="tns:LoginSoapOut"/>
    </wsdl:operation>
    <!-- ExecuteCommand Operation | PORT -->
    <wsdl:operation name="ExecuteCommand">
  	  <wsdl:input message="tns:ExecuteCommandSoapIn"/>
  	  <wsdl:output message="tns:ExecuteCommandSoapOut"/>
    </wsdl:operation>
</wsdl:portType>
```
- `Binding`: Binds the operation to a particular port type. Think of bindings as interfaces.
```xml
<wsdl:binding name="HacktheboxServiceSoapBinding" type="tns:HacktheBoxSoapPort">
    <soap:binding transport="http://schemas.xmlsoap.org/soap/http"/>
    <!-- SOAP Login Action -->
    <wsdl:operation name="Login">
  	  <soap:operation soapAction="Login" style="document"/>
  	  <wsdl:input>
  		  <soap:body use="literal"/>
  	  </wsdl:input>
  	  <wsdl:output>
  		  <soap:body use="literal"/>
  	  </wsdl:output>
    </wsdl:operation>
    <!-- SOAP ExecuteCommand Action -->
    <wsdl:operation name="ExecuteCommand">
  	  <soap:operation soapAction="ExecuteCommand" style="document"/>
  	  <wsdl:input>
  		  <soap:body use="literal"/>
  	  </wsdl:input>
  	  <wsdl:output>
  		  <soap:body use="literal"/>
  	  </wsdl:output>
    </wsdl:operation>
</wsdl:binding>
```
- `Service`: A client makes a call to the web service through the name of the service specified in the service tag. Through this element, the client identifies the location of the web service.
```xml
<wsdl:service name="HacktheboxService">
  <wsdl:port name="HacktheboxServiceSoapPort" binding="tns:HacktheboxServiceSoapBinding">
	<soap:address location="http://localhost:80/wsdl"/>
  </wsdl:port>
</wsdl:service>
```