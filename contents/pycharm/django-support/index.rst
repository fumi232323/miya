.. article::
   :date: 2018-09-30
   :title: Pycharm の Django support を使う
   :category: pycharm
   :tags:
   :canonical_url: /pycharm/django-support/
   :draft: false


==========================================
Pycharm の Django support を使う
==========================================

ガイド
======
- `Djangoのサポート <https://pleiades.io/help/pycharm/django-support7.html>`_
- `Django support <https://www.jetbrains.com/help/pycharm/django-support7.html>`_


Django Support を有効にする
====================================

ガイド
^^^^^^^^
- `Enabling or disabling Django support <https://www.jetbrains.com/help/pycharm/django-support.html>`_
- `Djangoのサポートを有効または無効にする <https://pleiades.io/help/pycharm/django-support.html>`_


Django Support を有効にする
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
1. Preferences
2. Languages & Frameworks -> Django

    - Enable Django Support: ``ON``
    - Django project root: ``apps`` ( `settings.py および manage.py ファイルを格納するディレクトリ` を指定するっぽかったので。合っているかどうかわからない。)
    - Settings: ``settings/_.py`` ( `Djangoプロジェクトルートの下にある名前が *settings*.pyに一致するファイルです` と書いてあったのでたぶんこれだろう。)


Template language を設定する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
1. Preferences
2. Languages & Frameworks -> Python Template Languages

    - Template language: ``Django``


モデル依存関係図の表示
====================================
これすごい

ガイド
^^^^^^^^
- `Viewing Model Dependency Diagram <https://www.jetbrains.com/help/pycharm/viewing-model-dependency-diagram.html>`_
- `モデル依存関係図の表示 <https://pleiades.io/help/pycharm/viewing-model-dependency-diagram.html>`_


手順
^^^^^^^^
1. Project tool window (プロジェクトがツリー状に表示されている部分) で、プロジェクトを選択して右クリック
2. Diagrams -> Show Diagram
3. Select Diagram Type ポップアップ上で ``Django Model Dependency Diagram`` を選択


    .. figure :: SelectDiagramType.png

Tips
^^^^^^
- プロジェクト内のアプリケーションを選択すると、選択したアプリケーションのみの Diagram が表示できる
- クラス図みたいなのも表示できる ( ``Python Class Diagram`` )
- 上下のスクロールはできるが、左右のスクロールがうまくない...
