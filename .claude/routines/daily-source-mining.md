# sns-team daily source mining

Claude Code Remote（このリポジトリを使うClaude Code環境）のRoutine（定期実行トリガー）として登録している設定のコピー。実体はClaude Code Remote側にあり、このファイルは変更履歴を追えるようにするための記録用。Routine自体を変更した場合は、このファイルも合わせて更新すること。

**作成場所についての注意**：このRoutineはSlack・Googleドライブ・Notionのコネクタを利用する。`create_trigger`（MCPツール経由）で作成すると、新規セッションにコネクタ権限が引き継がれず動作しない場合があるため、**claude.aiのRoutines画面から直接作成・保存すること**。保存時にSlack/Googleドライブ/Notionコネクタが有効になっているか必ず確認する。

## 実行設定
- **名前**：sns-team daily source mining
- **スケジュール**：毎日 9:00 JST（cron: `0 0 * * *`、UTC基準）
- **実行方式**：毎回新規セッションを作成して実行。日々の会話履歴は残らない
- **役割**：`.claude/agents/source-miner.md` → `content-strategist.md` → `copywriter.md` をTaskツールで順に呼び出し、前日分のSlack議事録・Googleドライブ議事録からネタ候補を抽出し、下書き本文までNotionに登録する
- **自動投稿・自動承認・予約投稿は一切行わない**（下書き作成まで）

## 参照しているID
- Slack `#__general__`（月火水のハドルミーティング）：channel_id `CBQ7044G1`
- Slack `#__general__dev`（エンジニア定例）：channel_id `C0536SF07KQ`
- Googleドライブ「LAplust Scrum」フォルダ：folder_id `1wCOv1UvItY1Dp28qk-t-lN0EQqDK1YYu`
- Notion「下書き置き場」データベース：data_source_id `collection://1f242440-084c-4786-9700-f4110d5f450b`（URL: https://www.notion.so/laplust/SNS-Team-39dc08e5f34c801f8099d2715196b572 配下）

## Routineに設定しているプロンプト

