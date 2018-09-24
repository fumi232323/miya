.. article::
   :date: 2018-09-02
   :title: Redshift のメモ
   :category: redshift
   :tags:
   :canonical_url: /redshift/note/
   :draft: false


================
Redshift のメモ
================


データ分散
================

DISTSTYLE
-----------
テーブル全体のデータ分散スタイルを定義する。

- ``EVEN`` 、 ``KEY`` 、 ``ALL`` の3種類ある

  - `CREATE TABLE AS <https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/r_CREATE_TABLE_AS.html>`_

    ::

      テーブルにどの分散スタイルを選択するかによって、データベースの全体的なパフォーマンスが左右されます。

  - `分散スタイル <https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/c_choosing_dist_sort.html>`_
  - `分散の例 <https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/c_Distribution_examples.html>`_

- よく考えて選んだほうが良さそう...


DISTKEY
-----------
分散キーの列名または位置番号を指定する。



文字データ型
==================

マルチバイト文字の VARCHAR
--------------------------
CHAR および VARCHAR のデータ型は、文字単位でなくバイト単位で定義される。

- `文字型 <https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/r_Character_types.html>`_

  ::

    例えば、VARCHAR(12) 列には、シングルバイト文字なら 12 個、2 バイト文字なら 6 個、3 バイト文字なら 4 個、4 バイト文字なら 3 個含めることができます。

  - 3バイト文字と4バイト文字の違いがわかってない・・


MAX キーワード
--------------------------
CHAR と VARCHAR は、 MAX キーワードを指定できる。

- `ストレージと範囲 <https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/r_Character_types.html#r_Character_types-storage-and-ranges>`_

  ::

    MAX 設定は列幅を定義します。CHAR の場合は 4096 バイトであり、VARCHAR の場合は 65535 となります。


  .. code-block:: sql

    create table test(col1 varchar(max));


圧縮
================

圧縮分析
-----------
圧縮分析レポート取得できるコマンドがある。

- `ANALYZE COMPRESSION <https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/r_ANALYZE_COMPRESSION.html>`_

  ::

    圧縮分析を行い、分析されたテーブルの推奨列エンコードスキームのレポートを生成します。レポートには、列ごとに現在のエンコードと比較したディスク容量の圧縮可能率の推定値が含まれます。


列圧縮タイプの選択
--------------------
- テーブルの作成後に変更することはできない
- ALTER 文で列追加する場合は指定できる
- 圧縮すると、クエリパフォーマンスが向上する

  - `列圧縮タイプの選択 <https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/t_Compressing_data_on_disk.html>`_

    ::

      テーブルの作成後に列の圧縮エンコードを変更することはできません。ALTER TABLE コマンドを使用して列をテーブルに追加する際には、列のエンコードを指定できます。


    ::

      圧縮は、データの格納時にそのサイズを小さくする列レベルの操作です。圧縮によってストレージスペースが節約され、ストレージから読み込まれるデータのサイズが小さくなり、ディスク I/O の量が減少するので、クエリパフォーマンスが向上します


圧縮エンコード
--------------
- 指定しないと自動的に割り当てられる。
- 型によっては、CREATE TABLE または ALTER TABLE ステートメントで明示的に指定した方が良さそう。

  - `圧縮エンコード <https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/c_Compression_encodings.html>`_

    ::

      圧縮エンコードは、行がテーブルに追加されるときにデータ値の列に適用される圧縮のタイプを指定します。

      CREATE TABLE または ALTER TABLE ステートメントで圧縮が指定されていない場合、Amazon Redshift は以下のように自動的に圧縮エンコードを割り当てます。...
