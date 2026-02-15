# 免費/低成本 LLM API 方案整合 OpenClaw 指南

## 概述

OpenClaw 是一個強大的 AI agent 框架，支援 OpenAI-compatible API endpoints。本文檔詳細評估了所有可免費或低成本使用的 LLM API 服務，特別確認其與 OpenClaw 的相容性。

## ⚠️ Antigravity Auth 警告

**強烈不建議使用 Antigravity Auth！**

Antigravity Auth 濫用 Google Cloud Code Assist 的免費方案，這會導致：
- **永久停用**：Google 帳號的 Gemini 功能可能被永久停用
- **違反服務條款**：未經授權使用企業級服務
- **帳號風險**：可能影響整個 Google 帳號
- **不穩定性**：服務隨時可能中斷

請使用本文檔推薦的正當方案替代。

## 完整方案比較表

| 服務 | 類型 | 免費額度 | Tool Calling 支援 | 模型品質 | 註冊門檻 | 推薦度 |
|------|------|----------|------------------|----------|----------|--------|
| **Groq** | 完全免費 | 200K token/天 | ✅ 完整支援 | ⭐⭐⭐⭐ | 低 | ⭐⭐⭐⭐⭐ |
| **HuggingFace** | 完全免費 | 每日限制 | ✅ 透過 API | ⭐⭐⭐⭐ | 低 | ⭐⭐⭐⭐⭐ |
| **Cloudflare Workers AI** | 完全免費 | 10K neurons/天 | ⚠️ 部分模型 | ⭐⭐⭐ | 低 | ⭐⭐⭐ |
| **DeepSeek API** | 超低價 | $0.14/$1.10 | ✅ 支援 | ⭐⭐⭐⭐⭐ | 低 | ⭐⭐⭐⭐⭐ |
| **Together.ai** | 超低價 | $5 免費額度 | ✅ 支援 | ⭐⭐⭐⭐ | 低 | ⭐⭐⭐⭐ |
| **Fireworks.ai** | 超低價 | $1 免費額度 | ✅ 支援 | ⭐⭐⭐⭐ | 低 | ⭐⭐⭐⭐ |
| **OpenRouter** | 超低價 | $1 免費額度 | ✅ 支援 | ⭐⭐⭐⭐ | 低 | ⭐⭐⭐⭐ |
| **Ollama** | 本地免費 | 無限制 | ✅ 支援 | ⭐⭐⭐⭐ | 硬體需求 | ⭐⭐⭐ |

## 1. 完全免費方案

### 🔥 Groq (強烈推薦)

**特色**：極快推理速度，優秀的免費額度

- **免費額度**：200K tokens/天，分模型限制
- **定價**：
  - Llama 3.1 8B: $0.05 input / $0.08 output (per 1M tokens)
  - Llama 3.3 70B: $0.59 input / $0.79 output
  - GPT OSS 120B: $0.15 input / $0.60 output
- **Tool Calling**：✅ 完整支援，包含 parallel tool use
- **速度**：560-1000 T/sec（業界領先）
- **模型品質**：Llama 3.3 70B 表現優異
- **註冊**：簡單，只需 email

**OpenClaw 設定**：
```yaml
model_provider: groq
api_key: your_groq_api_key
base_url: https://api.groq.com/openai/v1
model: llama-3.3-70b-versatile
```

### 🤗 HuggingFace Inference API

**特色**：模型種類最豐富，支援多種 providers

- **免費額度**：每日限制（具體數量視模型而定）
- **定價**：透過多個 provider 提供不同價格
- **Tool Calling**：✅ 透過 OpenAI-compatible API
- **模型選擇**：數千種模型可選
- **特色**：自動 provider 選擇（fastest/cheapest/preferred）

**OpenClaw 設定**：
```yaml
model_provider: openai
api_key: your_huggingface_token
base_url: https://router.huggingface.co/v1
model: openai/gpt-oss-120b:fastest
```

### ☁️ Cloudflare Workers AI

**特色**：與 Cloudflare 生態系整合

- **免費額度**：10,000 Neurons/天
- **定價**：$0.011 / 1,000 Neurons（超過免費額度後）
- **Tool Calling**：⚠️ 僅部分模型支援
- **模型選擇**：Llama, Mistral, Qwen 等
- **限制**：tool calling 支援有限

## 2. 超低價方案（月費 < $5）

### 🚀 DeepSeek API (最佳性價比)

**特色**：業界最便宜的高品質 API

- **定價**：
  - DeepSeek V3.1: $0.60 input / $1.70 output (per 1M tokens)
  - DeepSeek Chat: $0.14 input / $1.10 output
