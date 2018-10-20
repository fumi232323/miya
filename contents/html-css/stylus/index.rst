.. article::
   :date: 2018-10-18
   :title: stylus を使う
   :category: css
   :tags:
   :canonical_url: /html-css/stylus/
   :draft: false


=============
stylus を使う
=============


リファレンス
============
http://stylus-lang.com/

インストール
============
1. インストールガイド

    http://stylus-lang.com/#get-styling-with-stylus

2. Node.js をインストールする

    https://nodejs.org/en/

3. ガイドに書いてある通りにコマンドを実行する。

    .. code-block:: console

      $ sudo npm install stylus -g
      /usr/local/bin/stylus -> /usr/local/lib/node_modules/stylus/bin/stylus
      + stylus@0.54.5
      added 20 packages from 46 contributors in 1.399s

4. インストールできたか確認する。

    .. code-block:: console

      $ stylus -V
      0.54.5


使い方
============
- ファイルの拡張子は、 ``.styl`` もしくは ``.stylus`` とする。
- 初めて ``styl`` ファイルを作成すると、 ``File Watchers`` を有効にするか否か聞かれるので (PyCharm の上部に表示される) ``OK`` を押下すると、自動ビルドされる。

  以降、変更を検知すると勝手にビルドされる。

  - ログ

    .. code-block:: console

      ...
      2018-10-18 22:14:19,451 INFO Building files/static/css/style-sp.css
      2018-10-18 22:14:19,454 INFO Building files/static/css/style-sp.styl
      ...

  - ``File Watchers`` の設定変更は、こちらで行う。

    .. figure :: file-watchers.png


- PyCharm 上ではこんな風に見える

  .. figure :: stylus-image.png

- HTML の ``<head></head>`` には、こう書く

  .. code-block:: html

    <link rel="stylesheet" href="/static/css/style-sp.css">

  - ``style-sp.styl`` と書いても認識してくれない

- ``.styl`` ファイルに、stylus の書き方とふつうの CSS の書き方を併記できる。


  - ``style-sp.styl`` にこう書くと、

    .. code-block:: css

      @charset "UTF-8"

      /* --------------------------------
       * stylus の書き方
       * -------------------------------- */
      body
        background-color #f8b500

      /* --------------------------------
       * CSS の書き方
       * -------------------------------- */
      body {
        background-color: #ed6d3d;
      }


  - ``style-sp.css`` にこう出力される

    .. code-block:: css

      @charset "UTF-8";
      /* --------------------------------
       * stylus の書き方
       * -------------------------------- */
      body {
        background-color: #f8b500;
      }
      /* --------------------------------
       * CSS の書き方
       * -------------------------------- */
      body {
        background-color: #ed6d3d;
      }
