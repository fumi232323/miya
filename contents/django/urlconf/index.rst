.. article::
   :date: 2019-05-05
   :title: Django: URL ディスパッチャ, URLconf
   :category: django
   :tags:
   :canonical_url: /django/urlconf/
   :draft: false


===================================
Django: URL ディスパッチャ, URLconf
===================================


.. raw:: html

  <details>
    <summary>目次</summary>

.. contents::

.. raw:: html

  </details>


書籍/リファレンス
=================
- `現場で使える Django の教科書 <https://www.amazon.co.jp/dp/B07GK7BWB7/>`_
- `Django documentation > URL ディスパッチャ <https://docs.djangoproject.com/en/2.2/topics/http/urls/>`_


URL ディスパッチャ
==================

- 例外のハンドリングもしてくれる

  - マッチするURLが見つからなかった場合は ``handler404`` -> ``404.html`` を返してくれたりする

URLconf
=======
- ``urlpatterns`` にリストで書く

書き方
-------

- config/urls.py

  .. code-block:: python

    urlpatterns = [
        # Django2.0 から path になった
        # 後ろには / つける、前にはつけないのが慣習
        path('admin/', admin.site.urls),
        path('', views.index, name='index'),
        # アプリケーションディレクトリごとに１つずつ urls.py を配置するのがベストプラクティス
        path('accounts/', include('accounts.urls')),  # /accounts/ で始まるURLパターンは accounts.urls 見てね
        path('shop/', include('shop.urls')),          # /shop/ で始まるURLパターンは shop.urls 見てね
        # GenericView: 短いので、URLconf に直接書くこともある
        path(r'^$', TemplateView.as_view(template_name='index.html'), name='index'),  # TemplateView
        path(r'^$', RedirectView.as_view(template_name='index.html'), name='index'),  # RedirectView
    ]

- shop/urls.py

  .. code-block:: python

    # app_name:
      # * 必ず設定する、アプリケーション名と同じにする
      # * reverse メソッドや url タグから `名前空間名.名前空間内の name>` で逆引きできるようにするため
    app_name = 'shop'
    urlpatterns = [
        path('', views.index, name='index'),
        # パスコンバーター:
          # * ビュー関数を呼び出すときのキーワード引数として使われる
          # * int, str, path, slug, uuid の 5種類がある、自作も可
        path('<int:book_id>/', views.detail, name='detail'),
    ]