- **Tool Calling**：✅ 完整支援
- **模型品質**：⭐⭐⭐⭐⭐ 頂級表現
- **OpenAI 相容**：完全相容

**OpenClaw 設定**：
```yaml
model_provider: openai
api_key: your_deepseek_api_key
base_url: https://api.deepseek.com
model: deepseek-chat
```

### 🔥 Together.ai

**特色**：模型選擇豐富，價格合理

- **免費額度**：$5 免費 credits
- **熱門模型定價**：
  - Llama 3.3 70B: $0.88 input/output
  - Llama 3.1 8B: $0.18 input/output
  - DeepSeek V3.1: $0.60 input / $1.70 output
- **Tool Calling**：✅ 支援
- **特色**：支援最新模型，包括 Llama 4

### ⚡ Fireworks.ai

**特色**：高速推理，企業級可靠性

- **免費額度**：$1 免費 credits
- **定價結構**：
  - <4B 參數：$0.10/1M tokens
  - 4B-16B 參數：$0.20/1M tokens
  - >16B 參數：$0.90/1M tokens
- **Tool Calling**：✅ 完整支援
- **特色**：快速推理，批次推理 50% 折扣

### 🌐 OpenRouter

**特色**：統一介面訪問多個 provider

- **免費額度**：$1 免費 credits
- **定價**：透明的統一定價，無加成
- **Tool Calling**：✅ 支援
- **特色**：模型比較方便，統一計費

## 3. 本地方案

### 🖥️ Ollama

**特色**：完全本地運行，無隱私顧慮

- **成本**：完全免費（僅硬體成本）
- **Tool Calling**：✅ 支援（透過 OpenAI-compatible API）
- **硬體需求**：
  - **最低配置**：8GB RAM，CPU 推理（7B 模型）
  - **推薦配置**：16GB+ RAM，RTX 4070 或以上
  - **高性能**：32GB+ RAM，RTX 4090 / A100
- **支援模型**：Llama 3, Qwen, Mistral, CodeLlama 等

**硬體需求詳細**：

| 模型大小 | RAM 需求 | GPU 需求 | 推理速度 |
|----------|----------|----------|----------|
| 7B | 8GB | CPU/集顯 | 5-15 tokens/s |
| 13B | 16GB | RTX 3060+ | 10-30 tokens/s |
| 70B | 64GB+ | RTX 4090+ | 2-10 tokens/s |

**安裝設定**：
```bash
# macOS
curl -fsSL https://ollama.com/install.sh | sh

# 拉取模型
ollama pull llama3.1:8b

# 啟用 OpenAI API 相容模式
export OLLAMA_ORIGINS="*"
ollama serve
```

**OpenClaw 設定**：
```yaml
model_provider: openai
api_key: ollama  # 可以是任意值
base_url: http://localhost:11434/v1
model: llama3.1:8b
```

## 🏆 Top 3 推薦方案

### 1. 🥇 Groq (最佳免費選擇)
- **理由**：免費額度慷慨，速度極快，tool calling 完整
- **適合**：日常開發、原型驗證、中小型應用
- **月成本**：$0
- **Token 限制**：200K/天，多數應用足夠

### 2. 🥈 DeepSeek API (最佳性價比)
- **理由**：價格極低，模型品質頂級，無使用限制
- **適合**：生產應用、大量使用、商業部署
- **月成本**：$1-5（視使用量）
- **特色**：R1 模型推理能力強

### 3. 🥉 HuggingFace Inference API (最佳彈性)
- **理由**：模型選擇最豐富，可自動選擇最佳 provider
- **適合**：需要多種模型、實驗性項目
- **月成本**：$0-5（視使用量和模型選擇）
- **特色**：統一介面訪問多個 provider

## OpenClaw 設定範例

### 基本設定檔 (config.yaml)

```yaml
# Groq 設定（推薦新手）
model_provider: groq
api_key: ${GROQ_API_KEY}
base_url: https://api.groq.com/openai/v1
model: llama-3.3-70b-versatile

# DeepSeek 設定（推薦生產）
# model_provider: openai
# api_key: ${DEEPSEEK_API_KEY}
# base_url: https://api.deepseek.com
# model: deepseek-chat

# HuggingFace 設定（推薦實驗）
# model_provider: openai
# api_key: ${HF_TOKEN}
# base_url: https://router.huggingface.co/v1
# model: openai/gpt-oss-120b:fastest

# Ollama 本地設定
# model_provider: openai
# api_key: ollama
# base_url: http://localhost:11434/v1
# model: llama3.1:8b
```

