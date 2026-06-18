# HN コメント取得は公式 Firebase ではなく Algolia HN API を使う

HN 公式 Firebase API はコメントを `kids` に従って1件ずつ個別 fetch する必要があり、記事10件分のコメントツリーを取ると外部 fetch が容易に100回を超える。Workers 無料プランの**外部サブリクエスト上限は1実行あたり50**で、これに収まらない。Algolia HN API（`hn.algolia.com/api/v1/items/{id}`）は1リクエストで記事＋コメントツリー全体（本文付き・ネスト込み）を返すため、記事ごと1回で済む。

よって **Firebase `topstories.json` で当日トップのランキングと記事IDを取得し、各記事の本文・コメントは Algolia `items/{id}` で一括取得**するハイブリッド構成を採る。合計の外部 fetch は「1 ＋ 記事数」で、50制限に余裕を持って収まる。

## Consequences

- ランキングの権威（「今のトップ」）は Firebase、スレッド本体は Algolia という役割分担になる。
- Algolia は検索インデックスのミラーで反映に若干の遅延があり得るが、日次バッチのため実用上問題ない。
- 非公式 API への依存が増える。仕様変更リスクは取得層アダプタ（Phase 7）で吸収する。
