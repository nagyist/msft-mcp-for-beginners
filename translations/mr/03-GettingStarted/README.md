## Getting Started  

[![Build Your First MCP Server](../../../translated_images/mr/04.0ea920069efd979a.webp)](https://youtu.be/sNDZO9N4m9Y)

_(या धड्याचा व्हिडिओ पाहण्यासाठी वरच्या प्रतिमेवर क्लिक करा)_

हा विभाग अनेक धडे समाविष्ट करतो:

- **1 आपला पहिला सर्व्हर**, या पहिल्या धड्यात, आपण आपला पहिला सर्व्हर कसा तयार करायचा आणि inspector टूलसह कसा तपासायचा हे शिकाल, जे आपल्या सर्व्हरची चाचणी आणि डीबगिंग करण्याचा एक मौल्यवान मार्ग आहे, [धड्यासाठी](01-first-server/README.md)

- **2 क्लायंट**, या धड्यात, आपण असा क्लायंट लिहिणे शिकाल जो आपल्या सर्व्हरशी कनेक्ट होऊ शकेल, [धड्यासाठी](02-client/README.md)

- **3 LLM सह क्लायंट**, क्लायंट लिहिण्याचा आणखी एक चांगला मार्ग म्हणजे त्यात LLM जोडणे जेणेकरून तो आपल्या सर्व्हरशी "वाटाघाट" करू शकेल की काय करावे, [धड्यासाठी](03-llm-client/README.md)

- **4 Visual Studio Code मध्ये सर्व्हर GitHub Copilot Agent मोड वापरणे**. येथे, आपण Visual Studio Code मधून आपला MCP सर्व्हर चालवण्याकडे पाहत आहोत, [धड्यासाठी](04-vscode/README.md)

- **5 stdio ट्रान्सपोर्ट सर्व्हर** stdio ट्रान्सपोर्ट स्थानिक MCP सर्व्हर-टू-क्लायंट संवादासाठी शिफारस केलेला स्टँडर्ड आहे, जो अंतर्निर्मित प्रक्रिया पृथक्करणासह सुरक्षित subprocess-आधारित संवाद प्रदान करतो [धड्यासाठी](05-stdio-server/README.md)

- **6 MCP सह HTTP स्ट्रीमिंग (Streamable HTTP)**. आधुनिक HTTP स्ट्रीमिंग ट्रान्सपोर्ट (मोडर्न HTTP स्ट्रीमिंग ट्रान्सपोर्ट) बद्दल जाणून घ्या (दूरच्या MCP सर्व्हरसाठी शिफारस केलेला मार्ग [MCP Specification 2025-11-25](https://spec.modelcontextprotocol.io/specification/2025-11-25/basic/transports/#streamable-http) नुसार), प्रगती सूचना, आणि Streamable HTTP वापरून स्केलेबल, रिअल-टाईम MCP सर्व्हर आणि क्लायंट कसे राबवायचे. [धड्यासाठी](06-http-streaming/README.md)

- **7 VSCode साठी AI Toolkit वापरणे** आपल्या MCP क्लायंट्स आणि सर्व्हरचे उपभोग आणि चाचणी करण्यासाठी [धड्यासाठी](07-aitk/README.md)

- **8 चाचणी**. येथे आपण विशेषतः आपला सर्व्हर आणि क्लायंट वेगवेगळ्या पद्धतीने कसे टेस्ट करू शकतो याकडे लक्ष केंद्रित करू, [धड्यासाठी](08-testing/README.md)

- **9 तैनाती**. हा प्रकरण आपल्या MCP सोल्यूशन्सचे वेगवेगळ्या प्रकारे तैनाती कशी करायची याकडे पाहील, [धड्यासाठी](09-deployment/README.md)

- **10 प्रगत सर्व्हर वापर**. हा प्रकरण प्रगत सर्व्हर वापरावर कव्हर करतो, [धड्यासाठी](./10-advanced/README.md)

- **11 प्राधिकरण (Auth)**. या प्रकरणात सोपी प्रमाणीकरण कशी जोडायची याबद्दल चर्चा आहे, बेसिक Auth पासून JWT आणि RBAC वापरापर्यंत. आपण येथे सुरु करण्यासाठी प्रोत्साहित केले जाता आणि मग Advanced Topics प्रकरण 5 मध्ये पाहा आणि प्रकरण 2 मधील शिफारशींनुसार अतिरिक्त सुरक्षा सख्ती करा, [धड्यासाठी](./11-simple-auth/README.md)

- **12 MCP होस्ट्स**. Claude Desktop, Cursor, Cline, आणि Windsurf यांसह लोकप्रिय MCP होस्ट क्लायंट्स कसे कॉन्फिगर आणि वापरायचे ते शिका. ट्रान्सपोर्ट प्रकार आणि समस्या निवारण बद्दल जाणून घ्या, [धड्यासाठी](./12-mcp-hosts/README.md)

- **13 MCP Inspector**. MCP Inspector टूल वापरून आपण आपल्या MCP सर्व्हरचे इंटरएक्टिव डीबग आणि चाचणी करू शकता. टूल्स, संसाधने, आणि प्रोटोकॉल मेसेजेस कसे त्रुटीशोधन करायचे ते शिका, [धड्यासाठी](./13-mcp-inspector/README.md)

Model Context Protocol (MCP) हे एक खुले प्रोटोकॉल आहे जे LLMs ला संदर्भ प्रदान करण्यासाठी अनुप्रयोगांनी कसे वागावे याचे मानकीकरण करते. MCP एआय अनुप्रयोगांसाठी USB-C पोर्टसारखे समजा - हे AI मॉडेल्सना वेगवेगळ्या डेटा स्रोत आणि टूल्सशी जोडण्याचा मानकीकृत मार्ग उपलब्ध करून देते.

## शिकण्याची उद्दिष्टे

या धड्याच्या शेवटी, आपण पुढील गोष्टी करू शकाल:

- C#, Java, Python, TypeScript, आणि JavaScript मध्ये MCP साठी विकास वातावरण सेट अप करा
- सानुकूल वैशिष्ट्यांसह मूलभूत MCP सर्व्हर तयार करा आणि तैनात करा (संसाधने, प्रॉम्प्ट, आणि टूल्स)
- MCP सर्व्हरशी जोडणाऱ्या होस्ट अनुप्रयोग तयार करा
- MCP अंमलबजावणीची चाचणी करा आणि डीबग करा
- सामान्य सेटअप अडचणी आणि त्यांचे उपाय समजून घ्या
- आपले MCP अंमलबजावणी लोकप्रिय LLM सेवा जोडू शकतील

## आपले MCP वातावरण सेट करणे

MCP सोबत काम सुरू करण्यापूर्वी, आपले विकास वातावरण तयार करणे आणि मूलभूत कार्यप्रवाह समजून घेणे महत्वाचे आहे. या विभागात सुरूवातीचे सेटअप टप्पे मार्गदर्शन केले जातील जेणेकरून MCP सोबत सुरळीत सुरुवात होईल.

### पूर्वआवश्यकता

MCP विकासात उतरण्यापूर्वी, खात्री करा की:

- **विकास वातावरण**: आपली निवडलेली भाषा (C#, Java, Python, TypeScript, किंवा JavaScript) साठी
- **IDE/एडिटर**: Visual Studio, Visual Studio Code, IntelliJ, Eclipse, PyCharm, किंवा कोणताही आधुनिक कोड एडिटर
- **पॅकेज मॅनेजर्स**: NuGet, Maven/Gradle, pip, किंवा npm/yarn
- **API कीज**: आपल्या होस्ट अनुप्रयोगांमध्ये वापरायच्या कोणत्याही AI सेवा साठी


### अधिकृत SDKs

आगामी प्रकरणांमध्ये आपण Python, TypeScript, Java आणि .NET वापरून तयार केलेली सोल्यूशन्स पाहाल. येथे सर्व अधिकृतपणे समर्थीत SDKs आहेत.

MCP अनेक भाषांसाठी अधिकृत SDKs पुरवतो ([MCP Specification 2025-11-25](https://spec.modelcontextprotocol.io/specification/2025-11-25/) नुसार):

- [C# SDK](https://github.com/modelcontextprotocol/csharp-sdk) - Microsoft सह सहकार्याने देखरेख
- [Java SDK](https://github.com/modelcontextprotocol/java-sdk) - Spring AI सह सहकार्याने देखरेख
- [TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk) - अधिकृत TypeScript अंमलबजावणी
- [Python SDK](https://github.com/modelcontextprotocol/python-sdk) - अधिकृत Python अंमलबजावणी (FastMCP)
- [Kotlin SDK](https://github.com/modelcontextprotocol/kotlin-sdk) - अधिकृत Kotlin अंमलबजावणी
- [Swift SDK](https://github.com/modelcontextprotocol/swift-sdk) - Loopwork AI सह सहकार्याने देखरेख
- [Rust SDK](https://github.com/modelcontextprotocol/rust-sdk) - अधिकृत Rust अंमलबजावणी
- [Go SDK](https://github.com/modelcontextprotocol/go-sdk) - अधिकृत Go अंमलबजावणी

## मुख्य मुद्दे

- भाषा-विशिष्ट SDKs सह MCP विकास वातावरण सेट करणे सोपे आहे
- MCP सर्व्हर तयार करण्यासाठी स्पष्ट schemas सह टूल्स तयार करणे आणि नोंदणी करणे आवश्यक आहे
- MCP क्लायंट्स सर्व्हर आणि मॉडेल्सशी कनेक्ट होतात आणि विस्तारित क्षमता वापरतात
- विश्वसनीय MCP अंमलबजावणीसाठी चाचणी आणि डीबगिंग महत्त्वाचे आहे
- स्थानिक विकासापासून क्लाउड-आधारित सोल्यूशन्सपर्यंत अनेक तैनाती पर्याय उपलब्ध आहेत

## सराव

आमच्याकडे काही नमुने आहेत जे या विभागातील सर्व प्रकरणांमध्ये आपल्याला दिसतील त्या व्यायामांस पूरक आहेत. शिवाय प्रत्येक प्रकरणाचे स्वतःचे व्यायाम आणि असाइनमेंट्स देखील आहेत

- [Java Calculator](./samples/java/calculator/README.md)
- [.Net Calculator](../../../03-GettingStarted/samples/csharp)
- [JavaScript Calculator](./samples/javascript/README.md)
- [TypeScript Calculator](./samples/typescript/README.md)
- [Python Calculator](../../../03-GettingStarted/samples/python)

## अतिरिक्त संसाधने

- [Azure वर Model Context Protocol वापरून एजंट्स तयार करा](https://learn.microsoft.com/azure/developer/ai/intro-agents-mcp)
- [Azure Container Apps सह Remote MCP (Node.js/TypeScript/JavaScript)](https://learn.microsoft.com/samples/azure-samples/mcp-container-ts/mcp-container-ts/)
- [.NET OpenAI MCP एजंट](https://learn.microsoft.com/samples/azure-samples/openai-mcp-agent-dotnet/openai-mcp-agent-dotnet/)

## पुढे काय

पहिल्या धड्यासह सुरू करा: [आपला पहिला MCP सर्व्हर तयार करत आहे](01-first-server/README.md)

हा मोड्यूल पूर्ण झाल्यावर पुढे जा: [मोड्यूल 4: व्यावहारिक अंमलबजावणी](../04-PracticalImplementation/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
हा दस्तऐवज AI भाषांतर सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) चा वापर करून भाषांतरित केला आहे. आम्ही अचूकतेसाठी प्रयत्नशील असलो तरी, कृपया लक्षात ठेवा की स्वयंचलित भाषांतरे काही चुका किंवा अचूकतेत कमतरता असू शकतात. मूळ दस्तऐवज त्याच्या स्थानिक भाषेत अधिकृत स्रोत म्हणून समजला पाहिजे. महत्त्वपूर्ण माहिती साठी व्यावसायिक मानवी भाषांतर शिफारसीय आहे. या भाषांतराच्या वापरामुळे उद्भवलेल्या कोणत्याही गैरसमज किंवा चुकीच्या अर्थ लावणीबद्दल आम्ही जबाबदार नाही.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->