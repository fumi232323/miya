.. article::
   :date: 2019-05-05
   :title: Django: View
   :category: django
   :tags:
   :canonical_url: /django/view/
   :draft: false


============
Django: View
============


.. raw:: html

  <details>
    <summary>目次</summary>

.. contents::

.. raw:: html

  </details>


リファレンス/書籍
=================
- `Django documentation > クラスベースビュー <https://docs.djangoproject.com/ja/2.2/topics/class-based-views/>`_
- `Django documentation > ビルトインのクラスベースビュー API <https://docs.djangoproject.com/ja/2.2/ref/class-based-views/>`_
- `現場で使える Django の教科書 <https://www.amazon.co.jp/dp/B07GK7BWB7/>`_
- crohaco さん、 shimizukawa さん、 tell-k さん


関数ベースのビュー
==================

- 第一引数に django.http.request.HttpRequest を受け取る
- 戻り値に django.http.response.HttpResponse を返す

.. code-block:: python

  from django.shortcuts import render


  def HelloView(request):
      """関数ベースのビュー"""
      if request.method == 'GET':
          context = {
              'message': "Hello World!",
          }
          # render:
          #   * HttpResponse を作ってくれる
          #   * テンプレートをレンダリングするとき使う
          return render(request, 'hello.html', context)


クラスベースのビュー (Class-based views)
========================================
* 関数形式の view はコンテキスト、テンプレート、フォーム、全部変えしてあげないといけなかった。つまり全部関数内に書く必要があった。
  必要なものを切り離して属性として定義できるようにしたのが genericview (class based view)。

* akiyoko さんは断然、クラスベースビューをお勧めするとのこと

  * 汎用ビュー (Generic View) としてさまざまな用途のクラスが提供されているのでその恩恵を受けられる
  * すべての基本となる基本汎用ビュー ``django.views.generic.base.View`` を利用すれば見通しのよいビューが書けるし、応用も効く
  * 汎用ビューのほかにも便利な Mixin クラスを多重継承してお決まりの処理を再利用できる


.. code-block:: python

  from django.shortcuts import render
  from django.views import View
  # from django.views.generic import View と同義↑


  class HelloView(View):
      """クラスベースのビュー"""
      def get(self, request, *args, **kwargs):
          context = {
              'message': "Hello World!",
          }
          return render(request, 'hello.html', context)


  hello = HelloView.as_view()


.. code-block:: python

  from django.contrib.auth import login as auth_login
  from django.shortcuts import render, redirect
  from django.urls import reverse
  from django.views import View


  class LoginView(View):
      def get(self, request, *args, **kwargs):
          """GET リクエスト"""
          context = {
              'form': LoginForm(),
          }
          # ログイン画面用のテンプレートに値が空のフォームをレンダリング
          # render: テンプレートをレンダリングするとき使う
          return render(request, 'accounts/login.html', context)

      def post(self, request, *args, **kwargs):
          """POST リクエスト"""
          # リクエストからフォームを作成
          form = LoginForm(request.POST)
          # バリデーション (ユーザーの認証も併せて実施)
          if not form.is_valid():
              # バリデーション NG の場合はログイン画面のテンプレートを再表示
              return render(request, 'accounts/login.html', {'form': form})

          # User オブジェクトをフォームから取得
          user = form.get_user()

          # ログイン処理 (取得した User オブジェクトをセッションに保存 & User データを更新)
          auth_login(request, user)

          # ショップ画面にリダイレクト
          # redirect: リダイレクトするとき使う
          #   * リダイレクト先のURLは reverse を使って取得する
          #   * ハードコーディングするなかれ
          return redirect(reverse('shop:index'))


as_view
--------
* https://github.com/django/django/blob/master/django/views/generic/base.py#L49
* クラスベースのビューをビュー関数化してくれるメソッド

  * as_view は view 関数を生成して返している
  * これをしておくと、URL ディスパッチャだけでなく他のビューからも呼び出せるようになる
  * 実際の処理は self.dispatch で クラスベースビューに処理を委譲してるんだと思います

* urls.py で as_view せずに、 views.py で as_view した Class-based view をグローバル変数に代入するとうれしいタイミング

  * 同じ view を複数の url に設定したい

    * モジュールの import が1回しか発生しないのはモジュール毎の話じゃなくプロセス全体 ( Django で言うと ``runserver...`` した単位) の話
    * url ごとに同じ View を何回も生成するんだったら、同じでよい (シングルトン)


