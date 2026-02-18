# التنفيذ العملي

[![كيفية بناء واختبار ونشر تطبيقات MCP باستخدام أدوات وسير عمل حقيقية](../../../translated_images/ar/05.64bea204e25ca891.webp)](https://youtu.be/vCN9-mKBDfQ)

_(انقر على الصورة أعلاه لمشاهدة فيديو هذا الدرس)_

التنفيذ العملي هو حيث تصبح قوة بروتوكول سياق النموذج (MCP) ملموسة. بينما يعد فهم النظرية والهندسة المعمارية وراء MCP مهمًا، تظهر القيمة الحقيقية عند تطبيق هذه المفاهيم لبناء واختبار ونشر حلول تحل مشاكل العالم الحقيقي. هذا الفصل يجسر الفجوة بين المعرفة المفهومية والتطوير العملي، ويوجهك خلال عملية إحياء التطبيقات المبنية على MCP.

سواء كنت تطور مساعدين أذكياء، أو تدمج الذكاء الاصطناعي في سير عمل الأعمال، أو تبني أدوات مخصصة لمعالجة البيانات، يوفر MCP أساسًا مرنًا. تصميمه المستقل عن اللغة وأدوات تطوير البرمجيات الرسمية للغات البرمجة الشهيرة يجعله متاحًا لمجموعة واسعة من المطورين. من خلال الاستفادة من هذه الأدوات، يمكنك بسرعة إنشاء نماذج أولية، وتكرارها، وتوسيع حلولك عبر منصات وبيئات مختلفة.

في الأقسام التالية، ستجد أمثلة عملية، وأكواد عينة، واستراتيجيات نشر توضح كيفية تنفيذ MCP في C#، وJava مع Spring، وTypeScript، وJavaScript، وPython. ستتعلم أيضًا كيفية تصحيح واختبار خوادم MCP، وإدارة واجهات برمجة التطبيقات، ونشر الحلول على السحابة باستخدام Azure. تم تصميم هذه الموارد العملية لتسريع تعلمك ومساعدتك على بناء تطبيقات MCP قوية وجاهزة للإنتاج بكل ثقة.

## نظرة عامة

يركز هذا الدرس على الجوانب العملية لتنفيذ MCP عبر عدة لغات برمجة. سنستعرض كيفية استخدام أدوات MCP في C#، وJava مع Spring، وTypeScript، وJavaScript، وPython لبناء تطبيقات متينة، وتصحيح واختبار خوادم MCP، وإنشاء موارد وقوالب وأدوات قابلة لإعادة الاستخدام.

## أهداف التعلم

بنهاية هذا الدرس، ستكون قادرًا على:

- تنفيذ حلول MCP باستخدام الأدوات الرسمية في لغات برمجة مختلفة
- تصحيح واختبار خوادم MCP بشكل منهجي
- إنشاء واستخدام ميزات الخادم (الموارد، والقوالب، والأدوات)
- تصميم سير عمل MCP فعال للمهام المعقدة
- تحسين تنفيذات MCP من أجل الأداء والموثوقية

## موارد الأدوات الرسمية

يوفر بروتوكول سياق النموذج أدوات رسمية متعددة اللغات (متوافقة مع [مواصفة MCP 2025-11-25](https://spec.modelcontextprotocol.io/specification/2025-11-25/)):

- [أداة C# الرسمية](https://github.com/modelcontextprotocol/csharp-sdk)
- [أداة Java مع Spring الرسمية](https://github.com/modelcontextprotocol/java-sdk) **ملاحظة:** تتطلب الاعتماد على [Project Reactor](https://projectreactor.io). (انظر [نقاش رقم 246](https://github.com/orgs/modelcontextprotocol/discussions/246).)
- [أداة TypeScript الرسمية](https://github.com/modelcontextprotocol/typescript-sdk)
- [أداة Python الرسمية](https://github.com/modelcontextprotocol/python-sdk)
- [أداة Kotlin الرسمية](https://github.com/modelcontextprotocol/kotlin-sdk)
- [أداة Go الرسمية](https://github.com/modelcontextprotocol/go-sdk)

## العمل مع أدوات MCP

يوفر هذا القسم أمثلة عملية لتنفيذ MCP عبر لغات برمجة متعددة. يمكنك العثور على أكواد عينة في دليل `samples` منظمة حسب اللغة.

### العينات المتاحة

يوجد في المستودع [عينات تنفيذية](../../../04-PracticalImplementation/samples) باللغات التالية:

- [C#](./samples/csharp/README.md)
- [Java مع Spring](./samples/java/containerapp/README.md)
- [TypeScript](./samples/typescript/README.md)
- [JavaScript](./samples/javascript/README.md)
- [Python](./samples/python/README.md)

كل عينة توضح مفاهيم وأنماط تنفيذ رئيسية لـ MCP للغة والنظام البيئي المحددين.

### الأدلة العملية

أدلة إضافية لتنفيذ MCP عملي:

- [التصفح والنتائج الكبيرة](./pagination/README.md) - معالجة التصفح القائم على المؤشرات للأدوات والموارد ومجموعات البيانات الكبيرة

## ميزات الخادم الأساسية

يمكن لخوادم MCP تنفيذ أي تركيبة من الميزات التالية:

### الموارد

توفر الموارد سياقًا وبيانات لاستخدام المستخدم أو نموذج الذكاء الاصطناعي:

- مستودعات الوثائق
- قواعد المعرفة
- مصادر البيانات المنظمة
- أنظمة الملفات

### القوالب

القوالب هي رسائل وسير عمل نمطية للمستخدمين:

- قوالب محادثة معرفة مسبقًا
- أنماط تفاعل موجهة
- هياكل حوار متخصصة

### الأدوات

الأدوات هي وظائف لتنفيد نموذج الذكاء الاصطناعي:

- أدوات معالجة البيانات
- تكاملات واجهات برمجة التطبيقات الخارجية
- إمكانيات حسابية
- وظائف البحث

## عينات التنفيذ: تنفيذ C#

يحتوي مستودع أداة C# الرسمية على عدة عينات تنفيذ توضح جوانب مختلفة من MCP:

- **عميل MCP أساسي**: مثال بسيط يوضح كيفية إنشاء عميل MCP واستدعاء أدوات
- **خادم MCP أساسي**: تنفيذ خادم الحد الأدنى مع تسجيل أدوات أساسي
- **خادم MCP متقدم**: خادم كامل الميزات مع تسجيل أدوات، ومصادقة، ومعالجة أخطاء
- **تكامل ASP.NET**: أمثلة توضح التكامل مع ASP.NET Core
- **أنماط تنفيذ الأدوات**: أنماط متنوعة لتنفيذ الأدوات بمستويات تعقيد مختلفة

أداة MCP في C# في مرحلة العرض المسبق وقد تتغير واجهات برمجة التطبيقات. سنواصل تحديث هذه المدونة مع تطور الأداة.

### الميزات الرئيسية

- [C# MCP Nuget ModelContextProtocol](https://www.nuget.org/packages/ModelContextProtocol)
- بناء [أول خادم MCP لك](https://devblogs.microsoft.com/dotnet/build-a-model-context-protocol-mcp-server-in-csharp/).

للحصول على عينات تنفيذ C# كاملة، قم بزيارة [مستودع عينات أداة C# الرسمية](https://github.com/modelcontextprotocol/csharp-sdk)

## عينات التنفيذ: تنفيذ Java مع Spring

توفر أداة Java مع Spring خيارات تنفيذ MCP قوية مع ميزات من مستوى المؤسسات.

### الميزات الرئيسية

- تكامل إطار عمل Spring
- أمان الأنواع القوي
- دعم البرمجة التفاعلية
- معالجة شاملة للأخطاء

لعينة كاملة لتنفيذ Java مع Spring، راجع [عينة Java مع Spring](samples/java/containerapp/README.md) في مجلد العينات.

## عينات التنفيذ: تنفيذ JavaScript

تقدم أداة JavaScript نهجًا خفيف الوزن ومرن لتنفيذ MCP.

### الميزات الرئيسية

- دعم Node.js والمتصفح
- واجهة برمجة تطبيقات معتمدة على الوعود (Promises)
- سهولة التكامل مع Express وأطر عمل أخرى
- دعم WebSocket للبث

لعينة تنفيذ JavaScript كاملة، راجع [عينة JavaScript](samples/javascript/README.md) في مجلد العينات.

## عينات التنفيذ: تنفيذ Python

تقدم أداة Python نهجًا بايثونيًا لتنفيذ MCP مع تكامل ممتاز مع أطر تعلم الآلة.

### الميزات الرئيسية

- دعم async/await باستخدام asyncio
- تكامل FastAPI
- تسجيل أدوات بسيط
- تكامل أصلي مع مكتبات تعلم الآلة الشهيرة

لعينة تنفيذ Python كاملة، راجع [عينة Python](samples/python/README.md) في مجلد العينات.

## إدارة واجهة برمجة التطبيقات

Azure API Management هو حل رائع لكيفية تأمين خوادم MCP. الفكرة هي وضع مثيل Azure API Management أمام خادم MCP الخاص بك وتركه يتولى الميزات التي من المحتمل أن تحتاجها مثل:

- تحديد معدلات الطلب
- إدارة الرموز
- المراقبة
- توازن الأحمال
- الأمان

### عينة Azure

إليك عينة Azure تقوم بذلك بالضبط، أي [إنشاء خادم MCP وتأمينه باستخدام Azure API Management](https://github.com/Azure-Samples/remote-mcp-apim-functions-python).

شاهد كيف تتم عملية التفويض في الصورة أدناه:

![APIM-MCP](https://github.com/Azure-Samples/remote-mcp-apim-functions-python/blob/main/mcp-client-authorization.gif?raw=true)

في الصورة السابقة، تتم العمليات التالية:

- تتم المصادقة/التفويض باستخدام Microsoft Entra.
- يعمل Azure API Management كبوابة ويستخدم السياسات لتوجيه وإدارة حركة المرور.
- يسجل Azure Monitor جميع الطلبات لتحليلها لاحقًا.

#### تدفق التفويض

لنلق نظرة أكثر تفصيلاً على تدفق التفويض:

![مخطط تتابعي](https://github.com/Azure-Samples/remote-mcp-apim-functions-python/blob/main/infra/app/apim-oauth/diagrams/images/mcp-client-auth.png?raw=true)

#### مواصفة تفويض MCP

تعرف على المزيد حول [مواصفة تفويض MCP](https://spec.modelcontextprotocol.io/specification/2025-11-25/basic/authorization/)

## نشر خادم MCP عن بُعد على Azure

لنرَ إذا كان بإمكاننا نشر العينة التي ذكرناها سابقًا:

1. استنساخ المستودع

    ```bash
    git clone https://github.com/Azure-Samples/remote-mcp-apim-functions-python.git
    cd remote-mcp-apim-functions-python
    ```

1. تسجيل مزود المورد `Microsoft.App`.

   - إذا كنت تستخدم Azure CLI، نفذ الأمر `az provider register --namespace Microsoft.App --wait`.
   - إذا كنت تستخدم Azure PowerShell، نفذ الأمر `Register-AzResourceProvider -ProviderNamespace Microsoft.App`. ثم نفذ `(Get-AzResourceProvider -ProviderNamespace Microsoft.App).RegistrationState` بعد فترة للتحقق مما إذا كان التسجيل قد اكتمل.

1. تشغيل أمر [azd](https://aka.ms/azd) هذا لتوفير خدمة إدارة API، وتطبيق الدالة (مع الكود) وكل بقية موارد Azure المطلوبة

    ```shell
    azd up
    ```

    يجب أن تنشر هذه الأوامر جميع موارد السحابة على Azure

### اختبار خادمك باستخدام MCP Inspector

1. في نافذة طرفية **جديدة**، ثبت وقم بتشغيل MCP Inspector

    ```shell
    npx @modelcontextprotocol/inspector
    ```

    يجب أن ترى واجهة مشابهة لـ:

    ![الاتصال بـ Node inspector](../../../translated_images/ar/connect.141db0b2bd05f096.webp)

1. اضغط CTRL وانقر لتحميل تطبيق MCP Inspector من عنوان URL المعروض بواسطة التطبيق (مثلاً [http://127.0.0.1:6274/#resources](http://127.0.0.1:6274/#resources))
1. اضبط نوع النقل إلى `SSE`
1. اضبط عنوان URL إلى نقطة نهاية إدارة API لـ SSE المعروضة بعد `azd up` ثم **اتصل**:

    ```shell
    https://<apim-servicename-from-azd-output>.azure-api.net/mcp/sse
    ```

1. **قائمة الأدوات**. انقر على أداة وقم بـ **تشغيل الأداة**.

إذا نجحت كل الخطوات، يجب أن تكون متصلًا الآن بخادم MCP وأن تكون قادرًا على استدعاء أداة.

## خوادم MCP لـ Azure

[Remote-mcp-functions](https://github.com/Azure-Samples/remote-mcp-functions-dotnet): هذه المجموعة من المستودعات هي قالب بدء سريع لبناء ونشر خوادم MCP (بروتوكول سياق النموذج) عن بُعد مخصصة باستخدام Azure Functions مع Python, C# .NET أو Node/TypeScript.

يوفر النموذج حلاً كاملاً يسمح للمطورين بـ:

- البناء والتشغيل محليًا: تطوير وتصحيح خادم MCP على جهاز محلي
- النشر إلى Azure: نشر سهل إلى السحابة بأمر azd up بسيط
- الاتصال من العملاء: الاتصال بخادم MCP من عملاء متعددين بما في ذلك وضع وكيل Copilot في VS Code وأداة MCP Inspector

### الميزات الرئيسية

- الأمان بشكل افتراضي: خادم MCP مؤمن باستخدام مفاتيح و HTTPS
- خيارات المصادقة: دعم OAuth باستخدام المصادقة المدمجة و/أو إدارة API
- عزل الشبكة: يسمح بعزل الشبكة باستخدام شبكات Azure الافتراضية (VNET)
- البنية بدون خادم: يستفيد من Azure Functions لتنفيذ قابل للتوسع وتحفيزي
- التطوير المحلي: دعم شامل للتطوير المحلي وتصحيح الأخطاء
- النشر البسيط: عملية نشر مبسطة إلى Azure

يتضمن المستودع كل ملفات التكوين الضرورية، والكود المصدري، وتعريفات البنية التحتية للبدء السريع في تنفيذ خادم MCP جاهز للإنتاج.

- [Azure Remote MCP Functions Python](https://github.com/Azure-Samples/remote-mcp-functions-python) - عينة تنفيذ MCP باستخدام Azure Functions مع Python

- [Azure Remote MCP Functions .NET](https://github.com/Azure-Samples/remote-mcp-functions-dotnet) - عينة تنفيذ MCP باستخدام Azure Functions مع C# .NET

- [Azure Remote MCP Functions Node/Typescript](https://github.com/Azure-Samples/remote-mcp-functions-typescript) - عينة تنفيذ MCP باستخدام Azure Functions مع Node/TypeScript.

## النقاط الرئيسية

- توفر أدوات MCP أدوات خاصة باللغات لتنفيذ حلول MCP قوية
- عملية التصحيح والاختبار ضرورية لتطبيقات MCP موثوقة
- تُمكن قوالب القوالب القابلة لإعادة الاستخدام من تفاعلات ذكاء اصطناعي متسقة
- يمكن لسير العمل المصمم جيدًا تنسيق مهام معقدة باستخدام أدوات متعددة
- يتطلب تنفيذ حلول MCP مراعاة الأمان، والأداء، ومعالجة الأخطاء

## تمرين

صمم سير عمل MCP عملي يعالج مشكلة واقعية في مجالك:

1. حدد 3-4 أدوات ستكون مفيدة لحل هذه المشكلة
2. أنشئ مخطط سير يظهر كيف تتفاعل هذه الأدوات
3. نفذ نسخة أساسية من إحدى الأدوات باستخدام لغتك المفضلة
4. أنشئ قالب طلب يساعد النموذج على استخدام أداتك بفعالية

## موارد إضافية

---

## ماذا بعد

التالي: [المواضيع المتقدمة](../05-AdvancedTopics/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**إخلاء المسؤولية**:  
تمت ترجمة هذا المستند باستخدام خدمة الترجمة الآلية [Co-op Translator](https://github.com/Azure/co-op-translator). بينما نسعى جاهدين لتحقيق الدقة، يرجى العلم أن الترجمات الآلية قد تحتوي على أخطاء أو عدم دقة. يجب اعتبار المستند الأصلي بلغته الأصلية المصدر الرسمي والموثوق. يُنصح باستخدام الترجمة الاحترافية البشرية للمعلومات الحساسة أو الهامة. نحن غير مسؤولين عن أي سوء فهم أو تفسير ناتج عن استخدام هذه الترجمة.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->