.. article::
   :date: 2018-09-23
   :title: PyCharm の設定
   :category: pycharm
   :tags:
   :canonical_url: /pycharm/configuration/
   :draft: false


==========================================
PyCharm の設定
==========================================

ガイド
======
- `PyCharm ヘルプ <https://pleiades.io/help/pycharm/>`_


エディター設定
==================

コード補完
^^^^^^^^^^^^^
1. Preferences
2. Editor -> General -> Code Completion


import 文を折りたたまない
^^^^^^^^^^^^^^^^^^^^^^^^^^
1. Preferences
2. Editor -> General -> Code Folding
3. Imports: OFF


PEP 8 警告の色を目立つようにする
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
1. Preferences
2. Editor -> Inspections -> PEP 8 coding style violation
3. 色変えたりする


Django
==================

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


Sources Root
==================

Sources Root を設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
``定義へジャンプ`` ( ``Cmd + B`` ) などが使えるようになる

  1. 左ツリーの ``apps`` を右クリック
  2. メニューの ``Mark Directory as`` -> ``Sources Root``


Template Folder を設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
``HTML マーク`` (押下すると template へジャンプできる) が出るようになる

  1. 左ツリーの ``templates`` を右クリック
  2. メニューの ``Mark Directory as`` -> ``Template Folder``