Base views
==========

View
----
- django.views.generic.base.View
- すべての元となるクラスベースビューです。他の全てのクラスベースビューは、この基本クラスを継承しています。

TemplateView
------------

- テンプレートを表示することに特化した Generic View
- トップ画面やヘルプ画面などの単純なテンプレートを表示するのによく使う
- オーバーライドできる変数やメソッドがいくつか用意されていて、デフォルトの挙動をある程度自由に変更できる

.. code-block:: python

  from django.views.generic import TemplateView
  from django.contrib.auth.models import User


  class IndexView(TemplateView):
      template_name = 'index.html'

      def get_context_data(self, **kwargs):
          # get_context_data をオーバーライドした例
          # テンプレートに渡すコンテキストに任意の変数を追加できる
          context = super(IndexView, self).get_context_data(**kwargs)
          # テンプレートに渡すコンテキストに `user_count` という変数を追加
          context['user_count'] = User.objects.all().count()
          return context


  index = IndexView.as_view()


get_context_data
^^^^^^^^^^^^^^^^^
- 大抵の場合、ビューというのはレンダリングに必要なコンテキストを組み立てるものなので 大体の処理は ``get_context_data`` というメソッドに書く。


RedirectView
------------

- リダイレクトに特化した Generic View
- 任意の URL にリダイレクトすることに特化したやつ

.. code-block:: python

  from django.views.generic import RedirectView


  class IndexView(RedirectView):
      url = '/accounts/login/'
      # pattern_name = 'accounts:login'  # パターンで URL を指定する場合

      def get_redirect_url(self, *args, **kwargs):
          # リダイレクトする URL を動的に組み立てるためのやつをオーバーライドできる
          pass


  index = IndexView.as_view()


いろいろな Generic views
========================
- これが便利とのこと: https://ccbv.co.uk/

ListView
--------
- モデルオブジェクトの一覧を表示するための View

.. code-block:: python

  from django.views.generic import ListView
  from .models import Book


  class BookListView1(ListView):
      # リストしたいモデルを指定する
      # これだけで、 `shop/book_list.html` という名前のテンプレートに、
      # object_list (or book_list) という変数名で、
      # Book モデルの全てのレコードの一覧を渡してくれる
      model = Book

.. code-block:: python

      # ほかにもいろいろある...
      # 利用するテンプレートを指定する
      template_name = 'husky.html'
      # オブジェクトの一覧を取得するためのクエリセットを指定する
      queryset = Book.objects.filter(price__gt=1000)


FormView
--------
- 何らかの登録・更新処理で ``form`` を使ったバリデーションが必要なら 大体 ``FormView`` を使う


ログイン・ログアウト
====================
- `現場で使える Django の教科書 <https://www.amazon.co.jp/dp/B07GK7BWB7/>`_ P.43 によく書いてあるのでそちらを参照のこと

  - request.user:

    - ログイン済: User オブジェクト
    - 未ログイン: AnonymousUser オブジェクト

  - ログイン済みか否か: request.user.is_authenticated
  - ログアウトすると、

    - サーバーのセッションクリア
    - request.user に AnonymousUser をセット


LoginView, LogoutView
---------------------
- ログインに特化した View: django.contrib.auth.views.LoginView
- ログアウトに特化した View: django.contrib.auth.views.LogoutView

.. code-block:: python

  from django.contrib.auth.views import LoginView as AuthLoginView


  class LoginView(AuthLoginView):
      """
      ログインビューの実装例

      * 設定値の調整が必要になることもあるよ

        * LOGIN_URL
        * LOGIN_REDIRECT_URL
        * LOGOUT_REDIRECT_URL
      """
      template_name = 'accounts/login.html'


LoginRequiredMixin
------------------

- LoginRequiredMixin: 未ログインのユーザーがアクセスしようとしたときに何らかのペナルティを課すための Mixin

  - Django1.9 から導入された
  - 継承すると、未ログインユーザーがアクセスしたらば LOGIN_URL で定義した URL にリダイレクトしてくれるよ

.. code-block:: python

  from django.contrib.auth.mixins import LoginRequiredMixin
  from django.views.generic import ListView

  from .models import Book


  class BookListView(LoginRequiredMixin, ListView):
      model = Book
      # 403 エラー画面を表示する場合は次のコメントアウトを外す
      # raise_exception = True
