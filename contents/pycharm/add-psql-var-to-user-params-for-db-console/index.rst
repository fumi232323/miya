.. article::
   :date: 2018-09-26
   :title: PyCharm の Database Console で PostgreSQL のプレースホルダー (インタポレーション) を SQL パラメーターとして使えるようにする
   :category: pycharm
   :tags:
   :canonical_url: /pycharm/add-psql-var-to-user-params-for-db-console/
   :draft: false


====================================================================================================================================
PyCharm の Database Console で PostgreSQL のプレースホルダー (インタポレーション) を SQL パラメータとして使えるようにする
====================================================================================================================================


ガイド
======
- `PyCharm ヘルプ -> ユーザー・パラメーター <https://pleiades.io/help/pycharm/settings-tools-database-user-parameters.html>`_
- `PyCharm Help -> User Parameters <https://www.jetbrains.com/help/pycharm/settings-tools-database-user-parameters.html>`_

手順
============
1. Database Console 上で スパナのボタン ( ``Settings`` ) -> User Parameters

    (もしくは、 Preferences -> Tools -> Database -> User Parameters)

2. こう

    .. figure :: user-parameters.png

    - ``In scripts`` を ON にしないと、 Database Console で使用できない

3. Apply ボタンを押下
4. OK ボタンを押下
