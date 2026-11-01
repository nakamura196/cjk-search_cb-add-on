# cjk-search_cb-add-on

CollectionBuilder-CSV Add-on: CJK Search

This repository provides template files to search a CollectionBuilder-CSV collection in Chinese, Japanese and Korean. The files in this directory structure can be copied directly into a CB-CSV repository to add the new features.

*note: only compatible with CB-CSV out of the box, not other templates.*

[日本語版はこちら (README.ja.md)](README.ja.md)

## CJK Search

CollectionBuilder's search page uses [lunr.js](https://lunrjs.com/) with its English settings. lunr removes every character that is not a Latin letter or digit from the ends of each word, and kanji, kana and hangul are not, so a Chinese, Japanese or Korean query finds **nothing**, even when it is an exact title. The browse page's filter box does find exact text, but not text written another way (旧字体 and 新字体, hiragana and katakana, full-width letters).

This add-on replaces both with the rules of the [cjk_index](https://github.com/nakamura196/cjk_index) gem ([RubyGems](https://rubygems.org/gems/cjk_index)):

- **Search page**: the search index is built when the site is built, from the fields marked `index` in `_data/config-search.csv`. Chinese, Japanese and Korean text matches anywhere inside a field (大學 finds 東京帝國大學), with no dictionary. The matched part of each result is marked.
- **Browse page**: the filter box compares text with the same rules. Date ranges, sorting and facets work as before.
- **Both**: text written in different forms is treated as the same: old and new forms of kanji (圖 / 図), hiragana and katakana, full-width and half-width letters, 々. Traditional and Simplified Chinese, and old Hangul, can be added (see [Variant forms](#variant-forms)). English and other languages still work: the last word matches as a beginning, and words of five or more letters allow one wrong letter.

[Try the difference](https://nakamura196.github.io/cb-ja-demo/compare.html) (the same data searched with the stock search and with cjk_index), or watch the [2-minute video](https://www.youtube.com/watch?v=me46kC3JU8Q).

This cb-add-on contains:

| File | What it does |
| --- | --- |
| `_layouts/search-cjk.html` | search page layout: the `search` layout, with search options for CJK text |
| `_includes/js/lunr-cjk-js.html` | search page script: loads the prebuilt index, shows results like the stock page, with the matches marked |
| `_layouts/browse-cjk.html` | browse page layout: the `browse` layout, with the script below |
| `_includes/js/browse-cjk-js.html` | browse page script: CollectionBuilder's own `js/browse-js.html`, unchanged, then a CJK-aware text comparison for the filter box |

The stock `search` and `browse` layouts and their scripts are not changed. Switching back is one line per page.

## How To Use

Add to CB-CSV project:

- Start a CollectionBuilder-CSV project (or use your existing one!).
- Download this add-on to your computer (click "Code" > "Download ZIP", then unzip package).
- Copy all contents in the add-on (all files and folders except "README.md", "README.ja.md" and "LICENSE") and paste into your CollectionBuilder-CSV project. The folder structure in this add-on should match up with your CB-CSV template, adding new files in the correct places (do not delete or overwrite anything in your existing CB project!).
- Make the edits in [Changes to Existing Files](#changes-to-existing-files) below (`Gemfile`, `_config.yml`, `pages/search.md`, `pages/browse.md`), then run `bundle install`.

Build the site as usual. The build log shows a line like `cjk_index: assets/cjk-index/search.json (39 documents)`.

### Use on the Search Page

In `pages/search.md`, change `layout: search` to `layout: search-cjk`. The search box in the navigation bar sends its query to this page, so it uses the new search too.

The search box takes plain text. Words separated by spaces must all occur (`東京 大學` finds items that contain both). lunr's operators (`title:foo`, `+`, `-`, `*`, `~`, `^`) are not used; the "Search Options" button on the page explains this to your users.

### Use on the Browse Page

In `pages/browse.md`, change `layout: browse` to `layout: browse-cjk`. This works with or without `faceted-search` and `advanced-search` in `_data/theme.yml`. Links from item pages to the browse page (`browse.html#subject:...`) use it too.

The browse page needs no index: it compares the items it already has, in the browser.

### Preparing CJK Metadata

Most of the work is done by the rules above, but they cannot know how a word is read or what it means. A few habits in the metadata CSV help your users find items:

- **Write as the source does.** Old forms of kanji (圖, 學, 國) can stay as they are: a query in new forms finds them, and the result shows the original form with the match marked. Do not modernize titles only for search.
- **Add readings in their own column.** cjk_index matches characters, not readings: `とうきょう` does not find `東京`, and `Tokyo` does not find `東京`. If your users will type readings or romanization, add a column (for example `title_kana` or `title_romaji`) to the metadata CSV, and add a row for it in `_data/config-search.csv` with `index` `true` and `display` `false`:

  ```
  field,index,display
  title,true,true
  title_kana,true,false
  ```

  Katakana and hiragana are the same in search, so either can be used for readings.
- **Be consistent about spaces inside names.** A space splits a query into words that must all occur, but in the index a space also ends a run of CJK text. `東京 帝國大學` (with a space) is found by `帝國大學` and by `東京 大學`, but not by `東京帝國`. Write names the way your users will type them, usually without spaces.
- **Separate multiple values with `;`**, as CollectionBuilder expects (`地図;歴史`). The browse filter and facets split on it.
- **Long text makes a larger index.** Every field marked `index` in `_data/config-search.csv` goes into `assets/cjk-index/search.json`, which the search page downloads. Titles, names, subjects and short descriptions stay small (69 records with descriptions and page transcriptions: 44 KB compressed). Think before marking a long transcript field `index`.

### Variant Forms

The `variants` setting in `_config.yml` chooses which forms are treated as the same:

| `variants` | Use for |
| --- | --- |
| `[ja]` (default) | Japanese: old and new forms of kanji (圖 / 図, 學 / 学) |
| `[ja, zh-safe]` | Japanese collections that should also match Simplified Chinese queries (图 finds 圖 and 図). Pairs that are different characters in Japanese (後 / 后) are not merged |
| `[zh]` or `[ja, zh]` | Chinese collections: all Traditional and Simplified pairs (实录 finds 實錄). Also merges some characters that are different in Japanese |
| add `ko` | Old Hangul stored in the Private Use Area (by HWP and the Hanyang fonts). Ordinary hangul needs no table |

For variants of your own (local forms, 異体字 in your catalog), add a file name, for example `variants: [ja, _variants.tsv]` for a file `_variants.tsv` in the project folder (the `_` keeps it out of the built site; do not put it in `_data/`, where Jekyll would read it as data). Each line is one group, separated by tabs, the form to search as first: `島	嶋	嶌`. The [cjk_index README](https://github.com/nakamura196/cjk_index#a-table-of-your-own) explains the format.

### Limits

- Matching is by characters, not by meaning: no readings (see above), no synonyms, no word stems in English (`maps` does not find `map`).
- A query must occur as written, apart from the variant forms: `東京大學` does not find `東京帝國大學`. Use spaces to search for parts: `東京 大學`.
- In search results, field values are shown as text. If your metadata contains HTML (links in descriptions), the tags are shown, not followed.
- The index is rebuilt at every build; edit the CSV and build as usual. For collections with long full text (thousands of pages), cjk_index can split the index into parts that are loaded only when needed; this add-on does not set that up (see "Large collections" in the [cjk_index README](https://github.com/nakamura196/cjk_index#large-collections-split-indexes)).
- The Korean rules have not yet been checked by a native reader.

## Changes to Existing Files

1. `Gemfile`, at the end:

   ```ruby
   # CJK search: search index at build time (cjk-search_cb-add-on)
   group :jekyll_plugins do
     gem 'cjk_index', '~> 1.1', require: 'cjk_index/jekyll'
   end
   ```

2. `_config.yml`, anywhere (for example at the end):

   ```yaml
   # CJK search: index for the search-cjk layout (cjk-search_cb-add-on)
   cjk_index:
     variants: [ja]
     indexes:
       - name: search
         preset: collectionbuilder
   ```

   This section is needed by both pages: it also writes the script `assets/cjk-index/cjk-index.js`. The `collectionbuilder` preset reads CollectionBuilder's own settings (the `index` fields of `_data/config-search.csv`, and `search-child-objects` in `_data/theme.yml`).

3. `pages/search.md`: `layout: search` → `layout: search-cjk`

4. `pages/browse.md`: `layout: browse` → `layout: browse-cjk`

To switch back, change the two `layout:` lines back. The Gemfile and `_config.yml` lines do no harm if left.

## Example

- [cb-ja-demo](https://nakamura196.github.io/cb-ja-demo/) ([repository](https://github.com/nakamura196/cb-ja-demo)): 69 items from the University of Tokyo in Japanese, Chinese and Korean, in Japanese and English. Try [实录](https://nakamura196.github.io/cb-ja-demo/search/?q=%E5%AE%9E%E5%BD%95) (Simplified Chinese; finds 實錄 and 実録) or [帝国大学](https://nakamura196.github.io/cb-ja-demo/search/?q=%E5%B8%9D%E5%9B%BD%E5%A4%A7%E5%AD%A6) (finds 帝國大學). The site uses the same gem and settings, with its own version of the search script that adds facets and a Japanese interface. [compare.html](https://nakamura196.github.io/cb-ja-demo/compare.html) shows the stock search and cjk_index side by side.

## License

Code: MIT (see [LICENSE](LICENSE)).
Documentation (this README and README.ja.md): [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).
The cjk_index gem is MIT; its variant tables come from Unicode (Unicode License v3), OpenCC (Apache-2.0) and hypua (Unlicense), see its [NOTICE](https://github.com/nakamura196/cjk_index/blob/main/NOTICE).
