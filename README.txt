The競馬 EV分析 v5.13.3 統合版

内容:
- index.html
  GitHub Pages / FTP にアップロードする本体HTMLです。
  カレンダー、JRA/NAR、開催場、レース選択、netkeiba取得、EV計算、推奨買い目を統合しています。

- worker.js
  Cloudflare Workers に貼り付けるプロキシです。
  netkeiba取得用の強化ヘッダ入りです。

使い方:
1. Cloudflare Workers に worker.js を貼り付けて Deploy。
2. 発行された workers.dev URL を index.html の「プロキシ」タブへ入力。
3. 「馬データ」タブで日付、JRA/NAR、開催場を選ぶ。
4. 「レース一覧」を押す。
5. レースを選び「レースデータ取得」。
6. 推奨買い目タブでEV計算結果を確認。

注意:
- netkeiba側のHTML変更やアクセス制限により取得できない場合があります。
- 出馬表ページからの馬名・馬番・オッズ・人気取得を優先しています。
- result.htmlの払戻精算は次段階の拡張対象です。
