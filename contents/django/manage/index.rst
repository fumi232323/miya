.. article::
   :date: 2019-05-06
   :title: Django: django-admin.py, manage.py
   :category: django
   :tags:
   :canonical_url: /django/manage/
   :draft: false


===================================
Django: django-admin.py, ,manage.py
===================================


.. raw:: html

  <details>
    <summary>目次</summary>

.. contents::

.. raw:: html

  </details>


書籍/リファレンス
=================
- `Django documentation > django-admin と manage.py <https://docs.djangoproject.com/ja/2.2/ref/django-admin/>`_
- `現場で使える Django の教科書 <https://www.amazon.co.jp/dp/B07GK7BWB7/>`_


django-admin と manage.py の違い
================================
- 通常、単体の Django プロジェクトを用いる場合、 ``django-admin`` よりも ``manage.py`` の方が簡単に利用できる
- ``django-admin``, ``manage.py``, ``python -m django`` はすべて同じことができる ( ``startproject`` 後は)

django-admin.py, django-admin
-----------------------------
- インストールすると使えるようになる
- 基本的にどこからでも利用できる
- Django を ``setup.py`` ユーティリティを通じてインストールした場合は ``django-admin`` スクリプトはシステムのパスが通っている場所に配置されます。もしパスが通った箇所に存在しなければ、Python インストール先の ``site-packages/django/bin`` 内で見つける事ができます。そのスクリプトに対して、 ``/usr/local/bin`` 等のパスが通っている場所からシンボリックリンクを張ることを検討してください。
- 新しく Django プロジェクトを作成するときに利用する
- Django プロジェクトを作成してしまったら ``manage.py`` が使えるようになるので出番がなくなる

manage.py
---------
- ``startproject`` でプロジェクトを作成すると自動で作成される
- Django が提供する便利な管理コマンドが使えるほか...


よく使う Django 管理コマンド
============================

プロジェクト作成
----------------

.. code-block:: bash

  # プロジェクトのディレクトリに移動する
  $ cd project_dir
  # django-admin.py startproject <設定ディレクトリ名> <つくる場所>
  $ django-admin.py startproject config .


アプリケーション作成
--------------------

.. code-block:: bash

  # プロジェクトのディレクトリに移動する
  $ cd project_dir
  # python manage.py startapp <アプリケーション名>
  $ python manage.py startapp account

- 作成後、 settings.py > ``INSTALLED_APPS`` に手動でアプリケーションを追加する


マイグレーション
----------------

.. code-block:: bash

  # マイグレーションファイルを作成する
  # python manage.py makemigrations [<アプリケーション名>]
  $ python manage.py makemigrations account

  # マイグレーションを実行する
  # python manage.py migrate [<アプリケーション名>]
  $ python manage.py migrate account


スーバーユーザー作成
--------------------

.. code-block:: bash

  $ python manage.py cratesuperuser


runserver
---------

.. code-block:: bash

  # 開発用の Web サーバーを起動する
  # python manage.py runserver [<IPアドレス>:<ポート番号>]
  $ python manage.py runserver 0.0.0.0:8000


- IPアドレスとポート番号を省略すると ``127.0.0.1:8000`` で起動する
- Docker 上で runserver => ホストOSのブラウザから ``127.0.0.1:8000`` に接続できないときは、 ``0.0.0.0:8000`` で起動してみる


インタラクティブモードで実行する
--------------------------------

.. code-block:: bash

  # こうとか
  $ python manage.py shell
  # こうとか
  $ DJANGO_SETTINGS_MODULE=settings._ python manage.py shell
  # こう
  $ python manage.py shell --settings=settings._


使用例
^^^^^^

.. code-block:: python

  $ python manage.py shell --settings=settings._
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

.. code-block:: console

  $ python manage.py dbshell --settings=settings.local


System check framework を使って、Django プロジェクトの一般的な問題を検査する
============================================================================
- `Django documentation > django-admin と manage.py > Available commands > check <https://docs.djangoproject.com/ja/2.2/ref/django-admin/#check>`_
- `Django documentation > System check framework <https://docs.djangoproject.com/ja/2.2/ref/checks/#system-check-framework>`_


使用例 (python2)
----------------

  .. code-block:: bash

    $ DJANGO_SETTINGS_MODULE=settings.local python -Wd manage.py check


  - https://docs.python.org/ja/2.7/using/cmdline.html#cmdoption-w

    ::

      Python 2.7 から、 DeprecationWarning とその子クラスはデフォルトで無視されます。 -Wd オプションを指定して有効にすることができます。
