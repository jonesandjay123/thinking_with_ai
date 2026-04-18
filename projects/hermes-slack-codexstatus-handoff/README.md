# Hermes Slack `codexstatus` 交接檔

> **建立日期：** 2026-04-17  
> **建立原因：** 這次 `codexstatus` 在 Hermes + Slack 上排查花了 3+ 小時，燒掉大量 usage，但仍未完成真正的 Slack 端低-token quick-command 體驗。這份文件是給 **新 session / 新接手 agent** 的完整交接檔，目標是讓接手者不用再讓 Jones 重講背景，就能直接進入有效排查。

---

## 一句話先講結論

**現在已經證明：**
- `codexstatus` 腳本本身可用
- Hermes config 已正確配置 quick command
- Slack adapter / active-session guard 的程式修補已存在
- 針對這些修補的 focused tests 已經通過
- gateway 目前跑的是修補後版本

**但尚未證明：**
- Slack 中的
  - `/hermes codexstatus`
  - `/jarvis codexstatus`
  - DM 直接打 `codexstatus`
  是否真的 **穩定繞過 LLM / agent loop**，直接回傳 quick-command 結果。

也就是說：

> **內部 patch 與測試現在看起來 OK，但使用者體感仍然是「Slack 這邊沒有真正成功」。**

這就是目前的核心問題。

---

## 問題定義

Jones 的目標不是「理論上 quick command 存在」，而是以下體驗必須成立：

1. 在 Slack 直接輸入 `/hermes codexstatus`
2. 或輸入 `/jarvis codexstatus`
3. 或在 Hermes Slack DM 直接輸入 `codexstatus`
4. Hermes 應該 **不走正常 LLM 對話 loop**
5. 應該 **低 token / 幾乎不耗 agent usage**
6. 直接執行：
   - `python3 ~/.hermes/scripts/codex_status.py`
7. 立刻回傳 Codex usage（5h / week window）

### 使用者實際感受

Jones 的實際感受是：
- Slack 端輸入上述命令時，**仍像進了 agent 對話模式**
- 回應像是 agent 自己在「找答案」
- 不像真正的 quick command
- 中間還多次發生 `Gateway shutting down` / restart
- 整體結果是 **燒掉大量 usage，卻沒有得到確定成功的體驗**

這份感受是合理且重要的：

> 這次的 bug 不是「repo 裡沒改到」，而是 **使用者端體感沒有被證明成功**。

---

## 對使用者來說，真正的問題點是什麼？

如果要用最精準的話描述：

### 問題不是資料來源
因為資料來源已經能工作：
- `~/.hermes/scripts/codex_status.py` 可以直接回傳 live usage

### 問題也不完全是 config
因為 config 已存在：
- `~/.hermes/config.yaml`
- `quick_commands.codexstatus -> python3 ~/.hermes/scripts/codex_status.py`

### 真正的問題是
**Slack 這一層進來的請求，是否真的在 runtime 中被當作 quick command 處理，而不是掉回一般 agent / LLM 流程。**

更具體說，剩下的懷疑集中在這幾段：

1. **Slack slash command parsing**
   - `/hermes codexstatus`
   - `/jarvis codexstatus`
   是否真的被 rewrite 成 `"/codexstatus"`

2. **DM text rewrite**
   - `codexstatus`（裸字）
   是否真的被轉成 `"/codexstatus"`

3. **active session guard**
   - 即使 thread / session 正在忙
   - `codexstatus` 是否仍能 bypass，不被丟回 pending / follow-up / normal chat

4. **thread/source/session key 行為**
   - slash command 或 thread reply 是否帶了與預期不同的 `thread_id`
   - 造成 quick-command branch 雖存在，但實際 runtime 路徑不一樣

5. **Slack 端實際送進 gateway 的 payload 形狀**
   - 真實 Slack 事件可能與單元測試假設不同
   - 尤其 thread、DM、slash command 三條路可能不完全等價

---

## 目前已知、已被工具驗證的事實

以下是本輪排查中，已被工具重新驗證過的事實。

### 1. repo 與 config 現況
`~/.hermes/config.yaml` 目前包含：

