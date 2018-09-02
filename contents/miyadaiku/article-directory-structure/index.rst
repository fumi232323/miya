.. article::
   :date: 2018-07-01
   :title: article の置き方
   :category: miyadaiku
   :tags: タグ1, タグ2
   :canonical_url: /miyadaiku/article-directory-structure/
   :draft: false

===================
article の置き方
===================

article の置き方
=========================
- 最初に決めた方がよい。
- 分けないと後々何が何だかわからなくなるらしい。
- 年ごと > カテゴリごとにディレクトリを作る。
- article には、 ``index.rst`` というファイル名をつける。
- article ごとに、 ``canonical_url`` プロパティを設定する。

  e.g. ``contents/2018/sample-article1/index.rst``

  .. code-block:: python

    :canonical_url: /2018/sample-article1/

- article 本体とそこから参照する画像は同じディレクトリに配置する。
