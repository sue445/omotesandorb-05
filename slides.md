## 【今月のgem】index_shotgunを作った #omotesandorb
sue445

2015/10/01 表参道.rb #5

---
## 自己紹介
[![sue445](img/sue445.png)](https://twitter.com/sue445)

* [sue445](https://twitter.com/sue445)
* [株式会社ドリコム](http://www.drecom.co.jp/) 所属
* サーバサイド全般の雑用
  * インフラ、アプリ、ライブラリ、社内ツールetc
  * 最近はアプリの改善系がメイン
* TDDおじさん
* プリキュアおじさん

---
## 【今期の嫁】キュアトゥインクル
![cure_twinkle](img/cure_twinkle.png)

---
## 【本妻】キュアピース
![cure_peace](img/cure_peace.jpg)

---
## Agenda
* 前置き
* gemの紹介
* 使い方
* 重複index検出ロジック
* 不要なindexを消す理由
* まとめ

---

## 前置き
* DBの容量圧迫してたので削除できる不要indexの調査してた
* 本来ならアプリで発行されるSQLを全部explainをとって1つずつ精査すべきだが、大変そうだったので機械的に抽出するできるものを削除する方向にした

---
### 最初に作ったスクリプト
https://gist.github.com/sue445/f890ea3fb5ef4fb5b9da

```sh
bundle exec rails r scripts/search_duplicate_indexes.rb
```

---
### pt-duplicate-key-checker
* これを作った後に WEB+DB PRESS Vol.88 を読んで全く同じ用途の [pt-duplicate-key-checker](https://www.percona.com/doc/percona-toolkit/2.1/pt-duplicate-key-checker.html) があることを知ったｗ
  * 実際よかった
  * 結果もだいたい同じ
* 詳しいこと：[Macでpercona-toolkitを使う方法 - くりにっき](http://sue445.hatenablog.com/entry/2015/09/05/200316)

---
## index_shotgun :fire: :gun: :cop: 
https://github.com/sue445/index_shotgun

* さっきのスクリプトをベースにした、重複indexを抽出するためのgemです
* active_recordベースなので、[pt-duplicate-key-checker](https://www.percona.com/doc/percona-toolkit/2.1/pt-duplicate-key-checker.html) とは違いMySQL以外でも使えます
  * そのDBで本当に重複indexであるか妥当かどうかは厳密には精査していない
* 名前の由来は [SQLアンチパターン](http://www.oreilly.co.jp/books/9784873115894/) の1つの「インデックスショットガン（闇雲インデックス）」です

---
## 使い方（Gemfileに書く方法）
```ruby
group :development do
  gem 'index_shotgun'
end
```

```sh
rake index_shotgun:fire
```

タスク名が ~~厨二病~~ カッコいい :fire:

---
### 実行結果

```sh
# =============================
# user_stocks
# =============================
# index_user_stocks_on_user_id is a left-prefix of index_user_stocks_on_user_id_and_article_id
# To remove this duplicate index, execute:
ALTER TABLE `user_stocks` DROP INDEX `index_user_stocks_on_user_id`;
# =============================
# user_stocks
# =============================
# index_user_stocks_on_user_id_and_article_id_and_already_read has unnecessary column already_read (index_user_stocks_on_user_id_and_article_id is unique index!)
# To remove this duplicate index, execute:
ALTER TABLE `user_stocks` DROP INDEX `index_user_stocks_on_user_id_and_article_id_and_already_read`;
# =============================
# user_stocks
# =============================
# index_user_stocks_on_user_id is a left-prefix of index_user_stocks_on_user_id_and_article_id_and_already_read
# To remove this duplicate index, execute:
ALTER TABLE `user_stocks` DROP INDEX `index_user_stocks_on_user_id`;

# ########################################################################
# Summary of indexes
# ########################################################################
# Total Duplicate Indexes  3
# Total Indexes            6
# Total Tables             5
```
---

## 使い方（コマンドラインから使う方法）
`gem install` して

```sh
index_shotgun mysql --database=DATABASE
index_shotgun postgresql --database=DATABASE
index_shotgun sqlite3 --database=DATABASE
```

* 出力形式はさっきと同じ
* 別途 mysql2, pg, sqlite3などのgemが必要

---
## 重複index検出ロジック
基本は前方一致

* 同じテーブルに [column_1, column_2] と [column_1, column_2, column_3] のindexが混在していた時は前者を削除（多い方の前方一致でまかなえるため）
* 前者がunique indexだった場合、cardinalityによっては後者のindexが削除できる
  * 例：column_3がbooleanだったので後者を消した（NULLを考慮しても3パターンしか値をとらないｗ）

---
## 不要なindexを消す理由
* ストレージの容量を喰う
  * 2.6億レコードのテーブルで不要なindexを1つ消すだけで18GB削減されたｗ
* INSERTやUPDATE時にそのindexも更新されるのでその分遅くなる
* （MySQLだと）同じクエリでも意図しないindexが使われて遅くなることがある
  * user_idによって使われるindexが変わって遅くなることがあった

---
## まとめ
不要なindexはこまめにチェックしよう

<!--
  disable uppercase
  via. http://srz-zumix.blogspot.jp/2014/09/revealjs-markdown.html
-->
<style type="text/css">
    .reveal h1,
    .reveal h2,
    .reveal h3,
    .reveal h4,
    .reveal h5,
    .reveal h6 {
      text-transform: none;
    }
</style>
