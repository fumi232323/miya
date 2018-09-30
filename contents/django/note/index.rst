.. article::
   :date: 2018-09-03
   :title: Django のメモ
   :category: django
   :tags:
   :canonical_url: /django/note/
   :draft: false


==================
Django のメモ
==================


template
===========

独自のテンプレートタグとフィルタ
--------------------------------------------------
カスタムタグやカスタムフィルタをつくることができる。

- `独自のテンプレートタグとフィルタ <https://docs.djangoproject.com/ja/1.11/howto/custom-template-tags/#custom-template-tags-and-filters>`_


Class-based views
==========================

::

  関数形式の view はコンテキスト、テンプレート、フォーム、全部変えしてあげないといけなかった。つまり全部関数内に書く必要があった。
  必要なものを切り離して属性として定義できるようにしたのが genericview (class based view)。


FormView
-------------
- ``genericview`` はたくさん種類がある。
- 何らかの登録・更新処理で ``form`` を使ったバリデーションが必要なら 大体 ``FormView`` を使う


as_view
-------------
- as_view は view関数を生成して返している
- 実際の処理は self.dispatch で クラスベースビューに処理を委譲してるんだと思います
- ソースコード URL: https://github.com/django/django/blob/master/django/views/generic/base.py#L49


get_context_data
--------------------------
- 大抵の場合、ビューというのはレンダリングに必要なコンテキストを組み立てるものなので 大体の処理は ``get_context_data`` というメソッドに書く。


こんなのある
===================

インラインフォームセット
----------------------------------------
使い方はよくわかっていない

- https://docs.djangoproject.com/ja/1.11/topics/forms/modelforms/#inline-formsets
- https://docs.djangoproject.com/ja/1.11/ref/forms/models/#inlineformset-factory


MultiValueDict
-----------------------
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
----------------------------------------
`In an HttpRequest object, the GET and POST attributes are instances of django.http.QueryDict` だそうです。

  - `QueryDict オブジェクト <https://docs.djangoproject.com/ja/2.1/ref/request-response/#querydict-objects>`_

    ::

      In an HttpRequest object, the GET and POST attributes are instances of django.http.QueryDict, a dictionary-like class customized to deal with multiple values for the same key. This is necessary because some HTML form elements, notably <select multiple>, pass multiple values for the same key.


System check framework を使って、Django プロジェクトの一般的な問題を検査する
------------------------------------------------------------------------------
`check <https://docs.djangoproject.com/ja/1.11/ref/django-admin/#check>`_ ,
`System check framework <https://docs.djangoproject.com/ja/1.11/ref/checks/#system-check-framework>`_

- 使い方 (python2)

  .. code-block:: python

    $ DJANGO_SETTINGS_MODULE=settings.local python -Wd manage.py check


  - https://docs.python.org/ja/2.7/using/cmdline.html#cmdoption-w

    ::

      Python 2.7 から、 DeprecationWarning とその子クラスはデフォルトで無視されます。 -Wd オプションを指定して有効にすることができます。


UserManager
--------------------------------------------------
`マネージャメソッド <https://docs.djangoproject.com/ja/1.11/ref/contrib/auth/#manager-methods>`_

参考 URL
^^^^^^^^^^
- https://github.com/django/django/blob/master/django/contrib/auth/models.py#L131
- https://docs.djangoproject.com/ja/1.11/topics/auth/customizing/#a-full-example

  - こうすると登録できる

    .. code-block:: python

      objects = MyUserManager()


RequestFactory
-----------------------------------
https://docs.djangoproject.com/en/2.1/topics/testing/advanced/#django.test.RequestFactory


便利さん
===========

django に便利コマンド追加してくれるさん
----------------------------------------
- `django-extensions <https://django-extensions.readthedocs.io/en/latest/>`_
