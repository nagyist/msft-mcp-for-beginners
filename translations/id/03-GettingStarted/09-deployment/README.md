# Menyebarkan Server MCP

Menyebarkan server MCP Anda memungkinkan orang lain mengakses alat dan sumber dayanya di luar lingkungan lokal Anda. Ada beberapa strategi penyebaran yang harus dipertimbangkan, tergantung pada kebutuhan Anda untuk skalabilitas, keandalan, dan kemudahan pengelolaan. Di bawah ini Anda akan menemukan panduan untuk menyebarkan server MCP secara lokal, di kontainer, dan ke cloud.

## Ikhtisar

Pelajaran ini membahas cara menyebarkan aplikasi Server MCP Anda.

## Tujuan Pembelajaran

Pada akhir pelajaran ini, Anda akan dapat:

- Mengevaluasi berbagai pendekatan penyebaran.
- Menyebarkan aplikasi Anda.

## Pengembangan dan Penyebaran Lokal

Jika server Anda dimaksudkan untuk digunakan dengan menjalankan di mesin pengguna, Anda dapat mengikuti langkah-langkah berikut:

1. **Unduh server**. Jika Anda tidak menulis server, unduh terlebih dahulu ke mesin Anda.
1. **Mulai proses server**: Jalankan aplikasi server MCP Anda

Untuk SSE (tidak diperlukan untuk server tipe stdio)

1. **Konfigurasikan jaringan**: Pastikan server dapat diakses pada port yang diharapkan
1. **Hubungkan klien**: Gunakan URL koneksi lokal seperti `http://localhost:3000`

## Penyebaran Cloud

Server MCP dapat disebarkan ke berbagai platform cloud:

- **Fungsi Tanpa Server (Serverless Functions)**: Sebarkan server MCP ringan sebagai fungsi tanpa server
- **Layanan Kontainer**: Gunakan layanan seperti Azure Container Apps, AWS ECS, atau Google Cloud Run
- **Kubernetes**: Sebarkan dan kelola server MCP di kluster Kubernetes untuk ketersediaan tinggi

### Contoh: Azure Container Apps

Azure Container Apps mendukung penyebaran Server MCP. Ini masih dalam tahap pengembangan dan saat ini mendukung server SSE.

Berikut cara melakukannya:

1. Clone repositori:

  ```sh
  git clone https://github.com/anthonychu/azure-container-apps-mcp-sample.git
  ```

1. Jalankan secara lokal untuk menguji:

  ```sh
  uv venv
  uv sync

  # linux/macOS
  export API_KEYS=<AN_API_KEY>
  # windows
  set API_KEYS=<AN_API_KEY>

  uv run fastapi dev main.py
  ```

1. Untuk mencobanya secara lokal, buat file *mcp.json* di direktori *.vscode* dan tambahkan konten berikut:

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

  Setelah server SSE dimulai, Anda dapat mengklik ikon play di file JSON, Anda sekarang harus dapat melihat alat pada server yang dikenali oleh GitHub Copilot, lihat ikon Alat.

1. Untuk menyebarkan, jalankan perintah berikut:

  ```sh
  az containerapp up -g <RESOURCE_GROUP_NAME> -n weather-mcp --environment mcp -l westus --env-vars API_KEYS=<AN_API_KEY> --source .
  ```

Itulah cara Anda, menyebarkan secara lokal, menyebarkan ke Azure melalui langkah-langkah ini.

## Sumber Daya Tambahan

- [Azure Functions + MCP](https://learn.microsoft.com/en-us/samples/azure-samples/remote-mcp-functions-dotnet/remote-mcp-functions-dotnet/)
- [Artikel Azure Container Apps](https://techcommunity.microsoft.com/blog/appsonazureblog/host-remote-mcp-servers-in-azure-container-apps/4403550)
- [Repositori Azure Container Apps MCP](https://github.com/anthonychu/azure-container-apps-mcp-sample)

## Apa Selanjutnya

- Berikutnya: [Topik Server Lanjutan](../10-advanced/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Penafian**:  
Dokumen ini telah diterjemahkan menggunakan layanan terjemahan AI [Co-op Translator](https://github.com/Azure/co-op-translator). Meskipun kami berupaya untuk menjaga akurasi, harap diingat bahwa terjemahan otomatis mungkin mengandung kesalahan atau ketidakakuratan. Dokumen asli dalam bahasa aslinya harus dianggap sebagai sumber otoritatif. Untuk informasi yang sangat penting, disarankan menggunakan terjemahan profesional oleh manusia. Kami tidak bertanggung jawab atas kesalahpahaman atau penafsiran yang salah yang timbul dari penggunaan terjemahan ini.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->