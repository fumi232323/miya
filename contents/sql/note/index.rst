.. article::
   :date: 2018-11-06
   :title: SQL のメモ
   :category: sql
   :tags:
   :canonical_url: /sql/note/
   :draft: false

==========
SQL のメモ
==========


検索 CASE 式でしか NULL の判定はできない
========================================

- 検索 CASE 式: NULL 判定できる

  .. code-block:: sql

    CASE
      WHEN d.name IS NULL THEN
        CASE d.type
          WHEN 'husky' THEN 'risa'
          WHEN 'shiba' THEN 'natsu'
          WHEN 'border collie' THEN 'chon'
          WHEN 'labrador retriever' THEN 'robo'
          ELSE NULL
        END
      ELSE d.name
    END AS name


- 簡易 CASE 式: NULL 判定できない

  .. code-block:: sql

    CASE d.name
      WHEN NULL THEN
        CASE d.type
          WHEN 'husky' THEN 'risa'
          WHEN 'shiba' THEN 'natsu'
          WHEN 'border collie' THEN 'chon'
          WHEN 'labrador retriever' THEN 'robo'
          ELSE NULL
        END
      ELSE d.name
    END AS name

  - ``d.name`` が NULL の場合は、 ``FALSE`` と評価されてしまう ( ``ELSE`` 句に入ってしまう)
  - 使いようによっては、``FALSE`` と評価されてしまうことを逆に利用できるかも....


COUNT(DISTINCT CASE WHEN ...)
==============================
こういう書き方ができる

.. code-block:: sql

  COUNT(
    DISTINCT CASE
      WHEN l.type = 2 THEN l.user_id
      ELSE NULL
    END
  ) as num_unique_users,

SELECT文の評価順序
====================

- これがわかりやすい (いつも見せていただいています、ありがとうございます)

  https://qiita.com/suzukito/items/edcd00e680186f2930a8