```yaml
quick_commands:
  codexstatus:
    type: exec
    command: python3 ~/.hermes/scripts/codex_status.py
```

### 2. `codex_status.py` 腳本可直接執行
本輪重新執行結果（2026-04-17 20:13 EDT 附近）：

```text
Codex usage (live)
plan:        plus ($0.00)
5h left:        23% (reset 04-17 23:16 EDT)
Week left:      65% (reset 04-22 21:30 EDT)
```

這代表：
- usage endpoint 與 auth 在本機是通的
- quick command 的 payload 來源不是問題

### 3. focused tests 已通過
執行命令：

```bash
source ~/.hermes/hermes-agent/venv/bin/activate
cd ~/.hermes/hermes-agent
pytest -q tests/gateway/test_slack.py -k 'quick_command or active_session_guard or jarvis'
```

結果：

```text
5 passed in 0.77s
```

### 4. gateway 目前正在跑
查到的 gateway process：

```text
PID 35416
Fri Apr 17 20:04:24 2026
/Users/jarvis/.hermes/hermes-agent/venv/bin/python -m hermes_cli.main gateway run --replace
```

### 5. 本輪 patch 的核心內容存在於 working tree
目前 `~/.hermes/hermes-agent` 是 **未提交變更** 狀態，重點檔案：
- `gateway/platforms/base.py`
- `gateway/platforms/slack.py`
- `tests/gateway/test_slack.py`

這很重要：

> 目前修補 **還在 working tree，不是已提交 canonical 狀態**。若之後被覆蓋、reset、切 branch，這些修改可能消失。

---

## 已做過的程式修補（重要）

### A. `gateway/platforms/base.py`
目標：
讓 quick commands 也能 bypass active-session guard。

修補前的問題：
- 只有少數內建 command（approve / deny / status / stop ...）會 bypass
- `codexstatus` 這種 config-based quick command **沒有進 bypass 白名單**
- 若該 DM/thread 已有 active session，可能被當作一般 follow-up 處理

修補後：
- 讀取 `quick_commands.keys()`
- 與內建 bypass commands 合併
- 若 `cmd in bypass_commands`，直接 dispatch

### B. `gateway/platforms/slack.py`
目標：
讓 Slack 端把 quick commands 正確 rewrite 成 command event。

已加入的邏輯包含：

#### 1. `/jarvis` 也註冊到同一個 slash handler
不只 `/hermes`，也明確註冊 `/jarvis`。

#### 2. Slack DM 裡裸字 quick command 自動 rewrite
例如：
- `codexstatus`

在 DM 會變成：
- `/codexstatus`

#### 3. slash command 會辨識 configured quick commands
例如：
- `/hermes codexstatus`
- `/jarvis codexstatus`

會在 `_handle_slash_command()` 中改寫成：
- `/codexstatus`

#### 4. slash command 保留 thread id
修補後把：
- `thread_ts` / `thread_id`
帶進 `build_source(...)`

#### 5. thread context prepend 避開 command-style message
避免 thread context prepend 把 `/codexstatus` 重新污染成普通文字

### C. `tests/gateway/test_slack.py`
已新增 / 修改的測試重點：

1. quick command 在 slash command 中會映射成 `/codexstatus`
2. quick command 的 slash path 會保留 `thread_id`
3. DM 裡裸字 `codexstatus` 會變成 command event
4. DM thread quick command 不應該被 thread context prepend 污染
5. quick command 應 bypass active-session guard
6. `/jarvis` handler 已註冊

---

## 為什麼 Jones 仍然覺得「沒有成功」？

因為從使用者體感來看，**成功的標準不是測試綠**，而是：

- 在 Slack 打 `/hermes codexstatus`
- 幾乎瞬間得到 quota 數字
- 看起來像 quick command / exec result
- 而不是像 agent 開一個新 session 去思考、搜尋、再回答

如果回應的呈現仍像下面這種：
- 顯示很多工具呼叫
- 像一般 agent message flow
- 需要進 session 才有回應
- 甚至出現「command not found」或狀況外回答

那對使用者來說，就是 **沒成功**。

這個體感判準是對的，不能用「測試過了」硬壓掉。

