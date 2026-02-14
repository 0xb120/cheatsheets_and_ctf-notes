---
title: GFI MailEssentials - Yet Another .NET Target
source: https://frycos.github.io/vulns4free/2025/04/28/mailessentials.html?__readwiseLocation=
author:
  - Frycos Security Diary
published: 2025-04-28
created: 2026-02-14
description: What is this product GFI MailEssentials all about? We’re living the future, right? So let’s ask the GFI AI.
tags:
  - clippings/articles
  - dotnet
  - XXE-Injection
---
# GFI MailEssentials - Yet Another .NET Target

> [!summary]+
> > This blog post details a security research journey into GFI MailEssentials, a mail content processing product.
> The author sets up a trial version with IIS SMTP service and performs technology stack enumeration.
>
> Three vulnerabilities are disclosed:
> > **Vulnerability I - Built-in Local Privilege Escalation (LPE):** Discovered via .NET Remoting using `BinaryFormatter` deserialization. An unprivileged user could achieve SYSTEM privileges.
> > **Vulnerability II - Remote Code Execution (RCE) via Deserialization:** Found in the `InstallClientCertificate` method of the `MultiNodeConfigurationService` WCF endpoint. An authenticated user (potentially any user in AD environments) could trigger `BinaryFormatter` deserialization to achieve RCE on the master server. This was demonstrated by modifying client-side code to send a malicious gadget.
> > **Vulnerability III - XML External Entity (XXE):** Located in the `ImportAntiPhisingKeywordList` method, accessible via the web UI's phishing keywords import function. It allowed reading local files (e.g., `win.ini`) from the server.
>
> The post concludes that Vulnerability II is the most critical due to its authenticated RCE potential.
> GFI released version 21.8 with patches: .NET Remoting LPE received a `SerializationBinder`, the dangerous `BinaryFormatter` usage in the remote web call was removed, and XXE sinks had their `XmlResolver` set to null.