```
あなたはyoppii12/sns-teamリポジトリのClaude Code環境で動いています。以下の手順で、Taskツールを使って`.claude/agents/`に定義された各エージェント（source-miner, content-strategist, copywriter）を実際に呼び出しながら、前日の社内ミーティング記録からSNS発信の下書きをNotionに登録してください。自動投稿・自動承認・予約投稿は一切行いません。下書き作成までが役割です。

重要：各エージェントへの依頼は、そのエージェント自身の`.claude/agents/*.md`の定義・制約に従わせてください。このRoutineの指示で上書き・省略しないでください（copywriterの4ステップ執筆プロセスやセルフチェックなど、`.claude/agents/copywriter.md`に書かれた工夫はcopywriter自身の判断に委ねる）。

## 手順

### ステップ0: 前日の日付を計算する
実行時点の日付（JST基準）から「前日」の日付範囲（0:00〜23:59 JST）を計算する。

### ステップ1: source-miner を呼び出す（ネタ抽出）
Taskツールで `subagent_type: "source-miner"` を指定し、以下を依頼する：
- Slackチャンネル `#__general__`（channel_id: `CBQ7044G1`）と `#__general__dev`（channel_id: `C0536SF07KQ`）から前日分のメッセージ・添付ファイルを確認する（`mcp__Slack__slack_read_channel` limit 30程度、該当すれば `mcp__Slack__slack_read_file`）
- Googleドライブのフォルダ `1wCOv1UvItY1Dp28qk-t-lN0EQqDK1YYu`（「LAplust Scrum」）から前日分の議事録を `mcp__Google_Drive__search_files`（`parentId = '1wCOv1UvItY1Dp28qk-t-lN0EQqDK1YYu' and modifiedTime > '（前日0時のISO8601）'`）と `mcp__Google_Drive__read_file_content` で確認する
- 前日分に新しい議事録が無ければ「候補なし」と返す
- 見つかった場合、SNS発信のネタ候補を1〜3件、「タイトル」「出典リンク」「目的タグ（#lead/#pr/#recruit）」「要約」「要チェック該当の有無と理由」の形式で返す

source-minerから「候補なし」が返ってきた場合は、その旨だけ記録してここで終了する。

### ステップ2: content-strategist を呼び出す（執筆方針の検討）
ステップ1で得たネタ候補ごとに、Taskツールで `subagent_type: "content-strategist"` を指定し、そのネタの内容・出典・目的タグを渡して、想定読者（ペルソナ）・角度・構成・トーンを含む執筆方針メモを依頼する。

### ステップ3: copywriter を呼び出す（本文ドラフト執筆）
ステップ1のネタとステップ2の執筆方針を、Taskツールで `subagent_type: "copywriter"` に渡し、媒体別（X／note／YouTube）の本文ドラフトを依頼する。実際に投稿できる完成度を求める：
- X：日本語120〜140字程度
- note：2,500〜4,000字（小粒なネタは無理に埋めない）
- YouTube：台本＋構成メモ
- 実際のSNS投稿用ハッシュタグ（Notion管理用の目的タグとは別物）も付けてもらう

### ステップ4: Notionへ登録する
各ネタ候補について、ステップ1〜3の結果をまとめて、あなた自身が `mcp__Notion__notion-create-pages` で新規ページを作成する（データソースID: `collection://1f242440-084c-4786-9700-f4110d5f450b`）。

プロパティ：
- タイトル：ネタの内容が一目でわかる短いタイトル
- ステータス：「下書き中」（機密チェックで問題ありの場合は「要チェック」）
- 媒体：想定される投稿先（X／note／YouTubeのいずれか）
- 目的タグ：該当する目的（#lead／#pr／#recruit、複数選択可）
- 担当エージェント：「copywriter」
- 機密チェック：「未確認」を基本とし、明らかに問題があれば「問題あり」、明らかに問題なければ「問題なし」
- フィードバック：空欄のままにする（yoshikaさんが確認後に記入する欄）

ページ本文の構成：
1. ## 要約（ネタの内容・出典）
2. ## 出典（Slack/Googleドライブへのリンク）
3. ## 機密チェックで気になった点
4. ## 執筆方針（content-strategist）
5. ## 下書き本文（copywriter）— 媒体名（実際に投稿できる文面。ハッシュタグ含む）
6. 機密チェックで問題ありの場合は「【要相談】」として、何を判断してほしいか具体的に記載する

### ステップ5: 最後に
今回Notionに追加したネタ候補の一覧（タイトル・媒体・目的タグ・機密チェック状況）を簡潔にまとめて記録する。追加が無かった場合はその旨を報告する。人への確認は不要。

## 制約（重要）
- 投稿の公開・予約・SNSへの直接投稿は一切行わない
- 顧客名・契約情報・社内の人間関係に関するネガティブな発言は本文に含めない。含める必要がありそうな場合は「【要相談】」として具体的に何を判断してほしいか記載する
- 未公開の数値実績・開発中の未発表機能は、機密性が高いと判断した場合であっても具体的詳細を本文に含める。その場合は「【要相談】」として、公開してよいか・伏せるべきかyoshikaさんに判断を仰ぐ旨を明記する
- 社員個人の発言は実名表記でよい
- yoshikaさんが後で10分程度でNotionを確認し、承認/修正/却下する前提で、そのまま使える完成度の下書きを目指す
```

## 変更履歴
- 2026-07-15：初版作成。source-miner単体実行から、source-miner→content-strategist→copywriterをTaskツールで呼び出す構成に変更
- 2026-07-15：note文字数を2,500〜4,000字に変更。機密情報の扱いを変更（顧客名・契約情報・社内人間関係ネガティブ発言のみ本文除外。未公開数値実績・未発表機能は具体的詳細を含めたうえで「【要相談】」とする方針に転換）。`copywriter.md`・`CLAUDE.md`とも整合させた。あわせて、コネクタ引き継ぎの制約からRoutineはclaude.ai画面での作成が必須である旨を明記