---

## 目前最可信的總結判斷

### 已證明的部分
這些基本上已經不是主因：
- `codexstatus` script 壞掉
- `quick_commands` config 沒設好
- `/jarvis` slash handler 完全沒接上
- active-session bypass 完全沒做
- focused tests 完全沒補
- gateway 完全沒重啟到新版本

### 尚未證明的部分
目前真正還缺的是：

> **Slack 真實事件進來時，runtime 是否真的走到 quick-command exec branch，而不是中途某處掉回普通 agent flow。**

也就是說，剩下要查的是 **runtime evidence**，不是再盲 patch。

---

## 最可能的剩餘問題假說

以下是假說，**不是已證明 root cause**。

### 假說 1：Slack 真實 payload 與測試假設不同
例如：
- slash command 真實送進來的 `text` 並非單純 `codexstatus`
- thread / channel metadata 與測試不一致
- 某條 path 在真實 Slack 會帶不同欄位

### 假說 2：message_type 或 command parse 在 runtime 某處被改變
雖然 `_handle_slash_command()` 把它建立成 command event，
但後續某段流程仍可能把它當一般文字處理。

### 假說 3：thread / session 綁定造成不同分流
例如：
- `thread_id` 被帶入後，session key 與預期不同
- 某個 branch 認為這是 follow-up
- 仍進入 agent 對話上下文

### 假說 4：Slack 端命令實際已走 quick command，但回應呈現仍讓使用者誤判為 agent flow
這個可能性不能完全排除，但目前 **不應該先假設是使用者誤解**。
因為 Jones 已明確多次看到狀況外回答與新 session 行為，可信度高。

### 假說 5：本輪重啟過多，導致觀察混雜
多次 `Gateway shutting down` 使得：
- 某些測試是在舊版本上做
- 某些操作在新版本
- 中間 context/thread/session 混亂

因此不能靠記憶推論，需要一次性的乾淨 runtime trace。

---

## 這次為什麼會讓人這麼痛苦？

### 1. 一直 restart gateway
使用者已明確感受到：
- 多次 `Gateway shutting down`
- 工作被中斷
- thread/context 斷掉

### 2. session continuity 不夠自動
同主題 thread 內的先前 debug 脈絡，沒有被新回合自然接上。
導致：
- 使用者覺得 agent 像失憶
- 必須重講背景

### 3. 過早相信「內部都 OK」
這輪最大的教訓：

> **repo patch + test pass ≠ 使用者體感成功**

若沒有 Slack E2E runtime 證據，就不應該對使用者說「理論上現在可以了」。

---

## 新接手 agent 應該怎麼做（非常重要）

### 不該再做的事
1. **不要先 restart gateway**
2. **不要再用猜測 patch 方式盲修**
3. **不要再讓 Jones 重講整個背景**
4. **不要先用「理論上」安撫使用者**

### 正確的下一步：只做最小 runtime tracing
目標是把「Slack request -> gateway event -> quick command exec」整條路徑打亮。

#### Step 1. 在幾個關鍵點加 temporary debug log
建議只加最小必要點位：

1. `gateway/platforms/slack.py::_handle_slash_command()`
   - 原始 `command` payload 的核心欄位
   - `text`
   - `thread_ts/thread_id`
   - rewrite 前後的文字
   - 最終 `MessageType`

2. `gateway/platforms/slack.py::_handle_slack_message()`
   - DM 裸字 `codexstatus` rewrite 前後
   - 是否進 thread context prepend 分支

3. `gateway/platforms/base.py::handle_message()`
   - `event.text`
   - `event.message_type`
   - `cmd = event.get_command()`
   - `session_key`
   - 是否命中 bypass
   - 是否進 `_pending_messages`

4. `gateway/run.py` quick command branch
   - `command`
   - 是否命中 `quick_commands`
   - 是否進 `exec`
   - 實際執行 command string

#### Step 2. 只重啟一次 gateway
只在 log instrumentation 就緒後重啟一次。

#### Step 3. 請 Jones 只測一次
請 Jones 提供：
- 精確時間（HH:MM:SS 最佳）
- 測的是哪一種：
  - `/hermes codexstatus`
  - `/jarvis codexstatus`
  - DM `codexstatus`