The setup process turned out to be quite easy. One can simply get a [free trial](https://gfi.ai/products-and-solutions/network-security-solutions/mailessentials/free-trial) version on GFI’s company website after providing some data for registration.

Required components:
- Microsoft IIS SMTP service or Microsoft Exchange Server 2010/2013/2016

So how to install Microsoft’s IIS SMTP service instead. After some googling I found this snippet for PowerShell:

```powershell
Import-Module Servermanager
Add-WindowsFeature SMTP-Server
```

# Technology Stack Enumeration

Starting the `inetmgr` [^1], I observed two new application pools, `MailEssentials` and `MailEssentials_Services`.

A right click on one of the applications gives a context menu with an “Explore” entry, automatically opening the webroot of the targeted application in your file browser.

I always like to browse through the related directories to learn about the technology stack first. I.e. collecting all kind of file types, look into configuration files etc. Some facts I found are shown next, mainly read from `C:\Program Files (x86)\GFI\MailEssentials\wwwconf\web.config`:

- Learn about predefined application setting values in `<appSettings>`

- Find HTTP Modules in `<httpModules>` sections; these are usually triggered for every request (a nice pre-auth attack surface opportunity)

- Look for HTTP Handlers in `<httpHandlers>` sections, defining e.g. which file extension in an URI path might be handled by which .NET class

- In general, `<add assembly="[PLACEHOLDER]" />` is obviously a good indicator which .NET Assemblies are in use

- Authorization indicators are found in `<authorization>` sections

- `<system.serviceModel>` section often hold information about web services, their endpoints and their implementing classes

Of course, this is an incomplete list and also special to this specific product, but Microsoft provides excellent online documentation for every single attribute.

I spotted a `web.config` section of special interest almost immediately: `<system.runtime.remoting>`. 

This seems to match with the namespace of the famous [.NET Remoting](../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/dotNET.md#.NET%20Remoting) technology `System.Runtime.Remoting`. And indeed, I found several examples on older blogs about people using `<system.runtime.remoting>` sections to define well-known types with its implementing class(es) referenced.

```xml
<system.runtime.remoting>
  <application name="MEComplete">
    <client url="tcp://localhost:8013">
      <wellknown type="MEC.Reporting.Base.IReporting, MEC.Reporting.Base" url="tcp://localhost:9093/Report" />
      <wellknown type="MEC.ConfigurationServices.ConfigurationService, MEC.ConfigurationServices" url="tcp://localhost:9091/ConfigurationServices" />
      <wellknown type="MEC.Configuration.Base.IRemotingHelper, MEC.Configuration.Base" url="tcp://localhost:9091/RemotingHelper" />
      <wellknown type="MEC.Configuration.Base.IPatchChecker, MEC.Configuration.Base" url="tcp://localhost:9091/PatchChecker" />
      <wellknown type="MEC.Configuration.Base.RemotingHelper, MEC.Configuration" url="tcp://localhost:9091/RemotingHelper" />
      <wellknown type="MEC.RemoteMonitor.RemotingMonitor, MEC.RemoteMonitor" url="tcp://localhost:9091/ContentSecurity/RemoteMonitor" />
      <wellknown type="ContentSecurity.ML.QSS.QuarantineStoreServices, ContentSecurity.ML.QSS" url="tcp://localhost:9093/ContentSecurity/QuarantineServices" />
      <wellknown type="MEC.RSSRemotePlugin.RSSRemotePlugin, MEC.RSSRemotePlugin" url="tcp://localhost:9093/ContentSecurity/RemoteRSS" />
      <wellknown type="ContentSecurity.ML.Quar.IQuar, ContentSecurity.ML.Quar.Base" url="tcp://localhost:9093/ContentSecurity/Quar" />
      <wellknown type="ContentSecurity.ML.QSS.IQuarantineStore, ContentSecurity.ML.QSS.Base" url="tcp://localhost:9093/ContentSecurity/QuarantineServices" />
      <wellknown type="MEC.ConfigurationServices.IRuleDb, MEC.ConfigurationServices.Base" url="tcp://localhost:9091/ConfigurationServices" />
      <wellknown type="ContentSecurity.ML.AST.WdPFolders.PublicFolderTraining, ContentSecurity.ML.AST.WdPFolders" url="tcp://localhost:9091/ContentSecurity/PublicFolderTraining" />
      <wellknown type="ContentSecurity.ML.AST.EWSPFolders.PublicFolderTraining, ContentSecurity.ML.AST.EWSPFolders" url="tcp://localhost:9091/ContentSecurity/PublicFolderTrainingEWS" />
    </client>
  </application>
</system.runtime.remoting>
```

**Remoting** sounds pretty much like “remote”, but you might be surprised how often one finds inter-process communication on the same machine with this technology even nowadays. We all, hopefully, know that .NET Remoting is a gift for vulnerability researchers. 

I highly recommend you to read about a [series](https://code-white.com/blog/2022-01-dotnet-remoting-revisited/) [of](https://code-white.com/blog/leaking-objrefs-to-exploit-http-dotnet-remoting/) [blogs](https://code-white.com/blog/teaching-the-old-net-remoting-new-exploitation-tricks/) of my colleague @mwulftange. He basically destroyed the last hopes on making .NET Remoting secure and even Microsoft marked it as an obsolete and dangerous technique these days.

# Vulnerability I - Built-in Local Privilege Escalation

From the referenced blogs above, you learnt about different channel sink variants (`HTTPServerChannel`, `IPCServerChannel`, `TCPServerChannel`) and its different constellations with .NET formatters. 

Obviously, we see that the well-known services are based on the TCP variant and its most common serializer `System.Runtime.Serialization.Formatters.Binary.BinaryFormatter`. 

So without diving into the code, just by looking at the configuration files, we might be brave enough to simply use the existing .NET Remoting knowledge and tooling for a first quick finding: a local privilege escalation.

1. All we need is a low-priv user, we call him `nonadmin`. 
2. Then a malicious serialized object, later being deserialized by our friend `BinaryFormatter`, has to be created with [ysoserial .NET](https://github.com/pwntester/ysoserial.net) [^2]. 
3. Deliver this payload then with James Forshaw’s tooling [ExploitRemotingService](https://github.com/tyranid/ExploitRemotingService).
4. SYSTEM is ours!

# Vulnerability II - More Deserialization Please

Maybe we find another use of dangerous deserialization issues. But before diving into this kind of variant analysis, further enumeration is needed. We don’t want to miss attack surface, do we? 

Back to the `inetmgr`, two more applications were shown: `MailEssentials_Services` and `MailEssentialsRSS`.

Browse the target, otherwise your IIS worker processes `w3wp.exe` won’t show up in the process list and therefore not in your favourite .NET decompiler tool (which in my case is still [dnSpy](https://github.com/dnSpy/dnSpy) most of the time).

Also keep in mind, that you might find 32-bit and/or 64-bit optimized .NET processes running. This is why I always open both dnSpy versions in parallel to make sure that I don’t miss any related processes.

Since we see a lot of GFI MailEssentials processes, these processes probably have to communicate with each other a lot. We’ve already seen one potential technology being used for this: .NET Remoting using `BinaryFormatter` de/serialization.

Now, I assumed that this formatter might be used in other ways, too. So in [dnSpy](../Dev,%20ICT%20&%20Cybersec/Tools/dnSpy.md) we call for decompilation of `System.Runtime.Serialization.Formatters.Binary.BinaryFormatter`. We press `Crtl`+`Shift`+`R` to call the “Analyze” function, searching for all uses of this class.

![BF Users](https://frycos.github.io/assets/images/mailessentials/gfiblog_bfusages.png)

This returns a large amount of GFI MailEssentials classes but also other 3rd party libraries. 

Scrolling through the findings finally led me to an interesting `System.Runtime.Serialization.Formatters.Binary.BinaryFormatter::Deserialize(System.IO.Stream)` call at `MailEssentialsClientService.MasterClass/MultiNodeService::InstallClientCertificate(System.IO.Stream)`.

```cs
public static void InstallClientCertificate(Stream certificateStream)
{
	MasterClass.MultiNodeService.Log.Info("InstallClientCertificate() >>", new object[0]);
	try
	{
		IRemotingHelper @object = MasterClass.MultiNodeService._activator.GetObject<IRemotingHelper>();
		MemoryStream memoryStream = new MemoryStream();
		byte[] array = new byte[10000];
		int num;
		do
		{
			num = certificateStream.Read(array, 0, array.Length); // [1]
			memoryStream.Write(array, 0, num); // [2]
		}
		while (num > 0);
		memoryStream.Position = 0L;
		if (memoryStream != null)
		{
			using (memoryStream)
			{
				BinaryFormatter binaryFormatter = new BinaryFormatter();
				memoryStream.Seek(0L, SeekOrigin.Begin);
				X509Certificate2 x509Certificate = (X509Certificate2)binaryFormatter.Deserialize(memoryStream); // [3]
				@object.CreateCertificate(x509Certificate, "GFIMailEssentials", X509FindType.FindBySubjectName, StoreName.TrustedPeople, StoreLocation.LocalMachine, false);
				if (Environment.OSVersion.Version.Major == 5 && Environment.OSVersion.Version.Minor == 2)
				{
					@object.CreateCertificate(x509Certificate, "GFIMailEssentials", X509FindType.FindBySubjectName, StoreName.Root, StoreLocation.LocalMachine, false);
				}
			}
		}
	}
  // ...
```

The method takes a `System.IO.Stream` parameter `certificateStream` which is read into a byte array \[1\] and then written into a `System.IO.MemoryStream` \[2\]. The content gets deserialized by calling `System.Runtime.Serialization.Formatters.Binary.BinaryFormatter::Deserialize(System.IO.Stream)` \[3\] without any further protections/defenses. Where is this `certificateStream` coming from? Can we control it?

By using the dnSpy Analyzer, we follow the method’s call back to `MailEssentialsClientService.MultiNodeConfigurationService::InstallClientCertificate(System.IO.Stream)`.

`MailEssentialsClientService.MultiNodeConfigurationService` is annotated with the attribute [`System.ServiceModel.ServiceBehaviorAttribute`](https://learn.microsoft.com/en-us/dotnet/api/system.servicemodel.servicebehaviorattribute?view=netframework-4.8.1). Looking at the interface `MEC.Configuration.Base.IMultiNodeConfigurationService`, the attribute use of [`System.ServiceModel.ServiceContractAttribute`](https://learn.microsoft.com/en-us/dotnet/api/system.servicemodel.servicecontractattribute?view=netframework-4.8.1) is revealed.

Indicates that an interface or a class defines a service contract in a Windows Communication Foundation (WCF) application.

Let’s search for web service artifacts again on the file system, e.g. for `.svc` file extensions. We find it at `C:\Program Files (x86)\GFI\MailEssentials\wwwservices\MultiNodeConfigurationService.svc`.

![MultiNodeConfigurationService](https://frycos.github.io/assets/images/mailessentials/gfiblog_multinodeconfservice.png)

```xml
<%@ ServiceHost Language="C#" Debug="true" Service="MailEssentialsClientService.MultiNodeConfigurationService" CodeBehind="MultiNodeConfigurationService.svc.cs" %>
```

Browsing to `/MailEssentials_Services/MultiNodeConfigurationService.svc` shows us the standard service page with examples for client stubs etc., and of course also the link to the WSDL URI.

Using the good old [Wsdler](../Dev,%20ICT%20&%20Cybersec/Tools/Burpsuite.md#Wsdler) is a convenient way to build request templates for each remote method call.

And here we are, with a template for our `InstallClientCertificate` method call.

```http
POST /MailEssentials_Services/MultiNodeConfigurationService.svc HTTP/1.1
Connection: keep-alive
Referer: http://[HOST]/MailEssentials_Services/MultiNodeConfigurationService.svc
SOAPAction: http://tempuri.org/IMultiNodeConfigurationService/InstallClientCertificate
Content-Type: text/xml;charset=UTF-8
Host: [HOST]
Content-Length: [CONTENT_LENGTH]

<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope" xmlns:tem="http://tempuri.org/">
   <soap:Header/>
   <soap:Body>
      <tem:InstallClientCertificate>
         <!--type: StreamBody-->
         <tem:certificateStream>ZQ==</tem:certificateStream>
      </tem:InstallClientCertificate>
   </soap:Body>
</soap:Envelope>
```

Putting a malicious serialized object in Base64 encoded format into `<tem:certificateStream/>`, after correcting the content type to `Content-Type: application/soap+xml; charset=utf-8`, we got a 500 status code response

# Vulnerability III - A Pinch of XML External Entity at the End

o I got through my list of other dangerous sinks and found tons of other interesting stuff such as calls to `System.Xml.XmlDocument::LoadXml(System.String)`: a classic [XML External Entity Injection (XXE Injection)](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/XML%20External%20Entity%20Injection%20(XXE%20Injection).md) sink but with a subtle restriction. 

>[!info]
>As you might know, applications using [.NET Framework >= 4.5.2](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html#net) are (kind of) protected against various XXE attack vectors by default.

Now back to our code base, belonging to the user interface import button click event at `ContentSecurity.Configuration.PhishingKeywords::btnImport_Click(System.Object,System.EventArgs)`.

```cs
protected void btnImport_Click(object sender, EventArgs e)
{
	this.lblImportMessage.Text = string.Empty;
	try
	{
		if (this.fuUlpload.HasFile) // [9]
		{
			if (this.fuUlpload.FileName.ToLowerInvariant().EndsWith("xml")) // [10]
			{
				using (MemoryStream memoryStream = new MemoryStream())
				{
					byte[] array = new byte[16384];
					int num;
					while ((num = this.fuUlpload.PostedFile.InputStream.Read(array, 0, array.Length)) > 0) // [11]
					{
						memoryStream.Write(array, 0, num); // [12]
					}
					memoryStream.Position = 0L;
					MasterClass.GetRemotingHelperInstance().ImportAntiPhisingKeywordList(memoryStream.ToArray(), "xml"); // [13]
				}
				this.TmrUploadStatus.Enabled = true;
			}
			else
			{
				this.lblImportMessage.ForeColor = Color.Red;
				this.lblImportMessage.Text = this.StringsProvider1.GetString("fileTypeNotSupported");
			}
		}
		else
		{
			this.lblImportMessage.ForeColor = Color.Red;
			this.lblImportMessage.Text = this.StringsProvider1.GetString("/AttachmentCheckingAdd1_ascx.NoFile");
		}
	}
	// ...
```

`fuUlpload` is of type `System.Web.UI.WebControls.FileUpload` \[9\], somehow expected for an upload control with an import button. The uploaded file name extensions should end with `.xml` \[10\]. At \[11\] the inputstream of the uploaded file request is read into a byte array which then is written into a `System.IO.MemoryStream` object at \[12\]. Then at \[13\] our `ImportAntiPhisingKeywordList` method gets called with the content.

```cs
public bool ImportAntiPhisingKeywordList(byte[] streamArray, string extension) // [14]
{
	this.WriteToLogFile("Import ImportAntiPhisingKeywordList");
	this.UpdateProgressStatus(ItemInProgress.AntiPhishingKeyword, ImportStatus.Progress, 0, 0);
	List<object> list = new List<object>();
	list.Add(streamArray); // [15]
	list.Add(extension);
	Thread thread = new Thread(new ParameterizedThreadStart(this.ImportAntiPhisingKeywordList)); // [16]
	thread.Start(list);
	return true;
}
```

We know that the `extension` method parameter is set to a fixed value of `xml` \[14\] (see also \[13\]). Then `streamArray` with our XML content is added to a ``System.Collections.Generic.List`1`` with members of type `System.Object` \[15\]. Finally at \[16\], a new `System.Threading.Thread` is created, calling `MEC.Configuration.RemotingHelper::ImportAntiPhisingKeywordList(System.Object)` with `list` as parameter.

We reached our final method, containing the XXE sink described in the very beginning of this chapter.

```cs
private void ImportAntiPhisingKeywordList(object objectThreadObjs)
{
	List<object> list = (List<object>)objectThreadObjs; // [17]
	byte[] array = (byte[])list[0]; // [18]
	string text = (string)list[1]; // [19]
	using (MemoryStream memoryStream = new MemoryStream(array)) // [20]
	{
		try
		{
			int num = 0;
			int num2 = 0;
			using (StreamReader streamReader = new StreamReader(memoryStream)) // [21]
			{
				using (OleDbConnection oleDbConnection = new OleDbConnection(this.GetMEConfigConnString()))
				{
					oleDbConnection.Open();
					string text2 = text.ToLowerInvariant();
					if (!(text2 == "xml")) // [22]
					{
						if (text2 == "txt")
						{
							while (!streamReader.EndOfStream)
							{
								string text3 = streamReader.ReadLine();
								if (text3 != null && text3.Length > 2)
								{
									bool flag = this.SavePhisingKeywordToDb(text3, oleDbConnection);
									if (flag)
									{
										num++;
										this.UpdateProgressStatus(ItemInProgress.AntiPhishingKeyword, ImportStatus.Progress, num, num2);
									}
									else
									{
										num2--;
									}
								}
							}
						}
					}
					else
					{
						string text4 = streamReader.ReadToEnd();
						XmlDocument xmlDocument = new XmlDocument();
						xmlDocument.LoadXml(text4); // [23]
						// ...
```

Our `objectThreadObjs` is again “casted” into a `List<object>` \[17\]. The first entry \[18\] contains the XML itself, the second one \[19\] the file extension. The XML content is put into a `System.IO.MemoryStream` \[20\] to be finally used in a `System.IO.StreamReader` object \[21\]. If you recall, our extension equals `xml` so the `if` case at \[22\] is *not* taken. In the `else` branch we hit the XXE sink at \[23\] instead: our full chain from web user interface user control to XXE sink got traced back successfully.

But could XXE work here? What does the .NET Framework version of this GFI MailEssentials .NET Assembly say?

```cs
// C:\Program Files (x86)\GFI\MailEssentials\Backend\bin\MEC.Configuration.dll
// MEC.Configuration.dll

// Global type: <Module>
// Architecture: AnyCPU (64-bit preferred)
// Runtime: .NET Framework 4
// Timestamp: 62B091B8 (6/20/2022 8:26:48 AM)
```

PoC time! I used my favorite .NET XXE payload from [this gist](https://gist.github.com/staaldraad/01415b990939494879b4). As a two-stage payload, it fetches the file `ev.xml` from my attacker’s machine then reading the referenced file `C:\Windows\win.ini` locally on server-side, finally providing its content with a second HTTP request to the attacker machine again.

[^1]: `inetmgr` (Internet Information Services Manager) is the executable command in Windows to open the IIS management console, allowing users to configure web servers, websites, and application pools.

[^2]: [ysonet](../Raindrop/GitHub%20-%20irsdlysonet%20Deserialization%20payload%20generator%20for%20a%20variety%20of%20.NET%20formatters.md)
