# sns-team

SNS運用自動化 運営チームのプロジェクトリポジトリ。X（Twitter）・note・YouTubeでの発信を、Claude Codeのサブエージェントで支援する。

## 対象SNSと運用方針

- 対象媒体：X（Twitter）、note、YouTube
- 当面は1媒体1アカウントに統合（目的別・媒体別の分割はしない）
- noteはマガジン機能、YouTubeは再生リストで目的別に整理
- note・YouTubeへの投稿実行に公式APIはないため、「Notion下書き→人が手動でコピペ公開」が基本方針

## 3つの発信目的

すべての投稿に目的タグを付与する。

- `#lead` セールスリード獲得
- `#pr` 自社PR
- `#recruit` 採用

## チーム構成（7エージェント）

`.claude/agents/` 配下に定義。

| エージェント | 役割 | 権限 |
|---|---|---|
| content-strategist | 目的ポートフォリオ管理・ネタ選定 | 提案まで |
| source-miner | 社内一次情報からネタ抽出 | 自動 |
| copywriter | 媒体別下書き執筆 | 下書きまで |
| creative-producer | 画像・サムネ・動画編集案 | 提案まで |
| scheduler | 投稿カレンダー・予約投稿 | 承認済みのみ自動実行 |
| engagement | コメント・DM返信下書き | 下書きまで、送信は人 |
| reporter | 効果測定・比率の可視化 | 自動 |

## 運用フロー

```
source-miner（ネタ抽出）
  → content-strategist（ネタ選定・比率管理）
  → copywriter（媒体別下書き作成）
  → 人による秘密情報チェック・仕上げ
  → scheduler（承認済みのみ予約投稿）
```

公開判断、DM・コメントの送信、誹謗中傷への対応は必ず人が行う。エージェントに最終判断はさせない。

## ドキュメント

`docs/` 配下を参照。

- 設計書・意思決定の経緯
- 確定運用フロー
- セットアップ手順
- Windows移行手順書

プロジェクトの詳細な前提・KPI・秘密情報チェック基準は [`CLAUDE.md`](./CLAUDE.md) を参照。

## 使い方

各エージェントは明示的に名指しして依頼する（例：「content-strategistエージェントを使って〜」）。