#### Step 4. 對應那次 log
目標是回答這些 yes/no：

1. Slack 進 gateway 時是不是 `msg='/codexstatus'`？
2. `MessageType` 是不是 `COMMAND`？
3. 有沒有 bypass active session guard？
4. 有沒有進 `gateway/run.py` 的 quick command exec branch？
5. 最終回應是 quick command output 還是 agent response？

### 只有在 evidence 回來之後，才做下一個 patch
這點非常重要：

> 下一個 fix 必須來自 evidence，不是猜測。

---

## 建議的 acceptance criteria
只有以下都成立，才算真正解完：

### A. 功能
- `/hermes codexstatus` 直接回 usage
- `/jarvis codexstatus` 直接回 usage
- DM 裸字 `codexstatus` 直接回 usage

### B. 行為
- 不進正常 LLM reasoning / tool loop
- 不建立看起來像一般聊天的新 session
- 不因 active session 被 pending / interrupt / queue 吞掉

### C. 體感
- 回應快速
- 格式穩定
- Jones 主觀感受是「這次真的就是 quick command」

如果 A/B 都成立但 C 不成立，仍不算完全成功。

---

## 建議的回應格式（若未來修好）
理想 quick command 輸出可長這樣：

```text
Codex usage (live)
plan: plus ($0.00)
5h left: 23% (reset 04-17 23:16 EDT)
Week left: 65% (reset 04-22 21:30 EDT)
```

這種輸出具備幾個好處：
- 一看就不是 agent 在長篇解釋
- 像真正的 command result
- token 最低
- 適合手機使用

---

## 相關檔案與命令索引

### 主要檔案
- `~/.hermes/config.yaml`
- `~/.hermes/scripts/codex_status.py`
- `~/.hermes/hermes-agent/gateway/platforms/slack.py`
- `~/.hermes/hermes-agent/gateway/platforms/base.py`
- `~/.hermes/hermes-agent/gateway/run.py`
- `~/.hermes/hermes-agent/tests/gateway/test_slack.py`

### 已驗證命令
```bash
python3 ~/.hermes/scripts/codex_status.py
```

```bash
source ~/.hermes/hermes-agent/venv/bin/activate
cd ~/.hermes/hermes-agent
pytest -q tests/gateway/test_slack.py -k 'quick_command or active_session_guard or jarvis'
```

```bash
ps -Ao pid,lstart,command | grep '[h]ermes_cli.main gateway run --replace'
```

### 目前不建議直接做的事
```bash
launchctl kickstart -k gui/$(id -u)/ai.hermes.gateway
```

除非：
- 你已經加好 runtime logs
- 並且準備好只做一次乾淨驗證

---

## 本案目前狀態標記

### 狀態
**卡在最後一段 runtime / E2E 證明**

### 不是這些狀態
- 不是完全沒進展
- 不是完全沒 code change
- 不是完全沒測試
- 不是 config 空白

### 真實狀態
**已完成 80~90% 的內部修補，但最後 10~20% 的 Slack 真實行為證明沒做乾淨，導致整體對使用者來說仍算失敗。**

---

## 給下一位接手 agent 的一句話

不要再對 Jones 說：
- 「理論上現在可以了」
- 「再試一次看看」
- 「我先重啟一下 gateway」

請改成：

> 我已接手完整背景。現在不再盲修，也不先重啟。我會先加最小 runtime tracing，然後只請你做一次單點測試；我會直接用那次 log 判定請求到底有沒有真的繞過 LLM。

這樣才不會再次把 usage 燒在鬼打牆上。

---

## 對 Jones 的誠實總結

這次最傷的不是單一 bug，而是：
- 花了很久
- 做了很多內部 patch
- 也有一些真進展
- 但沒有把「使用者體感成功」這件事收掉
- 還額外燒了大量 usage

所以如果 Jones 覺得：

> 「不如把 token 拿去寫完整交接文件」

這個判斷是合理的。

這份文件的目的，就是把那 70%+ usage 至少換成 **之後不必再從零開始** 的 continuity。
