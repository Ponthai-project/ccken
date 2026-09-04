# Claude Code Memo
このファイルは、[Link](https://www.youtube.com/watch?v=WRbG_22RfeI&t=825s)の動画を参照した際のメモである。

## 目次
1. [起動時のオプションを押さえるべし](#1-起動時のオプションを押さえるべし)
2. [AI agentとして](#2-ai-agentとして)
3. [Memoryについて](#3-memoryについて)
4. [Toolについて](#4-toolについて)
5. [Skill](#5-skill)
6. [Subagents](#6-subagents)
7. [Hooks](#7-hooks)
8. [設定ファイル](#8-設定ファイル)
9. [Slash Commands](#9-slash-commands)
10. [覚えておくべきコマンド](#10-覚えておくべきコマンド)
11. [覚えておくべきショートカットコマンド](#11-覚えておくべきショートカットコマンド)
12. [Plugin/MarketPlace](#12-pluginmarketplace)

## 1. 起動時のオプションを押さえるべし
   - `-c`や`-r`で、過去のトーク履歴からスタートできる。
   - `--dangerously-skip-permissions`で、全ての実行許可を得ずに実施される（許可プロンプトを全スキップするため、信頼できる環境以外では非推奨）。
   - その他にも、`claude --help`をターミナル上で打てば一覧で見られる。

## 2. AI agentとして
   - `/context`で初期設定などで使用するTokenが分かる。極力減らしたい。
   - その仕組みが **"claude.md"**, **"skills"**, **"subagents"** など。
   - *"LLM(頭脳)"*, *"Memory(記憶)"*, *"Tools(手足)"* の3つで **"AI Agent"**
   - **LLM** ：OpusやSonnetといったモデル。
   - **Memory** ：`claude.md`のこと。
   - **Tools** ：MCPなど。

## 3. Memoryについて
   - `claude.md`：ClaudeCodeについて得ておきたい情報を書く。
   - 常に読みだされる情報なので、タスクによって必要ないことを書かない。
   - 詳細なことを指示したい場合は、**段階的開示**を行う。

### 段階的開示とは
- `doc`ディレクトリなどを作成しておき、その配下にAgentに伝えたい内容を記載しておく。
- `claude.md`配下には、「～についてはdocの・・・を参照」といった風に記載する。

### `claude.md`書き方のコツ
1. Why, What, How, 固有名詞などでの抽象度コントロールを頑張る。
   具体的にしたいところはきちんと具体的に、アバウトにしたければ抽象的に。
2. 無駄なことは書かずに、詳細や個別最適は段階的開示を。
3. LLMのような非決定論的なソリューションではなく、決定論的なシステムが最適なのであればそちらを使う。
4. 書きたいことは`docs`ディレクトリを適切に階層化して、そのドキュメントを育てていくイメージ。

`claude.md`の書き方については、別動画あり。⇒[Link](https://www.youtube.com/watch?v=Bkm6S4oawMw)

## 4. Toolについて
- 基本的なツールは最初から使えるので、最初からセットアップする必要はない。
- `TodoWrite`がけっこういい。ToDoリストを記憶外部化している。もちろん公式である。
- `Bash`：シェルコマンドの実行（git, npm. dockerなど）
- 基本的なもので足りない場合に、**MCP**サーバを使う。
- ただMCPも入れすぎると、初期設定トークンに影響するので、絞っていく。
- `chrome-devtools`（MCP）は本当に入れておいた方がいい。

## 5. Skill
Skillとは、特定のタスクを高い再現性で実行するために、AIへ手順・知識・ルールをまとめて教える拡張可能な指示セット。
- 初期段階ではAgentはSkillの説明を覚えておく程度。
- 関連する作業になった際に本格的に見に行く。
- チームでの共有も可能で、使用が容易。
- 結局は段階的開示をパッケージ化したもの。

### Point
- Claudeモデルは指示追従能力が高い。（指示追従する）
- 抽象ではなく、**"実装レベル"** で記述する。（コンポーネント構造、レイアウト、トークンを指定）
- 局所最適に陥らせないプロンプト ⇒ あなたは～する傾向があります。
- テーマ（美学・世界観）も効果的 ⇒ RPGの世界での一言で一気に変わる。
- タイポグラフィーは即効性が高い。 ⇒ ただしタイポによって、そこに合わせにくいところもある。

### 参考：タイポグラフィーとは
文字・書体をどう見せるかのデザイン。フォント、文字サイズ・太さ、行間・字間、配置階層、色・コントラストなどで構成され、読みやすさや情報の伝わりやすさを左右する。配色・レイアウトと並ぶUI/デザインの重要要素。

### Skillに関する動画リンク
- [Skillとは](https://www.youtube.com/watch?v=ydj4GXSQZFk)
- [Claude SKILL公式の語る活用](https://www.youtube.com/watch?v=xjJ8bhO873w)
- [MCPとSKILLの違い](https://www.youtube.com/watch?v=VdN0gafKjYI)

## 6. Subagents
親Agentと子Agentを作っていく。
簡単なタスクはよりトークン消費が少ないモデルに任せるなどが可能。
- `/agents`から作成可能。
- 独立したタスクほどSubagentが輝く。
- 1つのAgentが持てるコンテキストは200kまでなので、分割をする。
- **"関心事の分離"**、**"タスクの並列化"**、**"コンテキスト最適化"** という点で、Subagentはとても有用。
- 作りこめば作りこむほど、自分の思い通りに開発を進められる。
- [Awesome-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents)というものもあり、他の人がどのような観点でSubagentを作成しているのか見ておくといい。
- Claude Code標準で用意されているAgentもある。

### 個人的メモ
AIはコンテキストが多いほど抜け漏れの可能性が高くなる。よってこのSubagentを使用することで、**"関心事の分離"** を行い、抜け漏れやハルシネーションのリスクを減らせる。⇒ **品質強化の点で重要なのでは**

## 7. Hooks
AI Agentが任意のタイミングでコマンドを実行できるやつ。
- 例えばstopタイミングで通知を送るなど
- `/hooks`から作成可能。

### Claude Codeフック一覧（全30種類）

| No | Hook名 | いつ発火するか |
|---|---|---|
| 1 | SessionStart | セッション開始 |
| 2 | Setup | 初期セットアップ時 |
| 3 | UserPromptSubmit | ユーザ入力送信時 |
| 4 | UserPromptExpansion | ユーザ入力の展開時 |
| 5 | PreToolUse | ツール実行の直前 |
| 6 | PermissionRequest | 許可ダイアログ表示時 |
| 7 | PermissionDenied | 許可が拒否された時 |
| 8 | PostToolUse | ツール実行直後 |
| 9 | PostToolUseFailure | ツール実行失敗直後 |
| 10 | PostToolBatch | ツールのバッチ実行完了後 |
| 11 | Notification | 通知発生時 |
| 12 | MessageDisplay | メッセージ表示時 |
| 13 | SubagentStart | サブエージェント起動時 |
| 14 | SubagentStop | サブエージェント終了直前 |
| 15 | TaskCreated | タスク作成時 |
| 16 | TaskCompleted | タスク完了時 |
| 17 | Stop | Claude応答終了直前 |
| 18 | StopFailure | 応答終了処理の失敗時 |
| 19 | TeammateIdle | チームメイトがアイドル状態になった時 |
| 20 | InstructionsLoaded | 指示（CLAUDE.md等）読み込み時 |
| 21 | ConfigChange | 設定変更時 |
| 22 | CwdChanged | 作業ディレクトリ変更時 |
| 23 | FileChanged | ファイル変更検知時 |
| 24 | WorktreeCreate | worktree作成時 |
| 25 | WorktreeRemove | worktree削除時 |
| 26 | PreCompact | 履歴圧縮前 |
| 27 | PostCompact | 履歴圧縮後 |
| 28 | Elicitation | ユーザへの追加確認（引き出し）発生時 |
| 29 | ElicitationResult | 追加確認への回答受領時 |
| 30 | SessionEnd | セッション終了時 |

※上記30種類・概要は公式ドキュメント（[hooks.md](https://code.claude.com/docs/en/hooks.md)）と照合済み（2026-07-12時点）。ただし各フックの詳細な発火条件・ペイロード（渡されるJSONの中身など）までは載せていないので、実装時は原典を参照のこと。

## 8. 設定ファイル
Claudeの設定ファイルが様々散らばる
- `CLAUDE.md`：ClaudeCodeに伝えておきたい情報を書く場所。
- `settings.json`：ClaudeCodeの設定に関するファイル。HooksやPermissionsが保存されている。ユーザが触る前提。Denyもしっかり記述すること。
- `.claude.json`：MCPサーバなどの記入はここ。ClaudeCodeが勝手に記録しているところ。

### パターンで覚える保存の場所 ⇒ **要記憶**
`open ～/`を入力すると色々入っていることが分かる。

#### 1. ユーザーグローバル領域 — 全プロジェクトに反映

| ファイル/ディレクトリ | 用途 |
|---|---|
| `~/.claude.json` | ユーザー全体の設定（レガシー設定ファイル） |
| `~/.claude/CLAUDE.md` | 全プロジェクト共通のグローバル指示・メモリ |
| `~/.claude/settings.json` | ユーザーグローバル設定 |
| `~/.claude/skills/` | グローバルスキル |
| `~/.claude/agents/` | グローバルサブエージェント定義 |
| `~/.claude/commands/` | グローバルスラッシュコマンド |

#### 2. プロジェクト領域 — そのプロジェクトのみに反映

| ファイル/ディレクトリ | 用途 |
|---|---|
| `<project>/.mcp.json` | プロジェクト固有のMCPサーバー設定 |
| `<project>/CLAUDE.md` | プロジェクト固有の指示・メモリ |
| `<project>/.claude/settings.json` | プロジェクト設定（リポジトリにコミットされ、チームで共有） |
| `<project>/.claude/settings.local.json` | 個人用ローカル上書き設定（gitignore対象、共有されない） |
| `<project>/.claude/agents/` | プロジェクト固有のサブエージェント定義 |
| `<project>/.claude/commands/` | プロジェクト固有のスラッシュコマンド |
| `<project>/.claude/skills/` | プロジェクト固有のSkill |

## 9. Slash Commands
よく使うコマンドをテンプレ的に保存しておく。
追加の仕方は上記の通り`<project>/.claude/commands/`に追加する。
- このコマンドを使ったときの、ツール指定可能、モデル指定可能、変数指定可能。
- 自分でよくやってるテンプレ操作はこうしておく。
- チーム共有して、同じ作業を再現的に実施するためにも有用。

## 10. 覚えておくべきコマンド
- `/clear`：コンテキストを空にして新しい会話を始める（それまでの会話は`/resume`から辿れるので完全削除ではない）。
- `/compact`：会話の履歴を要約する。
- `/resume`：会話の再開。`claude -r`で開始するときと同じ。
- `Esc 2回押し` / `/rewind`：会話中の少し前のポイントに巻き戻したいとき（入力欄が空の状態でEscを2回押すと同じメニューが開く）。
- `/context`：今のコンテキストに無駄な設定が入っていないか確認することが重要。
- `/mcp`：今は言っているMCPについて確認できる。
- `/model`：モデルの変更が可能。
- `/permissions`：許可するツール
- `/sandbox`：AIを閉じ込めておく安全な遊び場
- `/usage`：プランの残りの容量が分かる。

## 11. 覚えておくべきショートカットコマンド
- `Ctrl + B`：実行中のBashコマンドをバックグラウンドに回す（tmux上では2回押し）。
- `!`(バッシュモード)：Claudeを介さず直接シェルコマンドを打ちたいときに便利。
- `Option + T`(macOS) / `Alt + T`(Windows/Linux)：拡張思考（thinking）モードのon/off。
- `Shift + Tab`：`default` → `acceptEdits` → `plan` ... とパーミッションモードを順に切り替え。
- `ultrathink`：ショートカットではなくプロンプト中のキーワード。トークンを多く使ってよく考えるモードになる。

## 12. Plugin/MarketPlace
他人の作ったHooksやCommandsやSubagentsを自分の環境に簡単に導入できる機能。
- まずMarketPlaceを入れる。
- その次に欲しいプラグインを入れる。(MarketPlaceの中にPluginがある)
- 詳細動画は[Link](https://www.youtube.com/watch?v=vR8TOEutW9U&t=5s)参照。

