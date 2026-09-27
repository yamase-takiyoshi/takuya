The競馬 v6.1.13 GitHub自動退避 セットアップ

目的
- JRA/NARのオッズ初回配信を5秒間隔で検知
- 配信開始後は全8券種を5秒刻みで保存
- 発走時刻+3分で収集停止
- GitHubへ「初回フル + 以後差分」JSONとして自動退避
- GitHub保存成功後、Durable Object内の時系列データを削除

GitHub保存先
Owner : yamase-takiyoshi
Repo  : takuya
Branch: main
Path  : odds-data

Cloudflare Worker
既存の the-keiba-timeline-collector Worker のコードを
cloudflare/worker_timeline_v6.1.13_NAROfficialBatch_OddsRelease5s_Post3m_GitHubArchive_All8Markets.js
へ差し替えて Deploy します。

必須 Binding
Durable Object
Variable name : TIMELINE
Class         : OddsTimelineCollector

Cron Trigger
* * * * *
（1分ごと。当日+翌日のmanifestを確認します）

Cloudflare Variables / Secrets

[Variables]
GITHUB_OWNER     = yamase-takiyoshi
GITHUB_REPO      = takuya
GITHUB_BRANCH    = main
GITHUB_BASE_PATH = odds-data

UPSTREAM_PROXY_URL = 既存のThe競馬プロキシWorker URL
MANIFEST_SOURCE_URL = /cron-status?date=YYYY-MM-DD が使えるWorker URL

任意:
ODDS_ARM_HOURS = 36
RELEASE_PROBE_MS = 5000

[Secrets]
GITHUB_TOKEN = 新しく作成したGitHubアクセストークン
TIMELINE_TOKEN = 任意の十分長い秘密文字列

重要:
- GITHUB_TOKENをGitHubのファイル、index.html、JavaScriptへ直接書かない。
- CloudflareのVariables and Secretsで「Secret」として保存する。
- チャット等に貼った古いトークンはRevokeし、新しいトークンを使う。
- Fine-grained tokenは、Repository accessをtakuyaだけにし、
  Repository permissions > Contents を Read and write にする。

保存例
odds-data/
  2026/
    09/
      28/
        JRA/
          <race_id>.json
        NAR/
          <race_id>.json

動作確認
1) Deploy後:
   https://the-keiba-timeline-collector.nitta-katsuhiko.workers.dev/health

2) 正常時の主な項目:
   github_archive: true
   github_repo_configured: true
   github_branch: "main"
   github_base_path: "odds-data"
   stop_mode: "POST_PLUS_3M"
   archive_format: "first_full_then_delta"

3) レース終了後、takuyaリポジトリの odds-data/ 以下にJSONが作られることを確認。

手動アーカイブAPI
POST /timeline/archive?race_id=<race_id>
TIMELINE_TOKENを設定した場合:
X-Timeline-Token: <TIMELINE_TOKEN>

注意
- /timeline/stop は単に停止するAPIです。通常は発走+3分の自動停止に任せます。
- GitHubへの保存に失敗した場合はarchive_pending/archive_errorで状態を確認できます。
- GITHUB_TOKEN / OWNER / REPO のどれかが欠けるとGitHub退避は実行されません。
