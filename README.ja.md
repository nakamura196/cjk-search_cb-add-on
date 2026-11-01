# cjk-search_cb-add-on

CollectionBuilder-CSV のアドオン: 日中韓の検索（CJK Search）

CollectionBuilder-CSV のコレクションを、日本語・中国語・韓国語で検索できるようにするテンプレートファイルです。このフォルダ構成のまま CB-CSV のリポジトリにコピーすると、機能が加わります。

*注: そのまま使えるのは CB-CSV だけです。ほかのテンプレートには対応していません。*

[English (README.md)](README.md)

## CJK Search

CollectionBuilder の検索ページは [lunr.js](https://lunrjs.com/) を英語の設定で使っています。lunr は語の両端からラテン文字・数字以外の文字を取り除きます。漢字・かな・ハングルはすべて取り除かれるので、日本語・中国語・韓国語で検索すると、題名そのものを入れても **1 件も出ません**。一覧（Browse）ページの絞り込み欄は、書かれたとおりの文字列なら見つけますが、書き方の違うもの（旧字体と新字体、ひらがなとカタカナ、全角の英数字）は見つけません。

このアドオンは、この 2 つを [cjk_index](https://github.com/nakamura196/cjk_index) gem（[RubyGems](https://rubygems.org/gems/cjk_index)）の規則に置き換えます。

- **検索ページ**: サイトのビルド時に、`_data/config-search.csv` で `index` にした項目から検索用の索引を作ります。日中韓の文字は、項目のどこにあっても当たります（「大學」で「東京帝國大學」が当たる）。辞書は使いません。結果の一致箇所に印を付けます。
- **一覧ページ**: 絞り込み欄の文字の比べ方を同じ規則にします。日付の範囲、並べ替え、ファセットは今までどおりです。
- **両方に共通**: 書き方の違う文字を同じとみなします。旧字体と新字体（圖 / 図）、ひらがなとカタカナ、全角と半角、「々」。繁体字と簡体字、古いハングルも足せます（[異体字の扱い](#異体字の扱い)）。英語などもそのまま使えます。最後の語は前方一致になり、5 文字以上の語は 1 文字の打ち間違いを許します。

[違いを試す](https://nakamura196.github.io/cb-ja-demo/compare.html)（同じデータを元の検索と cjk_index で並べたもの）。[2 分の動画](https://www.youtube.com/watch?v=B1PO2l3iPHo)もあります。

このアドオンの中身:

| ファイル | 役割 |
| --- | --- |
| `_layouts/search-cjk.html` | 検索ページのレイアウト。`search` レイアウトと同じで、検索のしかたの説明を日中韓向けにしたもの |
| `_includes/js/lunr-cjk-js.html` | 検索ページのスクリプト。作っておいた索引を読み、元のページと同じ形で結果を出し、一致箇所に印を付ける |
| `_layouts/browse-cjk.html` | 一覧ページのレイアウト。`browse` レイアウトに下のスクリプトを付けたもの |
| `_includes/js/browse-cjk-js.html` | 一覧ページのスクリプト。CollectionBuilder 本体の `js/browse-js.html` をそのまま読み込み、絞り込み欄の文字の比べ方だけを日中韓に対応したものに差し替える |

本体の `search`・`browse` レイアウトとそのスクリプトは変えません。元に戻すときは、ページごとに 1 行を戻すだけです。

## How To Use（使い方）

CB-CSV のプロジェクトに足す:

- CollectionBuilder-CSV のプロジェクトを始めます（今あるものでもかまいません）。
- このアドオンを手元に落とします（「Code」>「Download ZIP」を押して展開）。
- アドオンの中身を、「README.md」「README.ja.md」「LICENSE」を除いてすべてコピーし、CollectionBuilder-CSV のプロジェクトに貼り付けます。フォルダ構成は CB-CSV と同じなので、新しいファイルが正しい場所に加わります（今あるプロジェクトのファイルは消したり上書きしたりしないでください）。
- 下の [既存ファイルの変更](#既存ファイルの変更changes-to-existing-files) の編集（`Gemfile`、`_config.yml`、`pages/search.md`、`pages/browse.md`）をしてから、`bundle install` を実行します。

いつもどおりビルドします。ビルドの記録に `cjk_index: assets/cjk-index/search.json (39 documents)` のような行が出ます。

### 検索ページで使う

`pages/search.md` の `layout: search` を `layout: search-cjk` に替えます。ナビゲーションバーの検索欄もこのページに送るので、そちらも新しい検索になります。

検索欄には、ふつうの文字を入れます。空白で区切った語は、すべてを含むものが当たります（「東京 大學」は両方を含むもの）。lunr の演算子（`title:foo`、`+`、`-`、`*`、`~`、`^`）は使いません。ページの「Search Options」ボタンで、利用者向けにこのことを説明しています。

### 一覧ページで使う

`pages/browse.md` の `layout: browse` を `layout: browse-cjk` に替えます。`_data/theme.yml` の `faceted-search`・`advanced-search` がオンでもオフでも動きます。資料ページから一覧ページへのリンク（`browse.html#subject:...`）にも効きます。

一覧ページは索引を使いません。ページが既に持っている資料を、ブラウザの中で比べます。

### 日中韓の目録の整え方

ほとんどは上の規則で吸収できますが、読み方や意味までは分かりません。目録（メタデータの CSV）を次のように作っておくと、利用者が資料を見つけやすくなります。

- **原資料のとおりに書く。** 旧字体（圖・學・國）はそのままでかまいません。新字体で検索しても当たり、結果には元の字体のまま一致箇所に印が付きます。検索のためだけに題名を新字体に直す必要はありません。
- **読みは別の列に入れる。** cjk_index は文字を比べるもので、読みは比べません。「とうきょう」でも「Tokyo」でも「東京」は当たりません。利用者が読みやローマ字で探しそうなら、目録の CSV に列（たとえば `title_kana`・`title_romaji`）を足し、`_data/config-search.csv` にその行を `index` は `true`、`display` は `false` で加えます。

  ```
  field,index,display
  title,true,true
  title_kana,true,false
  ```

  検索ではカタカナとひらがなは同じなので、読みはどちらで書いてもかまいません。
- **名前の中の空白をそろえる。** 検索語の空白は「すべてを含む」の区切りになりますが、索引の側でも空白は日中韓の文字のつながりを切ります。「東京 帝國大學」（空白あり）は「帝國大學」「東京 大學」では当たり、「東京帝國」では当たりません。利用者が打つとおりに、ふつうは空白なしで書きます。
- **複数の値は `;` で区切る**（`地図;歴史`）。CollectionBuilder の決まりどおりです。一覧の絞り込みとファセットはこれで分けます。
- **長い文章は索引を大きくする。** `_data/config-search.csv` で `index` にした項目はすべて `assets/cjk-index/search.json` に入り、検索ページはこれを読み込みます。題名・人名・件名・短い解題なら小さく済みます（解題とページごとの翻刻つきの 69 件で、圧縮して 44 KB）。長い翻刻の列を `index` にするときは、大きさを確かめてください。

### 異体字の扱い

`_config.yml` の `variants` で、どの字を同じとみなすかを選びます。

| `variants` | 向いている場合 |
| --- | --- |
| `[ja]`（既定） | 日本語。旧字体と新字体（圖 / 図、學 / 学） |
| `[ja, zh-safe]` | 簡体字でも探せるようにしたい日本語のコレクション（「图」で「圖」「図」が当たる）。日本語では別の字になる組（後 / 后）はまとめない |
| `[zh]` か `[ja, zh]` | 中国語のコレクション。繁体字と簡体字の組をすべてまとめる（「实录」で「實錄」が当たる）。日本語では別の字になるものも一部まとまる |
| `ko` を足す | 私用領域に入った古いハングル（HWP やハニャン（漢陽）フォントで入力したもの）。ふつうのハングルには表は要らない |

自分の目録に合わせた異体字（地域の字形、目録で使っている異体字）は、ファイル名を足します。たとえばプロジェクトのフォルダに `_variants.tsv` を置き、`variants: [ja, _variants.tsv]` とします（名前の先頭の `_` で、できあがったサイトには入りません。`_data/` に置くと Jekyll がデータとして読もうとするので避けます）。1 行が 1 組で、タブで区切り、検索でそろえる字を先頭に書きます: `島	嶋	嶌`。書式は [cjk_index の README](https://github.com/nakamura196/cjk_index/blob/main/docs/ja/README.md) にあります。

### 限界

- 比べるのは文字で、意味ではありません。読み（上を参照）、同義語、英語の語形変化（`maps` で `map` は当たらない）は扱いません。
- 検索語は、異体字の違いを除いて、書いたとおりに含まれている必要があります。「東京大學」では「東京帝國大學」は当たりません。部分で探すときは「東京 大學」のように空白で区切ります。
- 検索結果では、項目の値を文字として出します。目録に HTML（解題の中のリンクなど）があると、タグがそのまま見えます。
- 索引はビルドのたびに作り直します。CSV を直して、いつもどおりビルドしてください。長い全文（数千ページ）を持つコレクションでは、cjk_index は索引を分割し、要る部分だけを読み込めますが、このアドオンではその設定をしていません（[cjk_index の README](https://github.com/nakamura196/cjk_index#large-collections-split-indexes) の「Large collections」）。
- 韓国語の規則は、まだ韓国語を母語とする人に確かめてもらっていません。

## 既存ファイルの変更（Changes to Existing Files）

1. `Gemfile` の末尾に:

   ```ruby
   # CJK search: search index at build time (cjk-search_cb-add-on)
   group :jekyll_plugins do
     gem 'cjk_index', '~> 1.1', require: 'cjk_index/jekyll'
   end
   ```

2. `_config.yml` のどこか（たとえば末尾）に:

   ```yaml
   # CJK search: index for the search-cjk layout (cjk-search_cb-add-on)
   cjk_index:
     variants: [ja]
     indexes:
       - name: search
         preset: collectionbuilder
   ```

   この節は両方のページに要ります。スクリプト `assets/cjk-index/cjk-index.js` もここから書き出されるためです。`collectionbuilder` の設定は、CollectionBuilder 自身の設定（`_data/config-search.csv` の `index` の項目と、`_data/theme.yml` の `search-child-objects`）を読みます。

3. `pages/search.md`: `layout: search` → `layout: search-cjk`

4. `pages/browse.md`: `layout: browse` → `layout: browse-cjk`

元に戻すときは、2 つの `layout:` の行を戻します。`Gemfile` と `_config.yml` の行は残しておいても害はありません。

## 使用例（Example）

- [cb-ja-demo](https://nakamura196.github.io/cb-ja-demo/)（[リポジトリ](https://github.com/nakamura196/cb-ja-demo)）: 東京大学の日中韓の資料 69 件、日英 2 か国語。[实录](https://nakamura196.github.io/cb-ja-demo/search/?q=%E5%AE%9E%E5%BD%95)（簡体字。「實錄」「実録」が当たる）や[帝国大学](https://nakamura196.github.io/cb-ja-demo/search/?q=%E5%B8%9D%E5%9B%BD%E5%A4%A7%E5%AD%A6)（「帝國大學」が当たる）を試せます。同じ gem と設定を使い、検索のスクリプトはファセットと日本語の画面を足した独自版です。[compare.html](https://nakamura196.github.io/cb-ja-demo/compare.html) で元の検索と cjk_index を並べています。

## ライセンス（License）

コード: MIT（[LICENSE](LICENSE)）。
文書（この README と README.md）: [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/deed.ja)。
cjk_index gem は MIT です。異体字の表は Unicode（Unicode License v3）、OpenCC（Apache-2.0）、hypua（Unlicense）に由来します。[NOTICE](https://github.com/nakamura196/cjk_index/blob/main/NOTICE) を参照してください。
