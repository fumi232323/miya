.. article::
   :date: 2018-10-28
   :title: Django のメモ
   :category: django
   :tags:
   :canonical_url: /django/note/
   :draft: false


==================
Django のメモ
==================


マイグレーションを間違えたときのリカバリ方法
=============================================
1. DjangoのDBシェルでローカルのポスグレにつなぐ

    .. code-block:: console

      $ python manage.py dbshell --settings=settings.local


2. django_migrations テーブルから該当アプリのレコードを削除する

    .. code-block:: sql

      SELECT * FROM django_migrations WHERE app like '%{application_name}%';
      DELETE FROM django_migrations WHERE id={該当のID};

3. 該当テーブルやカラムも DROP する

    .. code-block:: sql

      DROP TABLE {table_name};
      ALTER TABLE {table_name} DROP COLUMN {column_name};

4. 該当のマイグレーションファイルも削除しておく

5. もう一回最初からマイグレーションする

    .. code-block:: console

      $ python manage.py makemigrations {application_name} --settings=settings.local
      $ python manage.py migrate {application_name} --settings=settings.local


django-admin と manage.py
==========================

リファレンス
------------
https://docs.djangoproject.com/ja/2.1/ref/django-admin/#cmdoption-shell-command


Django をインタラクティブモードで実行する
-----------------------------------------

使い方
^^^^^^^

.. code-block:: bash

  # こうとか
  $ DJANGO_SETTINGS_MODULE=settings._ python manage.py shell
  # こう
  $ python manage.py shell --settings=settings._


使用例
^^^^^^^

.. code-block:: python

  (venv) [vagrant@localhost apps]$ python manage.py shell --settings=settings._
  Python 2.7.7 (default, Dec 11 2017, 18:45:38)
  [GCC 4.4.7 20120313 (Red Hat 4.4.7-18)] on linux2
  Type "help", "copyright", "credits" or "license" for more information.
  (InteractiveConsole)
  >>> from myapp.models import Entry
  >>> from django.db.models import Q
  >>> target_entry_id = None
  >>> Entry.objects.filter(
  ...     Q(expiration_year_month__gte=target_entry_id) |
  ...     Q(expiration_year_month__isnull=True)
  ... )


Django の DBシェルでローカルDBにつなぐ
--------------------------------------

使い方
^^^^^^
.. code-block:: console

  $ python manage.py dbshell --settings=settings.local

ガイド
^^^^^^
https://docs.djangoproject.com/ja/2.1/ref/django-admin/#dbshell


System check framework を使って、Django プロジェクトの一般的な問題を検査する
------------------------------------------------------------------------------
`check <https://docs.djangoproject.com/ja/1.11/ref/django-admin/#check>`_ ,
`System check framework <https://docs.djangoproject.com/ja/1.11/ref/checks/#system-check-framework>`_

- 使い方 (python2)

  .. code-block:: bash

    $ DJANGO_SETTINGS_MODULE=settings.local python -Wd manage.py check


  - https://docs.python.org/ja/2.7/using/cmdline.html#cmdoption-w

    ::

      Python 2.7 から、 DeprecationWarning とその子クラスはデフォルトで無視されます。 -Wd オプションを指定して有効にすることができます。


template
========

独自のテンプレートタグとフィルタ
--------------------------------
カスタムタグやカスタムフィルタをつくることができる。

- `独自のテンプレートタグとフィルタ <https://docs.djangoproject.com/ja/1.11/howto/custom-template-tags/#custom-template-tags-and-filters>`_


Class-based views
=================

::

  関数形式の view はコンテキスト、テンプレート、フォーム、全部変えしてあげないといけなかった。つまり全部関数内に書く必要があった。
  必要なものを切り離して属性として定義できるようにしたのが genericview (class based view)。

リファレンス
------------
https://docs.djangoproject.com/ja/2.1/topics/class-based-views/


FormView
--------
- ``genericview`` はたくさん種類がある。
- 何らかの登録・更新処理で ``form`` を使ったバリデーションが必要なら 大体 ``FormView`` を使う


as_view
-------
https://github.com/django/django/blob/master/django/views/generic/base.py#L49

- as_view は view 関数を生成して返している
- 実際の処理は self.dispatch で クラスベースビューに処理を委譲してるんだと思います
- urls.py で as_view せずに、 views.py で as_view した Class-based view をグローバル変数に代入するのは、

  - 同じ view を複数のurlに設定したいとき
  - モジュールの import が1回しか発生しないのはモジュール毎の話じゃなくプロセス全体 ( Django で言うと ``runserver...`` した単位) の話
  - url ごとに同じ View を何回も生成するんだったら、同じでよいでしょう (シングルトン) 、ということ


get_context_data
----------------
- 大抵の場合、ビューというのはレンダリングに必要なコンテキストを組み立てるものなので 大体の処理は ``get_context_data`` というメソッドに書く。


こんなのある
============

インラインフォームセット
------------------------
使い方はよくわかっていない

- https://docs.djangoproject.com/ja/1.11/topics/forms/modelforms/#inline-formsets
- https://docs.djangoproject.com/ja/1.11/ref/forms/models/#inlineformset-factory


MultiValueDict
--------------
なにがうれしいのかさっぱりわからない => `MultiValueDict を継承してる QueryDict とか見るとユースケースはなんとなく想像つくと思います` と教えて頂いた。

- https://docs.djangoproject.com/ja/2.1/_modules/django/utils/datastructures/

  ::

    A subclass of dictionary customized to handle multiple values for the same key.


- よく見たら、こういうところが便利だと思った ↓

  .. code-block:: python

    >>> from django.utils.datastructures import MultiValueDict
    >>> d = MultiValueDict({'name': ['Adrian', 'Simon'], 'position': ['Developer']})
    >>> d.update({'name': 'Momo'})
    >>> d
    <MultiValueDict: {'position': ['Developer'], 'name': ['Adrian', 'Simon', 'Momo']}>
    >>> dd = {'name': ['Adrian', 'Simon'], 'position': ['Developer']}
    >>> dd.update({'name': 'Momo'})
    >>> dd
    {'position': ['Developer'], 'name': 'Momo'}


QueryDict オブジェクト
----------------------
`In an HttpRequest object, the GET and POST attributes are instances of django.http.QueryDict` だそうです。

  - `QueryDict オブジェクト <https://docs.djangoproject.com/ja/2.1/ref/request-response/#querydict-objects>`_

    ::

      In an HttpRequest object, the GET and POST attributes are instances of django.http.QueryDict, a dictionary-like class customized to deal with multiple values for the same key. This is necessary because some HTML form elements, notably <select multiple>, pass multiple values for the same key.


UserManager
-----------
`マネージャメソッド <https://docs.djangoproject.com/ja/1.11/ref/contrib/auth/#manager-methods>`_

参考 URL
^^^^^^^^^^
- https://github.com/django/django/blob/master/django/contrib/auth/models.py#L131
- https://docs.djangoproject.com/ja/1.11/topics/auth/customizing/#a-full-example

  - こうすると登録できる

    .. code-block:: python

      objects = MyUserManager()


RequestFactory
--------------
https://docs.djangoproject.com/en/2.1/topics/testing/advanced/#django.test.RequestFactory


便利さん
========

django に便利コマンド追加してくれるさん
----------------------------------------
- `django-extensions <https://django-extensions.readthedocs.io/en/latest/>`_
