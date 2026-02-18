# Deployment de servidores MCP

Fazer o deployment do seu servidor MCP permite que outras pessoas acedam às suas ferramentas e recursos para além do seu ambiente local. Existem várias estratégias de deployment a considerar, dependendo dos seus requisitos para escalabilidade, fiabilidade e facilidade de gestão. Em baixo encontrará orientações para fazer deployment de servidores MCP localmente, em contentores e na cloud.

## Visão geral

Esta lição cobre como fazer o deployment da sua aplicação de servidor MCP.

## Objetivos de aprendizagem

No final desta lição, será capaz de:

- Avaliar diferentes abordagens de deployment.
- Fazer o deployment da sua aplicação.

## Desenvolvimento e deployment local

Se o seu servidor se destina a ser utilizado correndo na máquina dos utilizadores, pode seguir os seguintes passos:

1. **Descarregar o servidor**. Se não escreveu o servidor, descarregue-o primeiro para a sua máquina.
1. **Iniciar o processo do servidor**: Execute a sua aplicação de servidor MCP.

Para SSE (não necessário para servidores do tipo stdio)

1. **Configurar a rede**: Assegure que o servidor é acessível na porta esperada.
1. **Ligar clientes**: Use URLs de ligação local como `http://localhost:3000`.

## Deployment na cloud

Os servidores MCP podem ser deployados em várias plataformas cloud:

- **Funções Serverless**: Faça o deployment de servidores MCP leves como funções serverless.
- **Serviços de Contentores**: Utilize serviços como Azure Container Apps, AWS ECS ou Google Cloud Run.
- **Kubernetes**: Faça deployment e gestão de servidores MCP em clusters Kubernetes para alta disponibilidade.

### Exemplo: Azure Container Apps

Azure Container Apps suporta o deployment de Servidores MCP. Está ainda em desenvolvimento e atualmente suporta servidores SSE.

Aqui está como pode fazê-lo:

1. Clone um repositório:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. Execute localmente para testar:

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. Para testar localmente, crie um ficheiro *mcp.json* numa diretoria *.vscode* e adicione o seguinte conteúdo:

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

  Depois de iniciar o servidor SSE, pode clicar no ícone play no ficheiro JSON, deverá agora ver as ferramentas no servidor serem reconhecidas pelo GitHub Copilot, veja o ícone da ferramenta.

1. Para fazer o deployment, corra o seguinte comando:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

Aqui está, faça o deployment localmente, faça o deployment para Azure através destes passos.

## Recursos adicionais

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Artigo Azure Container Apps](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Repositório MCP para Azure Container Apps](https://github.com/anthonychu/azure-container-apps-mcp-sample)

## O que vem a seguir

- Seguinte: [Tópicos avançados de servidor](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Aviso Legal**:  
Este documento foi traduzido utilizando o serviço de tradução automática [Co-op Translator](https://github.com/Azure/co-op-translator). Embora nos esforcemos pela precisão, esteja ciente de que traduções automáticas podem conter erros ou imprecisões. O documento original, na sua língua nativa, deve ser considerado a fonte autoritativa. Para informações críticas, recomenda-se uma tradução profissional humana. Não nos responsabilizamos por quaisquer mal-entendidos ou interpretações incorretas decorrentes da utilização desta tradução.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->