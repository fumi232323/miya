.. article::
   :date: 2018-09-29
   :title: Django ORM のメモ
   :category: django
   :tags:
   :canonical_url: /django/django_orm/
   :draft: false


==================
Django ORM のメモ
==================


Django ORM
===========

aggregate と annotate
---------------------------------------------
``aggregate`` と ``annotate`` の違いがわかりやすい

- `Djangoの集計について <http://note.crohaco.net/2014/django-aggregate/>`_


prefetch_relatedの話
----------------------------------------
`Djangoでprefetch_relatedを使ってクエリ数を減らす <http://tokibito.hatenablog.com/entry/20140718/1405691738>`_


Q Object
----------------------------------------
`Q() objects <https://docs.djangoproject.com/ja/1.11/ref/models/querysets/#q-objects>`_

`Q オブジェクトを用いた複雑な検索 <https://docs.djangoproject.com/ja/1.11/topics/db/queries/#complex-lookups-with-q>`_

- クエリの検索条件を作ることができる
- ``OR`` を表現したいときはこれを使う

  - e.g.

    .. code-block:: python

      queryset = queryset.filter(
          Q(name__contains=q) |
          Q(username__contains=q)
      )


モデルクラス名を全部小文字にしたのに ``_set`` がつく
------------------------------------------------------------------
`Related objects reference <https://docs.djangoproject.com/ja/1.11/ref/models/relations/>`_

- ``_set`` というのは子テーブルのデータを参照する django の機能
- モデルクラス名を全部小文字にしたのに ``_set`` がつく


ForeignKey.on_delete
--------------------------
`ForeignKey.on_delete <https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.ForeignKey.on_delete>`_

- 6種類くらいあって、用途に応じて選べる
- Django 2.0 から、必須の引数となる
- それ以前のバージョンでは、デフォルトで ``CASCADE``

  ::

    A many-to-one relationship. Requires two positional arguments: the class to which the model is related and the on_delete option. (on_delete isn’t actually required, but not providing it gives a deprecation warning. It will be required in Django 2.0.)

  ::

    Deprecated since version 1.9:
    on_delete will become a required argument in Django 2.0. In older versions it defaults to CASCADE.


LEFT OUTER JOIN
------------------------------------------
Django のクエリセットは LEFT OUTER JOIN を表現できない

- SQLAlchemy でやろう
