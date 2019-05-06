.. article::
   :date: 2019-05-05
   :title: Django: Model, Django ORM
   :category: django
   :tags:
   :canonical_url: /django/model/
   :draft: false


=========================
Django: Model, Django ORM
=========================


.. raw:: html

  <details>
    <summary>目次</summary>

.. contents::

.. raw:: html

  </details>


リファレンス
=============
- `Django documentation > モデルフィールドリファレンス <https://docs.djangoproject.com/ja/2.2/ref/models/fields/>`_
- `Django documentation > QuerySet API reference <https://docs.djangoproject.com/ja/2.2/ref/models/querysets/>`_
- `現場で使える Django の教科書 <https://www.amazon.co.jp/dp/B07GK7BWB7/>`_


Model
======

* 複合キーは主キーにできない

リレーションまとめ
------------------
.. list-table::
  :widths: auto
  :header-rows: 1
  :stub-columns: 1

  * - リレーション
    - 使うフィールド
    - 使い方
  * - 一対一
    - OneToOneField
    - どちらか一方のモデルに OneToOneField をくっつける
  * - 多対一
    - ForeignKey
    - 多側に ForeignKey をくっつける
  * - 多対多
    - ManyToManyField
    - どちらか一方のモデルに ManyToManyField をくっつける


Model の実装例
--------------
.. code-block:: python

  from django.db import models


  class Publisher(models.Model):
      """出版社モデル"""
      class Meta:
          db_table = 'publisher'

      name = models.CharField(verbose_name='出版社名', max_length=255)

      def __str__(self):
          return self.name


  class Author(models.Model):
      """著者モデル"""
      class Meta:
          db_table = 'author'

      name = models.CharField(verbose_name='著者名', max_length=255)

      def __str__(self):
          return self.name


  class Book(models.Model):
      """本モデル"""
      class Meta:
          """
          対応するテーブルや、複数カラムに対するインデックスやユニーク制約などの
          モデル全体に対する付加情報を記述する
          """
          # テーブル名を定義
          # 定義しないと、 `アプリケーション名_モデルのクラス名をスネークケースにした文字列` がテーブル名になる
          db_table = 'book'

      title = models.CharField(
          verbose_name='タイトル',  # フィールド名
          max_length=255,  # 文字列の最大文字数
          # choices: フォーム利用時にセレクトボックスに表示する選択肢
          # validators: 文字種チェックなどのバリデーションを指定
          error_messages={'invalid': 'title ちがうよー'}  # バリデーションNGの場合のエラーメッセージ
      )
      publisher = models.ForeignKey(
          # 多対一のリレーション: 多側に ForeignKey をくっつける
          Publisher, verbose_name='出版社',
          # ForeignKey と OneToOneField には on_delete をつける癖をつけよう
          on_delete=models.PROTECT,  # 自身のレコードは削除されない
      )
      authors = models.ManyToManyField(
          # 多対多のリレーション: 一方のモデルに ManyToManyField をくっつける
          #   * マイグレーション実行すると Django が自動的に中間テーブルを作成してくれる
          Author, verbose_name='著者'
      )
      price = models.IntegerField(
          verbose_name='価格',
          null=True,       # NULL制約
          unique=False,    # ユニーク制約
          blank=True,      # フォーム利用時に入力必須にするかどうか
          db_index=False,  # DB のインデックスを設定するかどうか
          default=0,       # レコード登録時に値が指定されなかったときのデフォルト値
      )
      description = models.TextField(verbose_name='概要', null=True, blank=True)
      publisher_date = models.DateField(verbose_name='出版日')

      def __str__(self):
          # 管理サイトに表示されるよ
          return self.title


  class BookStock(models.Model):
      """本の在庫モデル"""
      book = models.OneToOneField(
          # 一対一のリレーション: どちらか一方のテーブルに OneToOneField をくっつける
          Book, verbose_name='本',
          on_delete=models.CASCADE  # 自身のレコードも削除される
      )
      quantity = models.IntegerField(verbose_name='在庫数', default=0)


on_delete
---------
`ForeignKey.on_delete <https://docs.djangoproject.com/ja/2.2/ref/models/fields/#django.db.models.ForeignKey.on_delete>`_

- 6種類くらいあって、用途に応じて選べる
- Django 2.0 から、必須の引数となる
- それ以前のバージョンでは、デフォルトで ``CASCADE``


