# Triển khai Máy chủ MCP

Triển khai máy chủ MCP của bạn cho phép người khác truy cập các công cụ và tài nguyên của nó ngoài môi trường cục bộ của bạn. Có nhiều chiến lược triển khai khác nhau để xem xét, tùy thuộc vào yêu cầu về khả năng mở rộng, độ tin cậy và sự dễ dàng trong quản lý. Dưới đây, bạn sẽ tìm thấy hướng dẫn để triển khai máy chủ MCP cục bộ, trong container và trên đám mây.

## Tổng quan

Bài học này bao gồm cách triển khai ứng dụng Máy chủ MCP của bạn.

## Mục tiêu học tập

Vào cuối bài học này, bạn sẽ có khả năng:

- Đánh giá các phương pháp triển khai khác nhau.
- Triển khai ứng dụng của bạn.

## Phát triển và triển khai cục bộ

Nếu máy chủ của bạn dự định được sử dụng bởi việc chạy trên máy người dùng, bạn có thể làm theo các bước sau:

1. **Tải xuống máy chủ**. Nếu bạn không viết máy chủ, hãy tải nó xuống máy của bạn trước.  
1. **Khởi động tiến trình máy chủ**: Chạy ứng dụng máy chủ MCP của bạn.

Đối với SSE (không cần thiết cho máy chủ loại stdio)

1. **Cấu hình mạng**: Đảm bảo máy chủ có thể truy cập được trên cổng dự kiến.  
1. **Kết nối các client**: Sử dụng URL kết nối cục bộ như `http://localhost:3000`

## Triển khai trên đám mây

Máy chủ MCP có thể được triển khai trên các nền tảng đám mây khác nhau:

- **Hàm không máy chủ (Serverless Functions)**: Triển khai máy chủ MCP nhẹ dưới dạng các hàm không máy chủ  
- **Dịch vụ Container**: Sử dụng các dịch vụ như Azure Container Apps, AWS ECS, hoặc Google Cloud Run  
- **Kubernetes**: Triển khai và quản lý máy chủ MCP trong các cụm Kubernetes để có độ khả dụng cao

### Ví dụ: Azure Container Apps

Azure Container Apps hỗ trợ triển khai Máy chủ MCP. Đây vẫn là một dự án đang phát triển và hiện tại nó hỗ trợ các máy chủ SSE.

Dưới đây là cách bạn có thể làm:

1. Sao chép một repo:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. Chạy nó cục bộ để thử nghiệm:

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. Để thử nghiệm cục bộ, tạo một tệp *mcp.json* trong thư mục *.vscode* và thêm nội dung sau:

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

  Khi máy chủ SSE được khởi động, bạn có thể nhấn biểu tượng chạy trong tệp JSON, bạn sẽ thấy các công cụ trên máy chủ được GitHub Copilot phát hiện, xem biểu tượng Công cụ.

1. Để triển khai, chạy lệnh sau:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

Vậy là xong, triển khai cục bộ, triển khai lên Azure thông qua các bước này.

## Tài nguyên bổ sung

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Bài viết Azure Container Apps](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Repo Azure Container Apps MCP](https://github.com/anthonychu/azure-container-apps-mcp-sample)

## Tiếp theo

- Tiếp theo: [Chủ đề máy chủ nâng cao](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Miễn trừ trách nhiệm**:  
Tài liệu này đã được dịch bằng dịch vụ dịch thuật AI [Co-op Translator](https://github.com/Azure/co-op-translator). Mặc dù chúng tôi cố gắng đảm bảo độ chính xác, xin lưu ý rằng các bản dịch tự động có thể chứa lỗi hoặc không chính xác. Tài liệu gốc bằng ngôn ngữ ban đầu được xem là nguồn tham khảo chính thức. Đối với thông tin quan trọng, nên sử dụng dịch vụ dịch thuật chuyên nghiệp do con người thực hiện. Chúng tôi không chịu trách nhiệm về bất kỳ sự hiểu lầm hoặc giải thích sai nào phát sinh từ việc sử dụng bản dịch này.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->