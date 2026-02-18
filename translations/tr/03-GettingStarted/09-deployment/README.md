# MCP Sunucularını Yayınlama

MCP sunucunuzu yayınlamak, başkalarının araçlarına ve kaynaklarına yerel ortamınızın ötesinde erişmesini sağlar. Ölçeklenebilirlik, güvenilirlik ve yönetim kolaylığı gereksinimlerinize bağlı olarak dikkate alınması gereken birkaç dağıtım stratejisi vardır. Aşağıda MCP sunucularını yerel olarak, konteynerlerde ve buluta dağıtmak için rehber bulacaksınız.

## Genel Bakış

Bu ders, MCP Server uygulamanızın nasıl dağıtılacağını kapsar.

## Öğrenme Hedefleri

Bu dersin sonunda şunları yapabileceksiniz:

- Farklı dağıtım yaklaşımlarını değerlendirmek.
- Uygulamanızı dağıtmak.

## Yerel geliştirme ve dağıtım

Sunucunuzun kullanıcıların makinelerinde çalıştırılarak tüketilmesi amaçlanıyorsa, aşağıdaki adımları takip edebilirsiniz:

1. **Sunucuyu indirin**. Sunucuyu siz yazmadıysanız, önce makinenize indirin.  
1. **Sunucu sürecini başlatın**: MCP sunucu uygulamanızı çalıştırın.

SSE için (stdio türü sunucu için gerekli değildir)

1. **Ağ yapılandırmasını yapın**: Sunucunun beklenen bağlantı noktasından erişilebilir olmasını sağlayın.  
1. **İstemcilere bağlanın**: `http://localhost:3000` gibi yerel bağlantı URL’lerini kullanın.

## Bulut Dağıtımı

MCP sunucuları çeşitli bulut platformlarına dağıtılabilir:

- **Sunucusuz Fonksiyonlar**: Hafif MCP sunucuları sunucusuz fonksiyonlar olarak dağıtın.  
- **Konteyner Servisleri**: Azure Container Apps, AWS ECS veya Google Cloud Run gibi servisleri kullanın.  
- **Kubernetes**: Yüksek kullanılabilirlik için MCP sunucularını Kubernetes kümelerinde dağıtın ve yönetin.

### Örnek: Azure Container Apps

Azure Container Apps, MCP Sunucularının dağıtımını destekler. Henüz geliştirme aşamasındadır ve şu anda SSE sunucularını desteklemektedir.

İşte nasıl yapabileceğiniz:

1. Bir repo klonlayın:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. Denemek için yerelde çalıştırın:

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. Yerelde denemek için, *.vscode* dizininde *mcp.json* dosyası oluşturun ve aşağıdaki içeriği ekleyin:

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

  SSE sunucusu başlatıldığında, JSON dosyasındaki oynat simgesine tıklayabilirsiniz; artık GitHub Copilot tarafından sunucudaki araçlar algılanacak, Araç simgesini görün. 

1. Dağıtmak için aşağıdaki komutu çalıştırın:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

İşte bu kadar, yerelde veya bu adımlarla Azure’a dağıtabilirsiniz.

## Ek Kaynaklar

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Azure Container Apps makalesi](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Azure Container Apps MCP reposu](https://github.com/anthonychu/azure-container-apps-mcp-sample)


## Sonraki Ne Var

- Sonraki: [Gelişmiş Sunucu Konuları](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Feragatname**:
Bu belge, AI çeviri hizmeti [Co-op Translator](https://github.com/Azure/co-op-translator) kullanılarak çevrilmiştir. Doğruluk için çaba göstermemize rağmen, otomatik çeviriler hata veya yanlışlıklar içerebilir. Orijinal belge, kendi ana dilinde yetkili kaynak olarak kabul edilmelidir. Kritik bilgiler için profesyonel insan çevirisi önerilir. Bu çevirinin kullanımı sonucunda oluşabilecek yanlış anlamalar veya yorum hatalarından sorumlu değiliz.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->