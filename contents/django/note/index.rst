.. article::
   :date: 2018-09-03
   :title: Django のメモ
   :category: django
   :tags:
   :canonical_url: /django/note/
   :draft: false


未整理
=========

django に便利コマンド追加してくれるさん
----------------------------------------
- https://django-extensions.readthedocs.io/en/latest/

MultiValueDict
----------------------------------------
- https://docs.djangoproject.com/ja/2.1/_modules/django/utils/datastructures/

::

  A subclass of dictionary customized to handle multiple values for the same key.

- 何がうれしいのかさっぱりわからない・・・とつぶやいたら、 `MultiValueDict を継承してる QueryDict とか見るとユースケースはなんとなく想像つくと思います` と教えて頂き見てみてたら、ほー、となった。
- あとよく見たら、こういうところが便利だと思った ↓

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
- https://docs.djangoproject.com/ja/2.1/ref/request-response/#querydict-objects

::

  In an HttpRequest object, the GET and POST attributes are instances of django.http.QueryDict, a dictionary-like class customized to deal with multiple values for the same key. This is necessary because some HTML form elements, notably <select multiple>, pass multiple values for the same key.

`In an HttpRequest object, the GET and POST attributes are instances of django.http.QueryDict` だった・・・今まで全然気にしてなかった・・・
