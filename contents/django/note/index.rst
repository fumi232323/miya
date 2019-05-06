.. article::
   :date: 2018-10-28
   :title: Django なんでもメモ
   :category: django
   :tags:
   :canonical_url: /django/note/
   :draft: false


===================
Django なんでもメモ
===================


.. raw:: html

  <details>
    <summary>目次</summary>

.. contents::

.. raw:: html

  </details>



マイグレーションを間違えたときのリカバリ方法
=============================================
1. DjangoのDBシェルでローカルDBにつなぐ

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


こんなのある
============

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


便利さん
========

django に便利コマンド追加してくれるさん
----------------------------------------
- `django-extensions <https://django-extensions.readthedocs.io/en/latest/>`_
