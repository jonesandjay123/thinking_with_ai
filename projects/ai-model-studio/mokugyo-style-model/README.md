# Mokugyo Style Model

> 狀態：概念與資料策略  
> 日期：2026-05-12

## 一句話

Mokugyo Style Model 是 Jones 曾經想把三千多篇 Facebook 貼文餵給「默行」，讓它模仿自己文字風格、協助寫日誌與文章的長期想法。

## 核心目標

建立一個能理解 Jones 個人文風、語氣、觀察方式與思想脈絡的寫作輔助系統。

它不是一般聊天機器人，而是個人表達模型：

- 保留 Jones 的語氣
- 延伸 Jones 的觀察
- 協助整理日誌
- 協助把生活體驗轉成文章
- 協助把 GPT / Jarvis 對話轉成可讀作品

## 第一版不從零訓練模型

不建議一開始從零訓練大模型。

更合理路線：

1. 匯出與整理歷史貼文。
2. 清理隱私與敏感資訊。
3. 建立文章 metadata：日期、主題、語氣、事件、人物、地點。
4. 建立向量資料庫。
5. 做 style retrieval：寫作前先找相似舊文。
6. 建立 style guide。
7. 用 RAG + prompt imitation 先達到 70–80% 效果。
8. 再評估 LoRA / fine-tuning。

## 資料風險

這條線牽涉大量私人資料，因此必須保守處理：

- 不把原始資料上傳不可信平台。
- 先在本地整理。
- 明確標記敏感人物、私人事件與不可外傳內容。
- 輸出前要有人工審稿。

## 與 Thought Atlas / Observer J 的關係

Mokugyo Style Model 可以成為：

- Thought Atlas 的個人表達層
- Observer J 的寫作人格基底
- Jones 長期日誌 / essay / creative nonfiction 的輔助工具
- 將生活體驗轉為可發佈作品的 pipeline

## 成功標準

第一階段成功不看「像不像 100%」，而看：

- 能否找到相似舊文
- 能否整理出 Jones 常用語氣與主題
- 能否產生需要少量修改即可使用的日誌草稿
- 能否協助把長對話轉成有 Jones 味道的文章
- 是否保護隱私

## 未來延伸

- LoRA fine-tuning
- 個人文風 benchmark
- 日誌自動整理
- 與 Thought Atlas ingest pipeline 串接
- 多語言風格轉換：繁中 / 英文 / 日文
