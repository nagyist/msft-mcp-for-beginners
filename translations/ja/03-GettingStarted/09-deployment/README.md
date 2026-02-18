# MCP サーバーのデプロイ

MCP サーバーをデプロイすると、ローカル環境を超えて他の人がそのツールやリソースにアクセスできるようになります。スケーラビリティ、信頼性、管理のしやすさに応じて、いくつかのデプロイ戦略を検討できます。以下では、ローカル、コンテナ、およびクラウドでの MCP サーバーのデプロイに関するガイドを示します。

## 概要

このレッスンでは、MCP サーバーアプリのデプロイ方法を扱います。

## 学習目標

このレッスンの終了時には、次のことができるようになります。

- さまざまなデプロイアプローチの評価。
- アプリのデプロイ。

## ローカル開発とデプロイ

サーバーがユーザーのマシンで実行されるために消費されることを想定している場合は、以下の手順に従ってください：

1. **サーバーのダウンロード**。サーバーを書いていない場合、まず自分のマシンにダウンロードします。 
1. **サーバープロセスの起動**：MCP サーバーアプリケーションを実行します。

SSE（stdio タイプのサーバーには不要）の場合

1. **ネットワーク設定**：サーバーが期待されるポートでアクセス可能であることを確認します。
1. **クライアントの接続**：`http://localhost:3000` のようなローカル接続 URL を使用します。

## クラウドデプロイ

MCP サーバーはさまざまなクラウドプラットフォームにデプロイ可能です：

- **サーバーレス関数**：軽量の MCP サーバーをサーバーレス関数としてデプロイ。
- **コンテナサービス**：Azure Container Apps、AWS ECS、Google Cloud Run のようなサービスを使用。
- **Kubernetes**：高可用性のために Kubernetes クラスター内で MCP サーバーをデプロイおよび管理。

### 例：Azure Container Apps

Azure Container Apps は MCP サーバーのデプロイをサポートしています。これはまだ開発途中で、現在は SSE サーバーをサポートしています。

以下の手順で進められます：

1. リポジトリをクローン：

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. ローカルで実行してテスト：

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. ローカルで試す場合は、*.vscode* ディレクトリ内に *mcp.json* ファイルを作成し、以下の内容を追加：

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

  SSE サーバーが起動したら、JSON ファイルの再生アイコンをクリックすると、サーバー上のツールが GitHub Copilot に認識され、ツールアイコンが表示されるはずです。

1. デプロイするには、次のコマンドを実行：

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

以上で、ローカルにデプロイし、これらの手順を通じて Azure にデプロイできます。

## 追加リソース

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Azure Container Apps 記事](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Azure Container Apps MCP リポジトリ](https://github.com/anthonychu/azure-container-apps-mcp-sample)

## 次に進む

- 次へ: [高度なサーバートピック](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責事項**：  
本書類はAI翻訳サービス「Co-op Translator」（https://github.com/Azure/co-op-translator）を使用して翻訳されています。正確性には努めておりますが、自動翻訳には誤りや不正確な部分が含まれる可能性があることをご了承ください。原文が正本となりますので、重要な内容については専門の人間による翻訳を推奨いたします。本翻訳の利用により生じた誤解や解釈の相違について、当方は一切責任を負いかねます。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->