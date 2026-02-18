# نشر خوادم MCP

يتيح نشر خادم MCP الخاص بك للآخرين الوصول إلى أدواته وموارده خارج بيئتك المحلية. هناك عدة استراتيجيات للنشر يجب النظر فيها، اعتمادًا على متطلباتك من حيث القابلية للتوسع، والموثوقية، وسهولة الإدارة. أدناه ستجد إرشادات لنشر خوادم MCP محليًا، في الحاويات، وإلى السحابة.

## نظرة عامة

تغطي هذه الدرس كيفية نشر تطبيق خادم MCP الخاص بك.

## أهداف التعلم

بحلول نهاية هذا الدرس، ستكون قادرًا على:

- تقييم أساليب النشر المختلفة.
- نشر تطبيقك.

## التطوير والنشر المحلي

إذا كان الغرض من خادمك هو أن يتم استخدامه من خلال تشغيله على جهاز المستخدم، يمكنك اتباع الخطوات التالية:

1. **تحميل الخادم**. إذا لم تكتب الخادم، قم بتحميله أولًا إلى جهازك.
1. **بدء عملية الخادم**: شغّل تطبيق خادم MCP الخاص بك.

بالنسبة لـ SSE (غير مطلوب لخادم النوع stdio)

1. **تكوين الشبكة**: تأكد من وصول الخادم إلى المنفذ المتوقع.
1. **اتصال العملاء**: استخدم عناوين الاتصال المحلية مثل `http://localhost:3000`

## النشر على السحابة

يمكن نشر خوادم MCP على مختلف منصات السحابة:

- **الدوال بدون خادم**: نشر خوادم MCP خفيفة الوزن كدوال بدون خادم.
- **خدمات الحاويات**: استخدام خدمات مثل Azure Container Apps، AWS ECS، أو Google Cloud Run.
- **Kubernetes**: نشر وإدارة خوادم MCP في مجموعات Kubernetes لتوفير توافر عالي.

### مثال: Azure Container Apps

تدعم Azure Container Apps نشر خوادم MCP. لا يزال هذا العمل قيد التقدم ويدعم حاليًا خوادم SSE.

إليك كيف يمكنك القيام بذلك:

1. قم باستنساخ مستودع:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. شغّله محليًا لاختبار الأمور:

  ```sh
  uv venv
  uv sync

  # لينكس/ماك أو إس
  export API_KEYS=<AN_API_KEY>
  # ويندوز
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. لتجربته محليًا، أنشئ ملف *mcp.json* في مجلد *.vscode* وأضف المحتوى التالي:

  ```json
  {
      "inputs": [
          {
              "type": "promptString",
              "id": "weather-api-key",
              "description": "Weather API Key",
              "password": true
          }
      ],
      "servers": {
          "weather-sse": {
              "type": "sse",
              "url": "http://localhost:8000/sse",
              "headers": {
                  "x-api-key": "${input:weather-api-key}"
              }
          }
      }
  }
  ```

  بمجرد تشغيل خادم SSE، يمكنك النقر على أيقونة التشغيل في ملف JSON، يجب أن ترى الآن الأدوات على الخادم يتم التقاطها بواسطة GitHub Copilot، انظر أيقونة الأداة.

1. للنشر، شغّل الأمر التالي:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

ها أنت ذا، انشره محليًا، وانشره إلى Azure من خلال هذه الخطوات.

## موارد إضافية

- [دوال Azure + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [مقال Azure Container Apps](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [مستودع Azure Container Apps MCP](https://github.com/anthonychu/azure-container-apps-mcp-sample)

## ما التالي

- التالي: [مواضيع متقدمة في الخادم](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**تنبيه**:  
تمت ترجمة هذا الوثيقة باستخدام خدمة الترجمة الآلية [Co-op Translator](https://github.com/Azure/co-op-translator). بينما نسعى لتحقيق الدقة، يُرجى العلم أن الترجمات الآلية قد تحتوي على أخطاء أو عدم دقة. يجب اعتبار الوثيقة الأصلية بلغتها الأصلية المصدر الرسمي والمعتمد. بالنسبة للمعلومات الحساسة أو الهامة، يُنصح بالاستعانة بترجمة إنسانية محترفة. نحن غير مسؤولين عن أي سوء فهم أو تفسير ناتج عن استخدام هذه الترجمة.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->