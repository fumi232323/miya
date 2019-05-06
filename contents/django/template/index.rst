.. article::
   :date: 2019-05-06
   :title: Django: Template
   :category: django
   :tags:
   :canonical_url: /django/template/
   :draft: false


================
Django: Template
================


.. raw:: html

  <details>
    <summary>目次</summary>

.. contents::

.. raw:: html

  </details>


リファレンス/書籍
=================
- `Django documentation > テンプレート <https://docs.djangoproject.com/ja/2.2/topics/templates/>`_
- `Django documentation > 組み込みタグとフィルタ <https://docs.djangoproject.com/ja/2.2/ref/templates/builtins/>`_
- `現場で使える Django の教科書 <https://www.amazon.co.jp/dp/B07GK7BWB7/>`_


なるほどめも
============
- 変数定義できない
- Python モジュールを直接 import して使うことはできない

  - 独自の関数を使うにはテンプレートタグやフィルタを自作する

- フィルター: 変数の内容を表示する際の表示形式を変更する


変数表示
========

.. code-block:: django

  {# __str__() の結果を表示してくれる #}
  {{ user }}

  {# このように書くと #}
  {{ user.username }}
  {% comment "以下の順番に値を取得しようとして、一番先に取得できた値を表示してくれる" %}
    user['username']  # 1. 辞書としての照合
    user.username     # 2. 属性値の照合
    user.username()   # 3. メソッド呼び出し
    user[username]    # 4. リストやタプルの添字指定
  {% endcomment %}


- テンプレートには「コンテキスト」と呼ばれる変数と値がマッピングされたオブジェクトが渡される

  - コンテキストに含まれるもの

    - ビューから渡された変数
    - ``settings.py`` の ``TEMPLATES.OPTIONS.context_processors`` に定義された関数でセットされた変数

      - ``request``: リクエストオブジェクト
      - ``user``: ユーザー
      - ``perms``: ユーザーのパーミッション
      - ``messages``: フラッシュメッセージ
      - e.t.c...

- 何もしなくも XSS 対策として ``<``, ``>``, ``'``, ``"``, ``&`` を自動でエスケープしてくれる


フィルタ
========

.. code-block:: django

  {# 変数名の直後に ``|`` (パイプ) を使って繋げる #}
  {{ <変数名>|<フィルタ名> }}

  {# フィルタによっては引数もとれる、 ``:`` で繋げる #}
  {{ <変数名>|<フィルタ名>:"<引数>" }}

  {# フィルタは ``|`` で連結できる #}
  {{ <変数名>|<フィルタ名1>:"<引数>"|<フィルタ名2>:"<引数>" }}


デフォルト表示
--------------

.. code-block:: django

  {{ user.first_name|default:"" }} {{ user.first_name|default:"" }}

* 変数が存在しない場合、あるいは変数の値が ``None``, ``''``, ``0``, ``False``, ``[]`` などの場合に指定した文字列を表示する


文字列長を表示
--------------

.. code-block:: django

  {{ user.username|length }}


エスケープの無効化
------------------

.. code-block:: django

  {{ book.description|safe }}

* XSS 対策として自動でエスケープしてくれる ``<``, ``>``, ``'``, ``"``, ``&`` をエスケープしない
* 変数の内容が安全だとわかっている場合のみ利用すること


日時フォーマット
----------------

.. code-block:: django

  {{ user.last_login|date:"Y-m-d H:i:s" }}


改行タグの変換
--------------

.. code-block:: django

  {{ book.description|linebreaksbr }}

* ``\n`` を ``<br>`` に変換してくれる
* 便利だな...


リンク変換
----------

.. code-block:: django

  {{ book.description|urlize }}

* URL と メールアドレスの部分だけをアンカータグで囲んでクリック可能なリンクに変換してくれる
* しゅごいな...


文字の切り詰め
--------------

.. code-block:: django

  {{ book.description|truncatechars_html:5 }}

* 指定した文字数まで切り詰めて ``...`` をくっつけてくれる
* HTML タグを考慮して省略後の文字にきちんと閉じタグをつけてくれる
* うええ...


テンプレートタグ
================

if
--

.. code-block:: django

  {% if user.is_superuser %}
    システム管理者です。
  {% elif user.is_staff %}
    スタッフです。
  {% else %}
    一般ユーザーだよ。
  {% endif %}


.. code-block:: django

  {# フィルタと組み合わせることもできる #}
  {% if user.username|length < 3 %}
    ユーザー名が短かすぎます。
  {%  endif %}


for
---

.. code-block:: django

  {% for book in book_list %}
    {{ book.title }}

    {# for ループ内で使える変数 #}
    {{ forloop.counter }}     {# 現在のループカウンタ番号 ( 1 から順にカウント ) #}
    {{ forloop.counter0 }}    {# 現在のループカウンタ番号 ( 0 から順にカウント ) #}
    {{ forloop.revcounter }}  {# 現在のループカウンタ値 ( 1 から順に、末尾からカウント) #}
    {{ forloop.revcounter0 }} {# 現在のループカウンタ値 ( 0 から順に、末尾からカウント) #}
    {{ forloop.first }}       {# 最初のループであれば True #}
    {{ forloop.last }}        {# 最後のループであれば True #}
    {{ forloop.parentloop }}  {# 入れ子のループであるとき、現在のループを囲んでいる 1 つ上のループを表します。 #}

  {%  empty %}
    本ない
  {%  endfor %}


テンプレートの継承
------------------

- 親テンプレート ``parent.html``

  .. code-block:: django

    {% block sample %}Hello {% endblock %}


- 子テンプレート ``child.html``

  .. code-block:: django

    {% extends "parent.html" %}
    {% block sample %}{{ block.super }World! {% endblock %}
    {# block.super: 親テンプレートのブロック内部の値をそのまま保持した変数 #}


外部テンプレートの読み込み
--------------------------

.. code-block:: django

  {% include "_message.html" %}

- ヘッダーフッターなど部品化したテンプレートを別のテンプレートから読み込む場合など


静的ファイルの配信 URL 取得
---------------------------

.. code-block:: django

  {# 利用する前に機能をロードしておく #}
  {% load static %}
  {% static "images/logo.png" %}
  {% static "shop/images/no-image.png" %}

- 静的ファイルの配信 URL を取得するためのタグ
- Django デフォルトで使える組み込みタグではないのでロードが必要
- ``extends`` している場合、継承元で ``load`` していても自身のテンプレートでは有効にならないので、
  つど ``load`` する必要がある


url
---

.. code-block:: django

  {# URL 逆引き #}
  {% url "index" %}
  {% url "accounts:login" %}
  {% url "shop:detail" book.id %}
  {% url "shop:detail" book_id=book.id %}


自動エスケープ制御
------------------

.. code-block:: django

  {% autoescape off %}
    自動エスケープがオフになる範囲
  {% endautoescape %}


ベーステンプレートを用意しよう
==============================
- 現場で使える Django の教科書《基礎編》 P.79 参照のこと

  - どのテンプレートにも書くような共通の内容は base.html テンプレートする


独自のテンプレートタグとフィルタ
================================
カスタムタグやカスタムフィルタをつくることができる。

- `Django documentation > 独自のテンプレートタグとフィルタ <https://docs.djangoproject.com/ja/2.2/howto/custom-template-tags/>`_
