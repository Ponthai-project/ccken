# Orca IDE 概要

Orcaは「複数のAIコーディングエージェントを並列実行するために設計されたIDE」であり、公式には **ADE（Agent Development Environment）** と称している。開発元はStably AIで、OSS（MIT License）として公開されている。

> ⚠️ 注意：セキュリティ製品の「Orca Security / ORCA GRC」とは全くの別製品なので混同しないこと。本資料が扱うのは開発ツールとしての **Orca（onorca.dev）** である。

## 目次
1. [できること（主な機能）](#1-できることmain機能)
2. [対応環境](#2-対応環境)
3. [対応AIエージェント](#3-対応aiエージェント)
4. [価格・ライセンス](#4-価格ライセンス)
5. [導入方法](#5-導入方法)
6. [向いているユーザー像](#6-向いているユーザー像)
7. [参考リンク](#7-参考リンク)

## 1. できること（主な機能）

| 機能 | 内容 |
|---|---|
| 並列worktree実行 | タスクごとに独立したgit worktreeを作成し、複数のエージェント（Claude Code, Codex等）を同時並行で走らせて出力を比較できる |
| 各worktreeに独立コンテキスト | worktreeごとに専用のターミナル・ブラウザタブ・コンテキストが割り当たる |
| Diffレビュー | AIが生成したdiffの行に直接コメントを付け、そのまま修正指示としてエージェントへ送り返せる（行番号コピーやチャット画面への切り替え不要） |
| Design Mode | UI要素をインスペクトしながら指示を出せるモード |
| GitHub/Linear連携 | Issue/タスク管理ツールとネイティブに連携 |
| SSH worktree | リモートの高スペックマシン上でworktreeを実行可能 |
| CLIインターフェース | スクリプトからのワークフロー操作に対応 |
| ファイルドラッグ＆ドロップ | プロンプトへファイルを直接ドロップ可能 |
| モバイル連携アプリ | iOS/Androidからデスクトップのセッションを遠隔監視・操作できる |
| ターミナル分割（WebGL描画） | 高速なターミナル表示 |

## 2. 対応環境

| プラットフォーム | 対応状況 |
|---|---|
| macOS | ○（Apple Silicon / Intel 両対応） |
| Windows | ○ |
| Linux | ○（AppImage形式で配布、Arch Linuxは AUR 経由も可） |
| iOS | ○（App Store / TestFlight、モバイル companion app） |
| Android | ○（APK配布） |

- 動作にはgitと、実際に使うCLIエージェント（Claude Code等）のインストールが前提。
- 詳細なハードウェア要件は公式ドキュメントに明記なし。SSH worktree機能を使えば「手元マシンが非力でもリモートの高性能サーバ側で実行する」運用も可能。

## 3. 対応AIエージェント

40種類以上のCLIエージェントに対応（「ターミナルで動くCLIエージェントなら基本何でも」がコンセプト）。

- **主要どころ**：Claude Code, Codex, Cursor CLI, GitHub Copilot CLI, Grok, OpenCode, Pi
- **その他**：Cline, Continue, Devin, Goose, Hermes Agent, Kimi Code, Mistral Vibe, Qwen Code 等

## 4. 価格・ライセンス

- **Orca本体：無料・OSS（MIT License）**。ダウンロード・利用に直接コストはかからない。
- ただし実行するのは各社のAIエージェントであるため、**利用料は使うエージェント側のサブスクリプション/API課金に依存**する。
  - 例：Claude Codeを使うならAnthropicの契約（Claude Pro/Max やAPI従量課金）、Codexを使うならOpenAI側の契約が別途必要。
  - Orcaは「自分が既に持っているサブスクリプションでエージェントを動かす」という設計思想（"Run any coding agent with your own subscription"）。

## 5. 導入方法

| 方法 | 内容 |
|---|---|
| デスクトップ | onorca.dev からダウンロード、またはmacOSはHomebrew、ArchはAUR経由でインストール可能 |
| モバイル | iOS App Store / Android APK（companion appとしてデスクトップとペアリング） |

## 6. 向いているユーザー像

「AIを代替ではなくレバレッジとして使いたい、普段からコードを書いている開発者」向け。diffを読み、コミットを気にかけ、worktreeを整理する運用に慣れていることが前提とされている。既存IDEにAI機能を後付けしたものではなく、**並列エージェントワークフローを前提に一から設計**されている点が特徴。

## 7. 参考リンク

- [Orca 公式サイト](https://www.onorca.dev/)
- [Orca Docs（公式ドキュメント）](https://www.onorca.dev/docs)
- [GitHub - stablyai/orca](https://github.com/stablyai/orca)
- [Orca IDE - App Store](https://apps.apple.com/us/app/orca-ide/id6766130217)
- [Orca Review 2026 - vibecodinghub](https://vibecodinghub.org/blog/orca-review)

※本資料は2026-07-12時点のWeb検索結果に基づく。価格・対応エージェント一覧は変更が早い領域のため、導入前に必ず公式サイト（onorca.dev）で最新情報を確認すること。
