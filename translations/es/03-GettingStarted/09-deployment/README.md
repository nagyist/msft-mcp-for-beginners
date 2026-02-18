# Desplegando Servidores MCP

Desplegar tu servidor MCP permite que otros accedan a sus herramientas y recursos más allá de tu entorno local. Existen varias estrategias de despliegue a considerar, dependiendo de tus requisitos de escalabilidad, confiabilidad y facilidad de gestión. A continuación encontrarás orientación para desplegar servidores MCP localmente, en contenedores y en la nube.

## Resumen

Esta lección cubre cómo desplegar tu aplicación MCP Server.

## Objetivos de aprendizaje

Al final de esta lección, podrás:

- Evaluar diferentes enfoques de despliegue.
- Desplegar tu aplicación.

## Desarrollo y despliegue local

Si tu servidor está pensado para ser consumido ejecutándose en la máquina del usuario, puedes seguir los siguientes pasos:

1. **Descargar el servidor**. Si no escribiste el servidor, primero descárgalo a tu máquina.  
1. **Iniciar el proceso del servidor**: Ejecuta tu aplicación MCP server.

Para SSE (no necesario para servidores tipo stdio)

1. **Configurar la red**: Asegúrate de que el servidor sea accesible en el puerto esperado.  
1. **Conectar clientes**: Usa URLs de conexión local como `http://localhost:3000`.

## Despliegue en la nube

Los servidores MCP pueden desplegarse en varias plataformas en la nube:

- **Funciones sin servidor (Serverless Functions)**: Despliega servidores MCP livianos como funciones sin servidor.  
- **Servicios de Contenedores**: Usa servicios como Azure Container Apps, AWS ECS, o Google Cloud Run.  
- **Kubernetes**: Despliega y gestiona servidores MCP en clusters de Kubernetes para alta disponibilidad.

### Ejemplo: Azure Container Apps

Azure Container Apps soporta el despliegue de Servidores MCP. Aún está en desarrollo y actualmente soporta servidores SSE.

Así puedes hacerlo:

1. Clona un repositorio:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. Ejecútalo localmente para probar:

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. Para probarlo localmente, crea un archivo *mcp.json* en un directorio *.vscode* y agrega el siguiente contenido:

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

  Una vez que el servidor SSE esté iniciado, puedes hacer clic en el ícono de reproducción en el archivo JSON, deberías ver ahora las herramientas del servidor ser detectadas por GitHub Copilot, mira el ícono de herramienta.

1. Para desplegar, ejecuta el siguiente comando:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

Ahí lo tienes, desplégalo localmente o a Azure siguiendo estos pasos.

## Recursos adicionales

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Artículo sobre Azure Container Apps](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Repositorio MCP para Azure Container Apps](https://github.com/anthonychu/azure-container-apps-mcp-sample)

## Qué sigue

- Siguiente: [Temas avanzados de servidor](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Descargo de responsabilidad**:  
Este documento ha sido traducido utilizando el servicio de traducción automática [Co-op Translator](https://github.com/Azure/co-op-translator). Aunque nos esforzamos por la precisión, tenga en cuenta que las traducciones automáticas pueden contener errores o inexactitudes. El documento original en su idioma nativo debe considerarse la fuente autorizada. Para información crítica, se recomienda una traducción profesional realizada por un humano. No nos hacemos responsables de ningún malentendido o interpretación errónea que surja del uso de esta traducción.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->