UserManager
-----------
- `Django documentation > django.contrib.auth > マネージャメソッド <https://docs.djangoproject.com/ja/2.2/ref/contrib/auth/#manager-methods>`_
- `Django documentation > Django の認証方法のカスタマイズ > 完全な具体例 <https://docs.djangoproject.com/ja/2.2/topics/auth/customizing/#a-full-example>`_
- https://github.com/django/django/blob/master/django/contrib/auth/models.py#L131
- こうすると登録できる

    .. code-block:: python

      objects = MyUserManager()

- TODO: これは Model のところじゃなくて、認証のところかもしれない。いったんここに置いておく。


Django ORM
==========

* 単体のオブジェクトを保存・更新するような行レベルのクエリ操作: モデルオブジェクトのメソッドを利用する
* データベースのテーブルレベルのクエリ操作: モデルの「モデルマネージャー」 ( ``objects`` ) を経由してクエリセットAPI のメソッドを利用する

  * モデルマネージャーは通常、モデルクラスに ``objects`` という名前で保存されている


1件だけ取得する
---------------
.. code-block:: python

  User.objects.get()  # この `objects` がモデルマネージャー

* モデルが返る
* 1件も見つからないと ``DoesNotExist``
* 2件以上見つかると ``MultipleObjectsReturn``

全件取得する
------------

.. code-block:: python

  User.objects.all()

* 即座にデータベースにはアクセスせず、クエリセットオブジェクトを返す
* しかるべきタイミングでデータベースアクセスする = 遅延評価
* 1件も見つからなくても例外発生しない、空のリストを返す

検索条件をつけて取得する
------------------------

.. code-block:: python

  User.objects.filter(is_active=True)

* 即座にデータベースにはアクセスせず、クエリセットオブジェクトを返す
* ``filter()`` を何度も繋げて書ける

filter をつなげる例
^^^^^^^^^^^^^^^^^^^

.. code-block:: python

  keyword = request.GET.get('keyword')
  queryset = Book.objects.filter()
  if keyword:
      queryset = queryset.filter(title=keyword)

  # ここでクエリが発行される (print すると発行される)
  print(queryset)


* この例の場合、発行されるクエリの総数は1本


AND 条件
--------

.. code-block:: python

  Book.objects.filter(title='Django Book', price=1000)

* 検索条件を列挙すれば AND 条件


OR 条件
-------

.. code-block:: python

  from django.db.models import Q
  Book.objects.filter(Q(title='Django Book') | Q(price=1000))

* ``Q`` と ``|`` (パイプ) を使う


>, <, >=, <=
-------------

.. code-block:: python

  Book.objects.filter(price__gt=1000)  # >1000
  Book.objects.filter(price__lt=1000)  # <1000
  Book.objects.filter(price__gte=1000)  # >=1000
  Book.objects.filter(price__lte=1000)  # <=1000

* ``__`` (アンダーバー2つ) でフィールド名とキーワード (``gt``, ``lt``, ``gte``, ``lte``) をつなぐ


IN
--

.. code-block:: python

  Book.objects.filter(price__in=[900, 1000])  # IN(900, 1000)

* ``__`` (アンダーバー2つ) でフィールド名とキーワード (``in``) をつなぐ
* IN 句の中身はリストで書く


LIKE
----

.. code-block:: python

  Book.objects.filter(title__icontains='Django')  # ILIKE '%Django%'
  Book.objects.filter(title__contains='Django')  # LIKE '%Django%'

* ``__`` (アンダーバー2つ) でフィールド名とキーワード (``icontains``, ``contains``) をつなぐ
* 大文字と小文字を区別しない: ``icontains``
* 大文字と小文字を区別する: ``contains``


EXISTS
------
* exists: レコードが存在するか否か True/False で返す

.. code-block:: python

  Book.objects.all().exists()
  Book.objects.filter(title__icontains='Django').exists()


ORDER BY
--------

* order_by:

.. code-block:: python

  # 降順はフィールド名の前に ``-`` つける
  Book.objects.all().order_by('-price')

  # 複数フィールドでソートするときはカンマ区切り
  Book.objects.all().order_by('price', 'publish_date')


Q Object
--------
- `Q オブジェクトを用いた複雑な検索 <https://docs.djangoproject.com/ja/2.2/topics/db/queries/#complex-lookups-with-q>`_


リレーション先のモデルを使った条件検索
--------------------------------------

.. code-block:: python

  Book.objects.filter(publisher__name='自費出版社')

* ``OneToOneField``, ``ForeignKey``, ``ManyToManyField`` でリレーションしていると、
  ``リレーションつけたフィールド名__リレーション先モデルのフィールド名`` で JOIN できる


