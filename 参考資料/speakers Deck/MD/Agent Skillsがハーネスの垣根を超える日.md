# Agent Skillsがハーネスの垣根を超える日

- 発表者: Gota (@gota_bara)
- 出典イベント: Claude CodeからCodex CLIへ — Tech Garden(ROSCAFE)
- 公開日: 2025-12-21
- 元ファイル: `Agent Skillsがハーネスの垣根を超える日.pdf`（24p）
- Speaker Deck: https://speakerdeck.com/gotalab555/agent-skillsgahanesunoyuan-gen-wochao-eruri

## 登壇者について

Gota（@gota_bara）。職種はAgentic AI Engineer & データアナリスト。小売向けデータプロダクト／AIエージェント開発／データ整備に従事。Kiro-styleの仕様駆動開発（SDD）ツール「cc-sdd」のテンプレート機能を開発しAgent Skills対応も進める一方、Agent Skillsの一元管理・運用（Ops）を容易にするOSSも鋭意更新中。

## エージェントハーネスとしてのClaude Codeは強力

| 機能 | Claude Code | Codex | 備考 |
|---|---|---|---|
| Model | Opus-4.5 | GPT-5.2-Codex | 違いはあるが両方賢い |
| Subagents | ○ | × | Subagentsが必要な場面は実は少ない |
| Hooks | ○ | × | この差は個人的に重要 |
| MCP | ○ | ○ | 仕様が異なる |
| Plugins | ○ | × | チーム・組織利用では重要だが結構複雑 |
| LSP (Plugins) | ○ | × | |

- Claude Codeの設計思想（カスタマイズ性・シンプルさ）を評価する一方、「目的に適したモデルが何か」で利用するエージェントは大きく変わるため、特定エージェントへのロックインのリスクは避けたい、という立場を取る。

## 「実務」には手順とコンテキストが必要

Anthropic「Equipping agents for the real world with Agent Skills」からの引用:

> Claude is powerful, but real work requires procedural knowledge and organizational context.

「手順」と「コンテキスト」をエージェントに渡す手法としてAgent Skillsは秀逸で徐々に認知が進んでいたが、当初はClaudeでしか利用できなかった（SkillPortはClaude以外でも使えることを目指し初期開発された）。

## Agent SkillsがOpen Standardに

- 2025年10月16日: AnthropicがClaudeにAgent Skillsを発表。
- 2025年12月18日: Agent SkillsがOpen Standardとして公開。Codex、GitHub Copilot Agent、Cursor等でも続々とSkillsへの対応が発表された（"Don't Build Agents, Build Skills Instead" というスローガンも引用されている）。

## Agent Skills＝ポータブルなコンテキスト

- エージェントに必要な「業務知識」を1つのフォルダで渡せる、どのエージェントでも読み込めるオープン標準。ただのファイルのため管理・共有が簡単。Agent Skillsはエージェントに依存せずどこでも使える。
- 構造はシンプルで、必須なのはSKILL.mdだけ。それ以外は必要に応じて柔軟に拡張可能。
- SKILL.mdの書き方: YAML Frontmatterが上部にあるだけのMarkdownファイルに業務手順や知識を書くだけ。Frontmatterには`name`、`description`が必須。

## エージェントがSkillsを使う仕組み：Progressive Disclosure

- コンテキストの段階的開示（検出→アクティベーション→実行）により、高速に膨大なコンテキストにアクセス可能。
- 「どんなSkillがあるか」を把握 → 必要なSKILL.mdだけを読む → リソースを段階的に展開しながら実行（コードは展開せずに実行可能）。

## Agent Skillsが汎用エージェントを特化型に簡単に変える

- 業務知識を持つ人自身がSkillsを作るだけでAIの振る舞いを定義でき、特化型エージェントを作ることができる。
- これまでAI Agentでは扱うことが難しかったタスクも実行可能になる: ①マルチモーダル（あらゆるファイル形式）②繰り返し可能なワークフロー ③実行環境が用意できるなら実行環境でできること大体何でも。
- シンプルかつ拡張性が高いフォーマットで業務知識をパッケージ化できる。

## CodexでAgent Skillsを使う（実演）

- `/skills`で利用可能なSkillsを確認可能。コマンドのように選択してSkillsを発動できる（Custom CommandsはSkillsに統合されそうという見立て）。デフォルトのSkillsも用意されている。
- 任意のリポジトリからSkillsをインストール可能。`~/.codex/skills/`配下（または`$CWD/.codex/skills/`）にインストールされる。Skill Installerのインストール先を`$CODEX_HOME`から変更すれば、別のエージェントでもSkillsを登録・使い回せる。
- あとは選択して使うだけでよく、明示しなくとも動的にSkillを発動できる。Codex Webでも、明示的な呼び出しはサポートしていないもののSkills自体は利用可能。
- CodexにはないplanモードがSkills経由でexperimentalとして追加されている例も紹介（Codexはエージェントがシンプルだからこそ、"Skills"中心の思考に変えやすいとする）。

## 到達すべき境地と年末年始のリセット提案

- 「ハーネスをカスタマイズしなくても、良いSkillsさえあれば十分だった」という到達点（本当かどうかは未検証としつつ提示）。
- 年末年始の提案「一度リセットしませんか」:
  1. 複雑にカスタマイズした設定やロジックを含むMCPサーバーを一度全て削除する
  2. まっさらなCodexに標準MCPサーバー（できるなら必要なツール）だけ接続する
  3. ロジックや業務知識はAgent Skillsとして載せていく
- ハーネスのカスタマイズは最終手段であり、Skillsで構築することを提案。

## Agent Skillsには見過ごせないリスクがある

- **Supply Chain Attack**: 野良Skillに悪意あるコードやプロンプトが含まれていたら？
- **Indirect Prompt Injection**: 読み込んだログファイルやドキュメントに攻撃命令が隠されていたら？
- **Shadow AI**: 組織で承認されていないSkillが勝手に利用されたら？
- 対策: Skills利用時は中身を全て確認した上で利用する。更新も注意が必要。

## まとめ

- Agent Skillsはエージェントの垣根を超えたポータブルな資産となる。Codex／Claude Code／その他Agent何でも良いので、とにかくSkills化を進めていくべき。
- Agent Skillsの運用・管理を容易にするOSSを開発中。今回はSkillsの良いところを語ったが課題もあるのは事実で、それらに対処できるよう鋭意更新中。

## 要点まとめ

- Claude CodeはSubagents・Hooks・Plugins等でCodexより機能的に優位だが、特定エージェントへのロックインは避けるべきという立場。
- Agent Skillsは「手順」と「コンテキスト」をポータブルに渡す仕組みで、2025年12月にOpen Standard化され、Codex・GitHub Copilot Agent・Cursor等が続々対応。
- SKILL.md1枚（name・descriptionを含むYAML Frontmatter）が必須で、Progressive Disclosure（検出→アクティベーション→実行）によりコンテキストを段階的に開示する。
- Agent Skillsにより汎用エージェントをマルチモーダル・繰り返しワークフロー対応の特化型エージェントに簡単に変えられる。
- 「ハーネスのカスタマイズより先にSkillsで構築する」という年末年始のリセット提案がなされている。
- Supply Chain Attack・Indirect Prompt Injection・Shadow AIといったAgent Skills特有のセキュリティリスクにも注意が必要。
