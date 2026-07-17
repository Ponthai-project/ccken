# Copilot ／ Claude Code 両方で使えるスキル・ハーネスの紹介

- 発表者: 平野孝明
- 出典イベント: 記載なし
- 公開日: 2026-07-13
- 元ファイル: `Copilot ／ Claude Code 両方で使えるスキル・ハーネスの紹介.pdf`（9p）
- Speaker Deck: https://speakerdeck.com/colopl/skills-and-harnesses-that-work-with-both-copilot-and-claude-code

## 発表概要

スライド内表記の登壇日は2026.06.30（タイトルは「Copilot / Claude Code 両方で使えるハーネス」）。GitHub CopilotとClaude Codeの双方で使い回せるハーネスの設計・運用について、実践的な知見を共有する内容。

## ハーネスとは

タスクの完了（ゴール）にたどりつくまでの**ガードレール**のこと。ユーザー側が定義できるハーネス（外部ハーネス）は、Claude／GitHub Copilotでそれぞれ以下のファイルに対応する。

| 種別 | Claude | GitHub Copilot |
|---|---|---|
| ルール定義 | CLAUDE.md | copilot-instructions.md |
| 追加ルール | .claude/rules | .github/instructions |
| スキル | .claude/skills | .github/skills |
| 設定・フック | .claude/settings.json | .github/hooks |
| コマンド | .claude/commands（skillsにマージ） | .github/prompts |

## ハーネスの2分類

1. **タスクのゴールまでの方向を最初に示すハーネス（初期コンテキスト）**: CLAUDE.md、skills（commands）、agents。
   - メリット: 無駄な寄り道をせず、トークンの消費量が減る
   - デメリット: システムプロンプトやMemoryの差で汎用化の難易度が高い（属人化しやすい）
2. **タスクのゴールへの方向がズレたときに修正するハーネス（遅延ロード）**: rules、skills、hooks。
   - メリット: 外科的に入れることができるため追加が容易
   - デメリット: 毎セッション利用されるとは限らないので性能向上が1と比べて限定的

→ 共通化・汎用化を行いたい場合は、2の遅延ロード型ハーネスの方が難易度が低くおすすめ。

## 利用パターン別のハーネス構成

AIの利用方法によって必要なハーネスの量は変化する、として3つの例が示される。

- **例① AgenticCoding**: Start→Goalの間にRules（CLAUDE.md）／Skills（commands）／Hooksを複数配置するフルセット構成。
- **例② VibeCoding**: Start→GoalをCLAUDE.mdのみで進める最小構成。
- **例③ 対話形式（短いセッション）**: CLAUDE.md＋commandsを併用し、Skillsを介して進める中間的構成。

## 今日覚えてほしい共通ハーネス

Skillsやrulesはプロジェクト依存で共通化しづらいため、以下の2つに絞って紹介する（それ以外は知見も少なく、アイデアがあれば逆に教えてほしいと呼びかけている）。

### 1. grill-me / grill-with-docs（初期コンテキストを改善するハーネス）

- 計画やデザインについて、共通理解に達するまでユーザーに徹底的にインタビューし、意思決定ツリーの各分岐を解決するスキル。
- できること: 初期コンテキストに質の高い情報を渡すための準備ができる／ヒアリング形式で進むのでゴールまでの方向を人間が理解できる／`grill-with-docs`では新規の仕様が決まったときにドキュメントとして残すことができる。
- デメリット: ヒアリングの時間が長く疲れる。

### 2. PreToolUse hooks + additional context（遅延ロードコンテキストを改善するハーネス）

- 危険なコマンドや非推奨のコマンドに対してdenyで拒否すると同時に、任意のコンテキストを注入する仕組み（参考: Claude公式reference）。
- できることの例:
  - `.env`等アクセス禁止ファイルへのアクセス時に「他の手段を利用して内容を確認せずブラックボックスとして扱ってください」と挿入
  - Unityのプレハブやアセットファイルへのアクセス時に「UnityMCPを利用してください」と挿入
  - `/tmp`等特定のディレクトリ以下へのアクセス時に「一時ファイルのため、必要な情報がありません」と挿入

## 要点まとめ

- ハーネスはClaude Code／GitHub Copilotの双方に、ルール・スキル・設定/フック・コマンドという対応するファイル体系が存在する。
- ハーネスは「初期コンテキスト型（最初にゴール方向を示す）」と「遅延ロード型（ズレを修正する）」の2種類に大別され、共通化・汎用化を狙うなら遅延ロード型（rules/skills/hooks）が有利。
- AIの利用スタイル（AgenticCoding／VibeCoding／対話形式）によって必要なハーネスの量・構成は変化する。
- 汎用的に使える共通ハーネスとして、初期コンテキストを改善する「grill-me / grill-with-docs」と、遅延ロードコンテキストを改善する「PreToolUse hooks + additional context」の2つが紹介された。
- PreToolUse hooksは危険・非推奨コマンドの拒否と同時にコンテキスト注入を行うことで、AIの誤った振る舞いを事前に修正できる。
