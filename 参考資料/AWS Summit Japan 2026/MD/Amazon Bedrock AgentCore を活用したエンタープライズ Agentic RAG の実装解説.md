# Amazon Bedrock AgentCore を活用したエンタープライズ Agentic RAG の実装解説

## 資料メタ情報

- **セッションID**: AIM412
- **イベント名**: AWS Summit Japan 2026
- **登壇者/出典**: 辻浩季（アマゾンウェブサービスジャパン合同会社、技術統括本部 ストラテジックエンタープライズ本部 自動車・製造第一ソリューション部 ソリューションアーキテクト）
- **元ファイル名**: Amazon Bedrock AgentCore を活用したエンタープライズ Agentic RAG の実装解説.pdf
- **ページ数**: 100

## セッション概要

- **対象者**: 社内でRAGシステムを導入したが性能が出ずに困っているエンジニア、認証認可を含むセキュアなエージェント運用に課題を持つエンジニア
- **アジェンダ**:
  1. Agentic RAG とは
  2. 本セッションで構築する Agentic RAG システムの概要紹介
  3. 実装の詳細解説
  4. まとめ

登壇者の得意領域は時系列予測・生成AI、好きなAWSサービスは Amazon Bedrock AgentCore・Amazon SageMaker AI。前職は総合電機メーカーで機器異常検知システムの設計・開発に従事。

## よくある課題（導入）

AI エージェント／RAGシステム運用の現場で直面しがちな課題として、以下4点が挙げられた。

1. **的外れな回答**: ナレッジベースを検索してもユーザーの意図・文脈を理解できず関連性の低い情報を返す
2. **コストとレイテンシーの増大**: 会話履歴を全てコンテキストに含めるとトークン数が膨大になり、コスト増・応答遅延・精度低下を招く
3. **本番環境への展開の困難さ**: PoCでは動作しても、エンタープライズ環境への展開ができない
4. **会話を覚えていない**: エージェントはステートレスであり、過去の会話やユーザーの好みを記憶できず毎回ゼロからのやり取りになる

これらに対し、従来のRAGでは解決できない課題への新しいアプローチとして Agentic RAG の構築が提案される。

## 1. Agentic RAG とは

### RAG (Retrieval-Augmented Generation) の基本

質問（Query）→ 検索（Retrieve）→ 拡張（Augment）→ 生成（Generate）→ 応答（Response）という基本サイクル。

### Context Engineering の観点

Anthropic の記事（"effective context engineering for ai agents"）を引用し、以下の考え方を紹介。

- **コンテキストロット（Context Rot）**: LLM はコンテキストウィンドウに含まれるトークン数が増えるほど、情報を正確に想起する能力が低下していく
- 対策として「本当に必要な情報を最小限に集めたトークンセット」＝「最小限の高シグナルトークンセット」を用意することが重要
- コンテキストは貴重で有限なリソースであり、多くの情報を詰め込むのではなく、ツールやメモリを含めて厳選した情報を与えることがエージェントのパフォーマンスを最大化する

### Agentic RAG の位置づけ

上記の Context Engineering の考え方を RAG サイクル（Query → Retrieve → Augment → Generate → Response）に組み込んだものが Agentic RAG として提示された。

## 2. 本セッションで構築する Agentic RAG システムの概要

### AWSome Summit Agent（構築対象のデモアプリ）

「金曜日の生成AIセッションは何がありますか？」「私におすすめのセッションは？」「AIエージェントのセキュリティについて学ぶのに良いセッションはどれですか？」「前回話したセッションはどれでしたか？」といった、AWS Summit 2026 のセッションに関する問い合わせに答えるエージェント。GitHub にて公開されている（QRコードで案内、URLはスライド画像内のため詳細不明）。

### アーキテクチャ概要

- ログイン: Amazon Cognito
- 会話履歴とユーザー設定の保存・取得: エージェントメモリ（Memory）
- イベント情報の検索: RAG（Knowledge Bases）
- 上記をエージェントが呼び出し、アプリケーションへレスポンスを返す構成

### AWSome Event Agent の詳細アーキテクチャ

- **AgentCore runtime**: セッションA・セッションBのように呼び出しごとに分離されたセッションで動作
- **AgentCore identity**: Amazon Cognito と連携し、セッショントークンを検証
- **AgentCore memory**: Actor（ユーザー）単位で短期メモリ（セッション）と長期メモリ（ユーザー設定）を管理
- **Amazon Bedrock Knowledge Bases**: イベント情報の検索
- 上記を **Strands Agent** が統合し、アプリケーションからの呼び出しに応答する

## 3. 実装の詳細解説

構築は以下5ステップで進められる。各ステップで利用するAWSサービスが積み上げ式に紹介される。

| Step | 内容 | 利用サービス（新規追加分） |
|---|---|---|
| Step 1 | エージェントの構築 | Strands Agent |
| Step 2 | Knowledge Bases の構築 | Amazon Bedrock Knowledge Bases |
| Step 3 | メモリコンポーネントの構築 | Amazon Bedrock AgentCore memory |
| Step 4 | Runtime へのデプロイ | Amazon Bedrock AgentCore runtime |
| Step 5 | 認証認可機能の追加 | Amazon Bedrock AgentCore identity |

### Step 1: エージェントの構築（Strands Agents）

