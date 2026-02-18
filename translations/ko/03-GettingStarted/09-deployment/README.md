# MCP 서버 배포

MCP 서버를 배포하면 로컬 환경을 넘어 다른 사람들이 해당 도구와 리소스에 접근할 수 있습니다. 확장성, 안정성 및 관리 용이성에 대한 요구 사항에 따라 여러 가지 배포 전략을 고려할 수 있습니다. 아래에는 MCP 서버를 로컬, 컨테이너, 클라우드에 배포하는 방법에 대한 안내가 있습니다.

## 개요

이 강의는 MCP 서버 앱을 배포하는 방법을 다룹니다.

## 학습 목표

이 강의를 마치면 다음을 할 수 있습니다:

- 다양한 배포 방식을 평가할 수 있습니다.
- 앱을 배포할 수 있습니다.

## 로컬 개발 및 배포

서버가 사용자 기기에서 실행되어 사용되도록 설계된 경우, 다음 단계를 따르세요:

1. **서버 다운로드**. 서버를 직접 작성하지 않았다면, 먼저 서버를 기기에 다운로드하세요.
1. **서버 프로세스 시작**: MCP 서버 애플리케이션을 실행하세요.

SSE의 경우 (stdio 유형 서버에는 필요 없음)

1. **네트워킹 구성**: 서버가 예상된 포트에서 접근 가능하도록 설정하세요.
1. **클라이언트 연결**: `http://localhost:3000` 같은 로컬 연결 URL을 사용하세요.

## 클라우드 배포

MCP 서버는 다양한 클라우드 플랫폼에 배포할 수 있습니다:

- **서버리스 함수**: 가벼운 MCP 서버를 서버리스 함수로 배포
- **컨테이너 서비스**: Azure Container Apps, AWS ECS, Google Cloud Run 등의 서비스를 사용
- **쿠버네티스**: 고가용성을 위해 쿠버네티스 클러스터에서 MCP 서버를 배포 및 관리

### 예시: Azure Container Apps

Azure Container Apps는 MCP 서버 배포를 지원합니다. 현재 개발 중이며 SSE 서버만 지원합니다.

다음과 같이 진행할 수 있습니다:

1. 저장소를 클론합니다:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. 로컬에서 실행하여 테스트합니다:

  ```sh
  uv venv
  uv sync

  # 리눅스/macOS
  export API_KEYS=<AN_API_KEY>
  # 윈도우즈
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. 로컬에서 테스트하려면 *.vscode* 디렉토리에 *mcp.json* 파일을 만들고 다음 내용을 추가하세요:

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

  SSE 서버가 시작되면 JSON 파일 내 실행 아이콘을 클릭할 수 있으며, GitHub Copilot이 서버의 도구들을 감지하는 것을 볼 수 있습니다. 도구 아이콘을 확인하세요.

1. 배포하려면 다음 명령어를 실행하세요:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

이렇게 해서 로컬에서 배포하고, 위 절차를 통해 Azure에 배포할 수 있습니다.

## 추가 자료

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Azure Container Apps 기사](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Azure Container Apps MCP 저장소](https://github.com/anthonychu/azure-container-apps-mcp-sample)


## 다음 단계

- 다음: [고급 서버 주제](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**면책 조항**:  
이 문서는 AI 번역 서비스 [Co-op Translator](https://github.com/Azure/co-op-translator)를 사용하여 번역되었습니다. 정확성을 위해 최선을 다하고 있으나, 자동 번역에는 오류나 부정확한 내용이 포함될 수 있음을 유의하시기 바랍니다. 원본 문서는 해당 언어의 권위 있는 자료로 간주되어야 합니다. 중요한 정보에 대해서는 전문 인간 번역을 권장합니다. 본 번역 사용으로 인한 오해나 잘못된 해석에 대해 당사는 책임을 지지 않습니다.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->