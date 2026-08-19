# claude-esp-rs

[![Crates.io](https://img.shields.io/crates/v/claude-esp)](https://crates.io/crates/claude-esp)

即時把 Claude Code 隱藏的輸出（思考、工具呼叫、subagent）串流到另一個終端機。

*English: [README.md](README.md)*

![claude-esp-rs 螢幕截圖](docs/screenshot.png)

> **注意：** 這是 [Go 版本](https://github.com/phiat/claude-esp)的 Rust 移植。執行檔命名為 `claude-esp-rs`，讓兩個版本可以並存。

## 要解決的問題

互動式使用 Claude Code 時，工具輸出和思考預設是收合的，得按 `Ctrl+O` 才能展開。這個工具讓你在**另一個終端機**用 TUI 看到全部輸出，不必打斷手上的 Claude Code session。

## 功能

- **多 session 支援** — 同時盯著所有活躍的 Claude session
- **階層式樹狀檢視** — session 底下巢狀著 Main／Agent 節點
- **即時串流** — 思考、工具呼叫、輸出一發生就看得到
- **subagent 追蹤** — 自動發現並顯示 subagent 的活動
- **session 事件** — compaction 邊界、hook 輸出、編輯後的 LSP 診斷、PR 連結事件、提示快取未命中（附原因與額外 token 數）、佇列操作、斜線指令呼叫、暫時性通知、subagent 被終止的標記、plan／auto 模式切換，以及 skill／MCP／工具的增減，全部就地顯示
- **agent 類型標籤** — 從 `.meta.json` 讀出 agent 類型（Explore、code-reviewer 等）並顯示
- **token 用量追蹤** — 標題列顯示累計的輸入／輸出 token 數
- **各 agent 的 context 大小** — 每個 Main／subagent 列會顯示目前 context 佔該模型 context window 上限的百分比（`Main 18%`、`Explore 9%`）。分母是模型的 *window 上限*（Claude 5 家族與 opus-4-6 到 4-8 ／ sonnet-4-6 為 1M，haiku-4-5 為 200k），**不是**自動 compact 的門檻
- **工具執行時間** — 顯示每次工具呼叫花了多久
- **背景任務可見度** — 背景任務（⏳／✓）顯示在派出它的 agent 底下
- **篩選** — 可依 session／agent 個別切換思考、工具、輸出的顯示
- **自動捲動** — 跟著新輸出走，也可以自由捲動翻歷史

## 需求

- Rust 1.86 以上

## 安裝

```bash
# 從 crates.io 安裝
cargo install claude-esp

# 或者 clone 下來自己編
git clone https://github.com/phiat/claude-esp-rs.git
cd claude-esp-rs
cargo build --release

# 選用：安裝到 PATH
cp target/release/claude-esp-rs ~/.local/bin/
```

### 預先編譯的執行檔

可以從 [Releases](https://github.com/phiat/claude-esp-rs/releases) 頁面下載。提供 Linux（amd64、arm64）、macOS（amd64、arm64）與 Windows（amd64）版本。

## 使用方式

```bash
# 主終端機：照常執行 Claude Code
claude

# 第二個終端機／tmux pane：執行 watcher
claude-esp-rs
```

### 選項

| 選項         | 說明                                          |
| ------------ | --------------------------------------------- |
| `-s <ID>`    | 只盯指定 ID 的 session                        |
| `-n`         | 從最新開始（略過歷史，只看即時輸出）          |
| `-l`         | 列出最近的 session                            |
| `-a`         | 列出活躍的 session                            |
| `-p <ms>`    | 輪詢間隔毫秒數（僅 fallback 模式有效，預設 500） |
| `-w <秒>`    | 活躍判定的時間窗（預設 `300` = 5 分鐘）       |
| `-m <N>`     | 樹狀圖最多顯示幾個 session（預設 0 = 無上限） |
| `-c <秒>`    | 閒置達 N 秒的 session 自動收合（預設 0 = 停用，例如 `120` 代表 2 分鐘） |
| `-D`         | debug：把 parser 平常會丟棄的 JSONL 行，其原始 `type:subtype` 也顯示出來 |
| `-V`         | 顯示版本                                      |
| `-h`         | 顯示說明                                      |

### 環境變數

| 變數          | 說明                                                |
| ------------- | --------------------------------------------------- |
| `CLAUDE_HOME` | 覆寫 Claude 設定目錄（預設：`~/.claude`）           |

### 範例

```bash
# 盯所有活躍的 session
claude-esp-rs

# 略過歷史，只顯示新輸出
claude-esp-rs -n

# 列出活躍的 session
claude-esp-rs -a

# 盯特定 session
claude-esp-rs -s 0b773376

# 縮短輪詢間隔（200ms）
claude-esp-rs -p 200

# 列出最近的 session
claude-esp-rs -l
```

## 快捷鍵

| 按鍵      | 動作                                      |
| --------- | ----------------------------------------- |
| `t`       | 切換思考的顯示                            |
| `i`       | 切換工具輸入的顯示                        |
| `o`       | 切換工具輸出的顯示                        |
| `x`       | 切換文字／回應的顯示                      |
| `a`       | 切換自動捲動                              |
| `h`       | 隱藏／顯示樹狀窗格                        |
| `A`       | 切換是否自動發現新的 session              |
| `tab`     | 在樹狀窗格與串流窗格之間切換焦點          |
| `j/k/↑/↓` | 在樹狀圖移動，或捲動串流                  |
| `space/enter` | 在 session 上：收合／展開（手動展開會釘住）· 在 agent 上：切換顯示 · 在背景任務上：載入輸出 |
| `s`       | 單獨顯示選取的 session／agent（可切回）   |
| `d`       | 把選取的 session 從樹狀圖移除             |
| `g/G`     | 跳到串流的最上／最下                      |
| `q/Ctrl+c`| 離開                                      |

## 自動收合

加上 `-c 120` 執行，閒置滿 2 分鐘的 session 就會自動收合。收合的 session 在樹狀圖顯示 `▸` 而非 `▾`，並標示被藏起來的 subagent 數量（例如 `📂▸ my-session (+2)`）。收合 session 的子節點也會從串流窗格濾掉——重點就在於別讓睡著的 session 佔滿畫面。

在收合的 session 上按 `space` 可以手動展開。手動展開會**釘住**該 session：除非它重新有活動、之後又再度閒置，否則不會再自動收合。按 `s` 可以單獨顯示某個 session；如果它是收合狀態，Solo 會強制展開並釘住，讓你看得到輸出。

## 運作原理

Claude Code 把對話記錄存成 JSONL 檔，位置在：

```
~/.claude/projects/<專案路徑>/<session-id>.jsonl
```

subagent 存在：

```
~/.claude/projects/<專案路徑>/<session-id>/subagents/agent-<id>.jsonl
```

背景任務的輸出存在：

```
~/.claude/projects/<專案路徑>/<session-id>/tool-results/toolu_*.txt
```

watcher 做的事：

1. 找出活躍的 session（最近 5 分鐘內有異動的）
2. 用作業系統原生的檔案系統通知（[notify](https://docs.rs/notify)）即時偵測檔案變動（Linux 用 inotify，macOS 用 kqueue／FSEvents）
3. 在不支援通知的檔案系統上（NFS、某些跨檔案系統的 WSL2 設定）退回輪詢模式（可用 `-p` 調整）
4. 對密集寫入做 debounce（50ms 窗口），有效處理爆量輸出
5. 解析 JSON 行，取出 thinking／tool_use／tool_result
6. 找出背景任務，並對應回派出它的 agent
7. 用 TUI 呈現，附帶樹狀導覽與篩選

## tmux 設定

建議的 tmux 版面：

```bash
# 建立含兩個 pane 的 tmux session
tmux new-session -s claude \; \
  split-window -h \; \
  send-keys 'claude-esp-rs' C-m \; \
  select-pane -L \; \
  send-keys 'claude' C-m
```

或者寫進 `.tmux.conf`：

```
bind-key C-c new-window -n claude \; \
  send-keys 'claude' C-m \; \
  split-window -h \; \
  send-keys 'claude-esp-rs' C-m \; \
  select-pane -L
```

之後按 `prefix + Ctrl+C` 就能開一個 Claude Code 工作區。

## 專案結構

```
claude-esp-rs/
├── Cargo.toml
├── src/
│   ├── main.rs          # CLI 進入點
│   ├── lib.rs           # 函式庫匯出
│   ├── types.rs         # 資料結構
│   ├── parser.rs        # JSONL 解析
│   ├── watcher.rs       # 檔案監看
│   └── tui/
│       ├── mod.rs       # Ratatui 主程式
│       ├── tree.rs      # session／agent 樹狀檢視
│       ├── stream.rs    # 可捲動的輸出串流
│       └── styles.rs    # 顏色與樣式
└── tests/
    └── integration_test.rs
```

## 開發

以 [Ratatui](https://github.com/ratatui/ratatui) 與 [Tokio](https://tokio.rs/) 建構。

```bash
# 執行測試
cargo test

# 編譯 release
cargo build --release

# 檢查格式
cargo fmt --check

# Lint
cargo clippy
```

## 授權

MIT