- モデル駆動型アプローチを採用したオープンソースのAIエージェント開発用SDK
- わずか数行のコードでAIエージェントを実装可能
- `MODEL_ID` を変更するだけでBedrock上の基盤モデルを柔軟に切り替え可能
- OpenAIのほか、Anthropic, LiteLLM, Llama, Ollama, Gemini, Writerなど主要プロバイダーのAPI呼び出しをサポート
- 23の組み込みツールに加え、`@tool` デコレータでカスタムツールを簡単に追加可能

### Step 2: Knowledge Bases の構築

Amazon Bedrock Knowledge Bases によるインデックス作成フロー（ソースデータ → テキスト抽出 → チャンキング → 埋め込み → インデックス作成）。Amazon S3にAWS Summit 2026のセッション情報を格納し、ベクトルストアには事前作成済みの Amazon S3 Vectors を使用。

エージェントとの統合は、Strands Agents の `@tool` デコレータを用いて Knowledge Base にアクセスするツール（`search_summit_sessions` 等）を定義し、システムプロンプトで「AWS Summit 2026のイベント情報が格納されたナレッジベースにアクセスできる優秀なイベントアシスタント」と役割を定義するという実装例が示された。

### Step 3: メモリコンポーネントの構築（AgentCore memory）

Amazon Bedrock AgentCore memory により、Agentic Memory の実装を数行で実現できる。

- **短期メモリ**: チャット内容・セッション状態を保持
- **長期メモリ**: 自動メモリ抽出モジュールが生イベントから非同期に抽出する「セマンティック」「ユーザー設定」「要約」「エピソード記憶」等の情報
- メモリは **アクターID** と **セッションID** によって論理的に分離される

メモリリソースの作成例として、`MemoryManager.get_or_create_memory()` に `USER_PREFERENCE` 戦略・namespace（`/users/{actorId}/preferences`）を指定するコードが紹介された（`event_expiry_days=30` 等の設定含む）。

**Strands Hooks によるエージェントライフサイクル制御**: `AgentInitialized` → `BeforeInvocation`/`MessageAdded` → `BeforeModelCall`/`AfterModelCall` → `BeforeToolCall`/`AfterToolCall` → `AfterInvocation` という一連のイベントにフックし、`MemoryHookProvider` クラスで `on_agent_initialized`（長期メモリの呼び出し）や `on_message_added`（メッセージ保存）のタイミングを制御する実装例が示された。

### Step 4: Runtime へのデプロイ（AgentCore runtime）

任意のモデル・任意のフレームワークでローカル構築したエージェントを、安全かつスケーラブルにホストする手順。

1. ローカルでエージェントを構築（`/ping`, `/invocations` エンドポイントと `requirements.txt` を追加）
2. コンテナイメージをビルドし Amazon ECR にプッシュ（AgentCore CLI 経由も可）
3. コンテナイメージ・プロトコル設定・ネットワーク設定を指定して AgentCore runtime 上にエージェントを作成
4. エンドポイントを作成
5. アプリからエンドポイントを呼び出し

各セッションは分離された microVM 上で実行され、セッション終了後に削除される点が特徴として強調された。

### Step 5: 認証認可機能の追加（AgentCore identity）

Amazon Bedrock AgentCore identity を用いたインバウンド認証の実装フロー。

1. ユーザーがアプリケーションにログインし、IdP（本セッションでは Amazon Cognito）からログイン情報・IdPトークンを取得
2. アプリケーションがIdPトークンを渡してエージェントを呼び出し
3. AgentCore runtime が AgentCore identity にIdPトークンの検証をリクエスト
4. AgentCore identity がIdPへユーザー検証をリクエストし、レスポンスを受領
5. AgentCore identity が AWS Workload Access Token を返却（トークン交換）
6. エージェントが実行される

## 4. まとめ

冒頭の4つの課題への回答として整理された。

1. **的外れな回答** → Amazon Bedrock Knowledge Bases と AgentCore memory を組み合わせた Agentic RAG の構築で改善
2. **コストとレイテンシーの増大** → AgentCore memory の長期メモリ戦略により、効率よく必要な情報だけをコンテキストに投入できる
3. **本番環境への展開** → AgentCore runtime と identity により、セッション分離されたmicroVM・自動スケーリング・JWT認証認可を備えたエンタープライズ対応のデプロイ基盤が得られる
4. （会話を覚えていない、についても AgentCore memory により解消）

### 本セッションのまとめ（4点）

1. **Context Engineering を用いたRAGの再定義**: ドキュメント検索＋ツールキュレーション＋メモリを統合し、エージェントに最適なコンテキストを提供
2. **短期メモリ＋長期メモリでコンテキストを圧縮**: AgentCore memory が会話から重要情報を抽出・統合し、コスト削減・レイテンシー改善・精度向上を同時に実現
3. **数行のコードでAgentic RAGを構築**: Strands Agents + Tools / Hooks で、Knowledge Bases とメモリを統合したエージェントを迅速に実装
4. **AgentCore runtime + identity で本番環境へ**: セッション分離されたmicroVM、自動スケーリング、JWT認証認可でエンタープライズ対応のデプロイ基盤を提供

本セッションで構築したAgentic RAGシステムの実装コードはGitHubで公開されている（具体的なURLはQRコード画像のため詳細不明）。
