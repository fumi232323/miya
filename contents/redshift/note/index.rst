.. article::
   :date: 2018-09-02
   :title: Redshift のメモ
   :category: redshift
   :tags:
   :canonical_url: /redshift/note/
   :draft: false


diststyle all
----------------
- 分散の設定だった。 `DISTKEY` も載ってた。

::

  CREATE TABLE ステートメントで定義したオプションに応じてデータがどのように分散されるかを示しています。


- https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/c_Distribution_examples.html

マルチバイト文字の VARCHAR
------------------------------------------------
- https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/r_Character_types.html

::

  例えば、VARCHAR(12) 列には、シングルバイト文字なら 12 個、2 バイト文字なら 6 個、3 バイト文字なら 4 個、4 バイト文字なら 3 個含めることができます。

- 3バイト文字と4バイト文字の違いがわかってない・・


圧縮分析
-----------
- https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/r_ANALYZE_COMPRESSION.html


未整理
----------

::

  fumi23 [13:47]
  >https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/r_ANALYZE_COMPRESSION.html
  こんなのあるのかあ
  docs.aws.amazon.com
  ANALYZE COMPRESSION - Amazon Redshift
  圧縮分析を実行し、分析されたテーブルの推奨列エンコードスキームと圧縮可能率の推定値のレポートを生成します。

  fumi23 [13:56]
  - https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/t_Compressing_data_on_disk.html
  > テーブルの作成後に列の圧縮エンコードを変更することはできません。ALTER TABLE コマンドを使用して列をテーブルに追加する際には、列のエンコードを指定できます。

  ALTER 文にもつけられるのかあ・・
  docs.aws.amazon.com
  列圧縮タイプの選択 - Amazon Redshift
  圧縮は、データの格納時にそのサイズを小さくする列レベルの操作です。圧縮によりストレージ領域が節約され、ストレージから読み取るデータのサイズが縮小されます。
  > CREATE TABLE または ALTER TABLE ステートメントで圧縮が指定されていない場合、Amazon Redshift は以下のように自動的に圧縮エンコードを割り当てます。
  - https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/c_Compression_encodings.html
  docs.aws.amazon.com
  圧縮エンコード - Amazon Redshift
  圧縮エンコードは、行がテーブルに追加されるときにデータ値の列に適用される圧縮のタイプを指定します。


MAX 設定
----------

https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/r_Character_types.html#r_Character_types-char-or-character
> 「MAX 設定は列幅を定義します。CHAR の場合は 4096 バイトであり、VARCHAR の場合は 65535 となります。」
docs.aws.amazon.com
文字型 - Amazon Redshift
Amazon Redshift によってサポートされている文字型を使用する際に従うべき規則について説明します。
> text    制限なし可変長
https://www.postgresql.jp/document/9.6/html/datatype-character.html
