# HackerNews はリンク先本文を要約せず「議論ダイジェスト」として扱う

HN は第三者ドメインの記事へのリンク集であり本文を持たない。本文を要約するには第三者ページを fetch して抽出する必要があるが、Workers ランタイムには `DOMParser` が無く readability 系がそのまま動かない。一方で HN のコメント論点要約は記事の概要を十分に代替できる。よって HN の記事エントリは **タイトル＋コメント論点要約** で構成し、リンク先本文の抽出・要約は当面行わない。

## Considered Options

- **linkedom + readability を持ち込む**: 純JSのDOMで本格抽出。バンドルが無料プラン1MB制限を圧迫し（※サイズ根拠は失効。下記 Amendment 参照）、ペイウォール/JSレンダリングには無力。
- **HTMLRewriter でセレクタ抜き取り**: Workers ネイティブだがストリーミングゆえ全文スコアリング不可。セマンティックHTMLのサイトでしか効かない。
- **本文抽出をしない（採用）**: HN はコメント論点要約で中身を伝える。第三者ページに触れない。

## Consequences

- Phase 2 の入力は HN では「タイトル＋コメント群」。本文要約 `summarizeArticle()` は発行元フィード（Zenn/AWS Blog, Phase 7）で初めて活きる。
- 本文抽出（HTMLRewriter）は独立した拡張フェーズへ後ろ倒し。HN の記事エントリは「リンク先の要約」ではなく「議論の要旨」という性格になる（catnose 氏プロダクトとの差別化軸とも整合）。
- コメントが薄い記事ではエントリも薄くなる。これは許容する。

## Amendment (2026-10-20)

**バンドルサイズを根拠とした却下理由は失効した。結論（本文抽出をしない）は維持する。**

- 2026-09-04 に Workers のサイズ上限が変更され、圧縮サイズ上限（Free 3 MB / Paid 10 MB）が撤廃されて全プラン一律・非圧縮 64 MiB になった（[changelog](https://developers.cloudflare.com/changelog/post/2026-09-04-increased-worker-size-limit/), [Limits](https://developers.cloudflare.com/workers/platform/limits/)）。linkedom + readability 程度のバンドルは制約にならない。
- なお本文の「無料プラン1MB」は記述時点で既に不正確だった（変更直前の Free 上限は圧縮 3 MB）。
- `DOMParser` 不在は linkedom で補える前提の話であり、linkedom 案の却下理由ではない（抽出に別ライブラリが要るというコストの説明）。
- 却下を支える根拠は以下で、いずれも有効なまま:
  - ペイウォール/JSレンダリングのページには抽出が効かない。
  - 本文抽出をしなければ第三者ページを fetch せずに済み、robots.txt / 利用規約尊重の不変条件と衝突しない。
  - HN のコメント論点要約が記事の概要を十分に代替できる。
