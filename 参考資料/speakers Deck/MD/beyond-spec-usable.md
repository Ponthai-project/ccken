# 仕様通り動くの先へ。Claude Codeで「使える」を検証する

- 登壇: gota（@gota_bara / GitHub: @gotalab）／AIエージェント開発・データプロダクト
- 場: Claude Code Meetup（2026.04.10）
- 作: cc-sdd（3,000+ ⭐）/ uxaudit / skillport
- 元ファイル: `beyond-spec-usable.pdf`（25p）

## 宣伝：cc-sdd v3.0

specを適切に切って承認したら、あとは長時間自律で実装する仕組み。

- **境界をfirst classに**: `/kiro-discovery` でspec間の境界と依存を定義。チーム間の分業が自然にできる。
- **Skillsとして配布**: Claude Code / Codex / Cursor / GitHub Copilot 含む全8エージェント対応。
- **長時間の自律実装**: `/kiro-impl` でhooksなしに自律で回り続ける（タスクの状態遷移を工夫して実現、誰でも変更できるシンプルな構造を優先）。
- ループの中身: 1タスクごとにfreshなimplementerがTDD（RED→GREEN）→独立reviewerが別コンテキストでレビュー→2回rejectでauto-debug。学習は`tasks.md`のImplementation Notesで次タスクに伝搬。タスクごとにコミットされ途中で止まっても安全に再開。

## 「仕様通り作れる時代」は来つつある

- 揃えるもの: 実行可能な粒度まで落としたplan、明確なアーキテクチャと境界設計、実行可能な制約（CLAUDE.md / hooks / subagents）、検証環境（lint / test / E2E）。
- 使える道具: Plan Mode（**Explore first, then plan, then code**）、UltraPlan、plugins（Superpowers / Compound Engineering / feature-dev 〈Anthropic公式〉）。
- 仕事をしながら2週間で、Notionクローン・orchestrator clone・イベント駆動開発ハーネス等が作れる。

## しかし「動く」と「使える」は別の話

| 動く | 使える |
|---|---|
| テストが通る／仕様を満たす／デモできる／CIがgreen | 触って目的が達成できる／コア体験がはっきり／次の行動が選べる／初見で始められる |

Anthropic社内でも出荷物の**7割以上はPRDなし**。ドキュメントを積むより先にプロトタイプを触って次を判断する（2週間以内のPJはPMを置かずエンジニアが兼任）。**使えるかは触るまで分からない**（Amol Avasare, Head of Growth, Anthropic）。

### 「動くけど使えない」の4例

1. **伝わらない**: 操作は通るが成功フィードバックがない
2. **ぼやける**: 16機能あるがコア体験がぼやける
3. **見つからない**: ボタンはDOMにあるが画面で見つけられない
4. **始まらない**: フォーム送信はできるが初見で始められない

## 課題：速度×品質×量、人手ではどれか必ず落ちる

前提は毎週変わる（仕様を書ききる前にモデルが進化）→速度が命。動くだけでは判断できない→品質も落とせない。1個に人手を挟むと量も速度も止まる。→**人手を極限まで減らして、使えるものを速く出す仕組みが要る**。

## 自律ハーネス：音声で話す→自律実行1〜3時間→使えるプロトタイプ

4つのsubagentで構成（Claude Code の Stop hook / SubagentStop hook でタスクが終わるまで自律実行）。設計思想はAnthropic「Harness design for long-running apps」に近い。

| Subagent | モデル/ツール | 役割 |
|---|---|---|
| Planner | Opus / Read+WebSearch | WHATだけ書く。HOWはBuilderに任せる |
| Builder | Opus / 全ツール | Vertical Slice先行。環境NGで強制STOP |
| Evaluator | Opus / Playwright MCP | 機能検証・4軸スコア。各軸7/10でpass |
| UX Reviewer | Opus / Computer Use | 体験検証・3軸スコア。各軸7/10でpass。スコア<7なら再ビルド |

### こだわり

1. **CredoをSkillとして全subagentに注入**（5原則）: ①Core First, Polish Later ②Wire Before You Decorate ③No Dead Code ④The Spec Is Law（AIは統計的平均に収束する。魂は意図的な制約に宿る）⑤Built to Grow（退屈で読めるコードが美しく不透明な抽象に勝つ）。
2. **手戻りを先に潰す**: Plannerが検証シナリオを先に全部切る（正常+異常+エッジをCore/Stretch分類、手順でなくゴール形式）。Builderは観測環境（lint/type/dev server/logs）を最初に繋ぎ、コアフローを縦に一本通す。
3. **Computer Useで触らせる**: DOMを直接叩けば動作検証はできるが「画面から見つけられるか」「次の行動を選べるか」はDOMでは検出できない。Computer Useはスクショ+マウス/キーボードで操作しUX Reviewerに組み込める。

### Computer UseベースのUX Reviewerの限界

遅い（スクショ→座標特定→操作の直列ループ）、シナリオが並列化できない、評価用途と自律性が合わない→結局回りきらない。

## uxaudit（Claude Code Plugin）

ユーザージャーニーを自動測定するプラグイン。`/uxaudit:uxaudit my-app --lang ja`。

- 評価対象は2種類: 全アプリ共通の不備（34項目: a11y / AI slop / Nielsen原則）＋そのプロジェクトのジャーニー（Scoutが洗い出し4軸判定）。判定はスクショだけ見る（コードもspecも見ない）。
- 仕組み: アプリ理解→早い→重いの順にチェック（静的スキャン〈ミリ秒〉→ブラウザ計測 Playwright〈秒〉→画像判定 LLM〈数十秒〉→ジャーニー判定〈分〉）→原因でまとめ優先度付き提案を2〜3個に絞る（dashboard.html）。
- 1軸でも落ちたらジャーニー全体を不合格。iterationを並べて何が直った/壊れたかをdiffで追える。

### テストのレイヤーと「触らないと分からない層」

| レイヤー | 問い | 代表ツール |
|---|---|---|
| Unit | 部品は仕様通り動くか | Jest / Vitest / pytest |
| Property-based | 不変条件はどんな入力でも成り立つか | fast-check / Hypothesis |
| Integration | 組み合わせても壊れないか | Jest / pytest |
| E2E | ユーザーフローは完走できるか | Playwright / Cypress |
| Visual / A11y | 見た目・操作可能性に破綻はないか | Percy / Chromatic / axe |
| **UX Audit** | 理解・判断・達成・回復できるか | **uxaudit** |
| Manual QA | 総合的に違和感はないか | humans |

CIをgreenにできる領域の先に「触らないと分からない層」があり、そこをuxauditで埋める。

## まとめ

1. **「動く」は解けつつある**（モデル進化＋Claude Codeで仕様通りに作れる）
2. **「使えるか」の評価を自動化した**（ハーネス＋uxauditで人間の介入を最小化しながらiterationを回す）
3. **「使い続けるか」は次の壁**（実ユーザーのログとオブザーバビリティが要る）