モデルクラス名を全部小文字にしたのに ``_set`` がつく
----------------------------------------------------
`Related objects reference <https://docs.djangoproject.com/ja/1.11/ref/models/relations/>`_

- ``_set`` というのは子テーブルのデータを参照する django の機能
- モデルクラス名を全部小文字にしたのに ``_set`` がつく


values() と values_list()
-------------------------

values()
^^^^^^^^^
辞書のクエリセットで取得できる。

- https://docs.djangoproject.com/ja/2.1/ref/models/querysets/#values

  .. code-block:: python

    >>> Blog.objects.filter(name__startswith='Beatles').values()
    <QuerySet [{'id': 1, 'name': 'Beatles Blog', 'tagline': 'All the latest Beatles news.'}]>


values_list()
^^^^^^^^^^^^^^
タプルのリストのクエリセットで取得できる。

- https://docs.djangoproject.com/ja/2.1/ref/models/querysets/#values-list

  .. code-block:: python

    >>> Entry.objects.values_list('id', 'headline')
    <QuerySet [(1, 'First entry'), ...]>


  - 1カラムしか取得しない場合は、 ``flat=True`` をつけると、リストのクエリセットで取得できる。

    .. code-block:: python

      >>> Entry.objects.values_list('id', flat=True).order_by('id')
      <QuerySet [1, 2, 3, ...]>


aggregate と annotate
---------------------
``aggregate`` と ``annotate`` の違いがわかりやすい

- `Djangoの集計について <http://note.crohaco.net/2014/django-aggregate/>`_


LEFT OUTER JOIN
---------------
Django のクエリセットは LEFT OUTER JOIN を表現できない

- SQLAlchemy でやろう


select_related, prefetch_related
--------------------------------

.. list-table::
  :widths: 50 50
  :header-rows: 1

  * - select_related
    - prefetch_related
  * - ``一`` や ``多`` 側から ``一`` のリレーションのモデルオブジェクトをJOINで取得
    - ``一`` や ``多`` 側から ``多`` のリレーションのモデルオブジェクトを取得してキャッシュに保持
  * - リレーション先のオブジェクトを取得するために JOIN を使ったクエリを発行できる
    - 取得したオブジェクト群をオブジェクト内部のキャッシュに保持し、それを使い回すことで同じクエリが何度も発行されないようにする
  * - .. code-block:: python

        Book.objects.all().select_related('publisher')
    - .. code-block:: python

        Book.objects.all().prefetch_related('authors')


- クエリの本数を減らそう!

  - ``N+1問題``
  - 特にリレーションを持ったモデルの検索結果 (クエリセットオブジェクト) をループ処理する場合に起こりがち
  - 後続の処理で何度もアクセスされそうなオブジェクトに対して前もって処理を施しておくことでクエリの本数を減らそう
  - その他参考: `偏った言語信者の垂れ流し > Djangoでprefetch_relatedを使ってクエリ数を減らす <http://tokibito.hatenablog.com/entry/20140718/1405691738>`_


遅延評価
--------
クエリセットを返す ``all()`` や ``filter()`` がクエリを発行するタイミング (データベースアクセスするタイミング) はこれら

* ``for`` ループなどイテレーションが開始されたタイミング
* ``[]`` を使ってスライスしたタイミング

  * ``[0:5]`` のように範囲指定するとクエリは即時発行されない

* オブジェクトを直列化したタイミング
* オブジェクトを ``REPL`` や ``print()`` で表示したタイミング
* ``len()`` でサイズを取得したタイミング
* ``list()`` で強制的にリストに変換したタイミング
* ``bool()`` で強制的に ``Boolean`` に変換したタイミング


トランザクション
-----------------
* デフォルト設定では、オブジェクトの保存・更新・削除は即時反映 (実行した時点でデータベースに反映される)
* トランザクションの範囲指定する場合は、

  .. code-block:: python

    from django.db import transaction
    with transaction.atomic():
        User(username='fumi23').save()
        User(username='fumi23').save()


* トランザクションのデフォルト設定を変更できる

  * settings.py

    .. code-block:: python

      DATABASES = [
          'default': {
              # リクエストの開始から終了までに設定
              'ATOMIC_REQUESTS': True,
          }
      ]


User モデルを拡張したい
========================
- 現場で使える Django の教科書《基礎編》 P.63 参照のこと


発行されるクエリを見たい
========================

* Django シェル

  .. code-block:: shell

    $ python manage.py shell --settings=settings._
    >>> print(Book.objects.filter(title__icontains='Django').query)

* ロギング
* django-debug-toolbar