### 環境變數設定

```bash
# .env 檔案
GROQ_API_KEY=your_groq_api_key_here
DEEPSEEK_API_KEY=your_deepseek_api_key_here
HF_TOKEN=your_huggingface_token_here
```

### 多 Provider 切換腳本

```bash
#!/bin/bash
# switch-provider.sh

case $1 in
  "groq")
    export OPENCLAW_PROVIDER="groq"
    export OPENCLAW_API_KEY=$GROQ_API_KEY
    export OPENCLAW_BASE_URL="https://api.groq.com/openai/v1"
    export OPENCLAW_MODEL="llama-3.3-70b-versatile"
    echo "已切換到 Groq"
    ;;
  "deepseek")
    export OPENCLAW_PROVIDER="openai"
    export OPENCLAW_API_KEY=$DEEPSEEK_API_KEY
    export OPENCLAW_BASE_URL="https://api.deepseek.com"
    export OPENCLAW_MODEL="deepseek-chat"
    echo "已切換到 DeepSeek"
    ;;
  "local")
    export OPENCLAW_PROVIDER="openai"
    export OPENCLAW_API_KEY="ollama"
    export OPENCLAW_BASE_URL="http://localhost:11434/v1"
    export OPENCLAW_MODEL="llama3.1:8b"
    echo "已切換到本地 Ollama"
    ;;
esac
```

## 功能支援比較

### Tool/Function Calling 測試

所有推薦方案都支援 OpenClaw 所需的 tool calling 功能：

```json
{
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "get_weather",
        "description": "獲取指定城市的天氣資訊",
        "parameters": {
          "type": "object",
          "properties": {
            "city": {
              "type": "string",
              "description": "城市名稱"
            }
          },
          "required": ["city"]
        }
      }
    }
  ]
}
```

### 速率限制比較

| 服務 | 每分鐘請求數 | 每分鐘 Token 數 | 並發限制 |
|------|-------------|-----------------|----------|
| Groq | 1000 | 200K-300K | 好 |
| DeepSeek | 無明確限制 | 取決於付費等級 | 優秀 |
| Together.ai | 1000 | 取決於模型 | 好 |
| HuggingFace | 取決於 provider | 取決於 provider | 中等 |
| Ollama | 無限制 | 無限制 | 取決於硬體 |

## 最佳實踐建議

### 1. 開發階段
- **使用 Groq**：免費額度足夠，速度快，適合快速迭代
- **本地測試**：使用 Ollama 進行離線開發

### 2. 生產部署
- **主要服務**：DeepSeek API（成本最低，品質最高）
- **備用服務**：Together.ai 或 Fireworks.ai
- **高可用性**：設定多個 provider 自動切換

### 3. 成本控制
- **監控使用量**：設定 token 使用上限
- **選擇合適模型**：不需要最大模型時使用較小模型
- **批次處理**：盡可能批次處理請求

### 4. 安全考慮
- **API 金鑰管理**：使用環境變數，不要硬編碼
- **請求記錄**：記錄 API 使用量以便監控
- **錯誤處理**：實現適當的重試機制

## 故障排除

### 常見問題

1. **Tool Calling 不工作**
   ```bash
   # 檢查模型是否支援 function calling
   curl -X POST "https://api.groq.com/openai/v1/chat/completions" \
     -H "Authorization: Bearer $GROQ_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{"model": "llama-3.3-70b-versatile", "messages": [{"role": "user", "content": "test"}], "tools": [...]}'
   ```

2. **API 金鑰問題**
   ```bash
   # 測試 API 金鑰有效性
   curl -H "Authorization: Bearer $YOUR_API_KEY" https://api.groq.com/openai/v1/models
   ```

3. **速率限制**
   - 實現指數退避重試
   - 分散請求時間
   - 考慮升級到付費計劃

## 結論

對於 OpenClaw agent 來說，**Groq** 是最佳的免費起點，**DeepSeek API** 是最佳的生產選擇。建議：

1. **新手/開發**：從 Groq 開始，熟悉 OpenClaw 的使用
2. **小型項目**：使用 HuggingFace 的多 provider 選擇
3. **生產應用**：DeepSeek API + 備用 provider
4. **隱私要求**：Ollama 本地部署
5. **避免使用**：Antigravity Auth 等非正當方案

所有推薦方案都完全支援 OpenClaw 所需的 function/tool calling 能力，可以放心使用。

---

**最後更新**：2026-02-15  
**作者**：Jarvis AI Assistant  
**版本**：v1.0