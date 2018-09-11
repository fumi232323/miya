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

- `In an HttpRequest object, the GET and POST attributes are instances of django.http.QueryDict` だった・・・今まで全然気にしてなかった・・・


こんなのあるらしい
----------------------------------------
- https://docs.djangoproject.com/ja/1.11/topics/forms/modelforms/#django.forms.models.BaseInlineFormSet
- https://docs.djangoproject.com/ja/1.11/ref/forms/models/#inlineformset-factory


aggregateとannotateの違いがわかりやすい
----------------------------------------
- http://note.crohaco.net/2014/django-aggregate/


prefetch_relatedの話
----------------------------------------
- http://tokibito.hatenablog.com/entry/20140718/1405691738


Q Object
----------------------------------------
- https://docs.djangoproject.com/ja/1.11/ref/models/querysets/#q-objects
- クエリの検索条件を作れるやつ


モデルクラス名を全部小文字にしたのに ``_set`` がつく
------------------------------------------------------------------
- https://docs.djangoproject.com/ja/1.11/ref/models/relations/
- ``_set`` というのは子テーブルのデータを参照する django の機能
- モデルクラス名を全部小文字にしたのに ``_set`` がつく


FormView
--------------------------------------------------
- ``genericview`` はたくさん種類がある。
- 何らかの登録・更新処理で ``form`` を使ったバリデーションが必要なら 大体 ``FormView`` を使う


get_context_data
--------------------------------------------------
- 大抵の場合、ビューというのはレンダリングに必要なコンテキストを組み立てるものなので 大体の処理は ``get_context_data`` というメソッドに書く。


as_view
--------------------------------------------------
- https://github.com/django/django/blob/master/django/views/generic/base.py#L49
- as_view は view関数を生成して返してる
- 実際の処理は self.dispatch で クラスベースビューに処理を委譲してるんだと思います。とのこと。


genericview(class based view)
--------------------------------------------------
- 関数形式のviewはコンテキスト、テンプレート、フォーム、全部変えしてあげないといけなかった。つまり全部関数内に書く必要があった。
- 必要なものを切り離して属性として定義できるようにしたのが genericview(class based view)。
- とのこと。
