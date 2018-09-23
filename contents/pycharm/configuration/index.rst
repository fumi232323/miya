.. article::
   :date: 2018-09-23
   :title: PyCharm の設定
   :category: pycharm
   :tags:
   :canonical_url: /pycharm/configuration/
   :draft: true


==========================================
PyCharm の設定
==========================================

ガイド
======
- `PyCharm ヘルプ <https://pleiades.io/help/pycharm/>`_


設定
======

エディター設定
----------------

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
----------

Django Support を有効にする
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Preferences
2. Languages & Frameworks -> Django

    - Enable Django Support: ``ON``
    - Django project root: ``apps`` ( `settings.py および manage.py ファイルを格納するディレクトリ` を指定するっぽかったので。合っているかどうかわからない。)
    - Settings: ``settings/_.py`` ( `Djangoプロジェクトルートの下にある名前が *settings*.pyに一致するファイルです` と書いてあったのでたぶんこれだろう。)

Template language を設定する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Preferences
2. Languages & Frameworks -> Python Template Languages

    - Template language: ``Django``


Sources Root
----------------
- Sources Root を設定

  1. 左ツリーの ``apps`` を右クリック
  2. メニューの ``Mark Directory as`` -> ``Sources Root``

- Template Folder を設定

  1. 左ツリーの ``templates`` を右クリック
  2. メニューの ``Mark Directory as`` -> ``Template Folder``



TODO: やってみる
============================
- SQL
  - SQL 実行結果をファイルにも出せる
- デバッガー
  - セロリーのデバッグもできる、Django のデバッグもできる
  - テスト実行中にもブレークポイントはって、デバッグできる
  - テンプレートの中でもデバッグポイントで止められる
- aws に ssh 接続して redshift につないだりもできる
- HTML のマークでる *
- post の中身を見られる
- Django ORM とか SQLA も
- Django サポート、モデル間のER図みたいなのとかも出してくれる
- リファクタリングできる
- コードデプロイもできる
- テストも便利
- 再起動したりクリーンインストールとかしても治らないときは、ジェットブレインさんに言う
