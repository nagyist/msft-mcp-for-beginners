# פריסת שרתי MCP

פריסת שרת MCP שלך מאפשרת לאחרים לגשת לכלים ולמשאבים שלו מעבר לסביבת המחשב המקומית שלך. ישנן מספר אסטרטגיות פריסה לשקול, בהתאם לדרישות שלך עבור יכולת סקלציה, אמינות ונוחות ניהול. למטה תמצא הנחיות לפריסת שרתי MCP באופן מקומי, במכולות, ולענן.

## סקירה כללית

שיעור זה מכסה כיצד לפרוס את אפליקציית שרת MCP שלך.

## מטרות הלמידה

בסיום שיעור זה, תוכל/י:

- להעריך גישות פריסה שונות.
- לפרוס את האפליקציה שלך.

## פיתוח ופריסה מקומית

אם השרת שלך מיועד לשימוש על ידי הרצה במחשב המשתמש, תוכל/י לבצע את השלבים הבאים:

1. **הורד את השרת**. אם לא כתבתי את השרת, הורד אותו תחילה למחשב שלך.
1. **הפעל את תהליך השרת**: הרץ את אפליקציית שרת MCP שלך.

ל־SSE (לא נדרש עבור שרת מסוג stdio)

1. **הגדר רישות רשת**: ודא שהשרת נגיש בכתובת הפורט הצפוי
1. **חבר לקוחות**: השתמש בכתובות חיבור מקומיות כגון `http://localhost:3000`

## פריסת ענן

ניתן לפרוס שרתי MCP בפלטפורמות ענן שונות:

- **פונקציות ללא שרת (Serverless)**: פרוס שרתי MCP קלים כפונקציות ללא שרת
- **שירותי מכולות**: השתמש בשירותים כמו Azure Container Apps, AWS ECS, או Google Cloud Run
- **קוברנטיס**: פרוס ונהל שרתי MCP באשכולות Kubernetes לזמינות גבוהה

### דוגמה: Azure Container Apps

Azure Container Apps תומכת בפריסת שרתי MCP. זה עדיין בתהליך עבודה וכעת תומך בשרתי SSE.

כך תוכל/י לעשות זאת:

1. שיבוט מאגר:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. הרץ את זה מקומית כדי לבדוק:

  ```sh
  uv venv
  uv sync

  # לינוקס/מק או אס
  export API_KEYS=<AN_API_KEY>
  # חלונות
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. כדי לנסות זאת מקומית, צור קובץ *mcp.json* בתיקיית *.vscode* והוסף את התוכן הבא:

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

  לאחר שהשרת SSE הופעל, תוכל/י ללחוץ על סמל ההפעלה בקובץ ה־JSON, כעת אמורים להופיע כלים על השרת שיזהו אותם GitHub Copilot, ראה את סמל הכלי.

1. לפריסה, הרץ את הפקודה הבאה:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

זה הכל, פרוס את השרת באופן מקומי, או לפרוס אותו ל־Azure באמצעות השלבים הללו.

## משאבים נוספים

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [מאמר על Azure Container Apps](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [מאגר Azure Container Apps MCP](https://github.com/anthonychu/azure-container-apps-mcp-sample)


## מה הלאה

- הבא: [נושאים מתקדמים בשרת](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**כתב ויתור**:
מסמך זה תורגם באמצעות שירות תרגום בינה מלאכותית [Co-op Translator](https://github.com/Azure/co-op-translator). למרות שאנו שואפים לדיוק, יש לקחת בחשבון כי תרגומים אוטומטיים עלולים להכיל שגיאות או אי דיוקים. יש להתייחס למסמך המקורי בשפתו המקורית כמקור הסמכותי. עבור מידע קריטי מומלץ לבצע תרגום מקצועי על ידי מתרגם אנושי. אנו לא נישא באחריות לכל הבנה שגויה או פרשנות מוטעית הנובעת מהשימוש בתרגום זה.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->