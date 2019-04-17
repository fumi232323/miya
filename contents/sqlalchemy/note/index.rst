.. article::
   :title: SQLAlchemy メモ
   :date: 2019-04-17
   :category: sqlalchemy
   :tags:
   :canonical_url: /sqlalchemy/note/
   :draft: false


===============
SQLAlchemy メモ
===============

.. raw:: html

  <details>
    <summary>目次</summary>

.. contents::

.. raw:: html

  </details>


リファレンス・ガイド
=====================

- `SQLAlchemy ORM <https://docs.sqlalchemy.org/en/13/orm/index.html>`_



モデル定義の String 型カラムの length
=====================================
リファレンスには `length` は安全に省略可能と書いてある (DDL を発行する機会がある場合は DB によって例外が発生するのでDBによっては必須) が指定するべきか否か

- リファレンス: https://docs.sqlalchemy.org/en/13/core/type_basics.html?highlight=string#sqlalchemy.types.String


DDL を発行する場合
-------------------
MySQL の場合は、指定するのが良い。

- MySQL はインデックスを貼れるデータ長の最大値があるため、通常は length 指定するのが良い
- length 指定ありだと varchar , length 指定なしだと text になる
- text にインデックスを貼る場合、「何文字目まで」という MySQL 特有な指定が必要になるのでだるい

DDL を発行しない場合
---------------------
DDLしないなら length 指定しないほうが無難。

- クエリしかしないなら気にすることはないはず

aodag さんありがとうございました。


SQLSoup
=======
DB から定義をとってくるのでアプリ側でのモデル定義が不要。すごい...

https://sqlsoup.readthedocs.io/en/latest/tutorial.html

aodag さんありがとうございました。
