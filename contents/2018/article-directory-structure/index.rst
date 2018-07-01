.. article::
   :date: 2018-07-01
   :title: contents ディレクトリ配下への article の置き方
   :category: blog
   :tags: タグ1, タグ2
   :canonical_url: /2018/article-directory-structure/
   :draft: false

==================================================
contents ディレクトリ配下への article の置き方
==================================================

note
=========================
- やる前に決めた方がよい。
- 年ごと、カテゴリごとにディレクトリを作る。
- 分けないと何が何だかわからなくなるらしい。
- article ごとに、 `canonical_url` プロパティを設定する。( http://pygments.org/docs/lexers/ )

  e.g.

  .. code-block:: python

    :canonical_url: /2018/sample-article1/

- article 本体とそこから参照する画像は同じディレクトリに配置する。
