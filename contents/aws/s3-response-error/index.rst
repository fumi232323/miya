.. article::
   :date: 2018-07-31
   :title: S3ResponseError の原因はなんと時刻ずれだった
   :category: aws
   :tags:
   :canonical_url: /aws/s3-response-error/
   :draft: false


何が起こったか
=========================
``storages.backends.s3boto.S3BotoStorage`` というのを使って、 AWS S3 にアクセスしようとしたら、こんなエラーが出た。

  - エラーメッセージ

    .. code-block:: shell

      S3ResponseError: S3ResponseError: 403 Forbidden
      <?xml version="1.0" encoding="UTF-8"?>
      <Error>
        <Code>RequestTimeTooSkewed</Code>
        <Message>The difference between the request time and the current time is too large.</Message>
        <RequestTime>Tue, 31 Jul 2018 07:40:44 GMT</RequestTime>
        <ServerTime>2018-07-31T07:58:01Z</ServerTime>
        <MaxAllowedSkewMilliseconds>900000</MaxAllowedSkewMilliseconds>
        <RequestId>428DE174AE9C91AA</RequestId>
        <HostId>VOjhVVVSgfq6rfE6Eh9pWr0+7bJGV9+4/sXeTeNnTVFi7JYAH5ZArd2NEP/hsgpgyJkMm/fNeCM=</HostId>
      </Error>

`403 Forbidden` と言っているし、以前はふつうにアクセスできたので、きっと ``settings.py`` の設定が変になっちゃってるのだろうと疑った。ものすごく疑った。

  - settings.py

    .. code-block:: python

      MY_BUCKET_NAME = 'my_bucket_name'
      DEFAULT_FILE_STORAGE = 'storages.backends.s3boto.S3BotoStorage'
      AWS_STORAGE_BUCKET_NAME = MY_BUCKET_NAME
      AWS_ACCESS_KEY_ID = 'ひみつ'
      AWS_SECRET_ACCESS_KEY = 'ひみつ'

違ってた！！
=========================
神々が教えてくれました・・・・

ここ↓（秒とかはてきとう）

- エラーメッセージ

  .. code-block:: shell

    <Message>The difference between the request time and the current time is too large.</Message>
    <RequestTime>Tue, 31 Jul 2018 07:40:00 GMT</RequestTime>
    <ServerTime>2018-07-31T07:58:01Z</ServerTime>

- ゲストOSの日時

  .. code-block:: shell

    (venv) [vagrant@localhost fumi23]$ date
    Tue Jul 31 16:40:00 JST 2018

- ホストOSの日時

  .. code-block:: shell

    host-machine:~ fumi23$ date
    2018年 7月 31日 火曜日 16時58分00秒 JST

時刻のなおしかた
=========================

.. code-block:: shell

  (venv) [vagrant@localhost fumi23]$ sudo date -s "07/31 17:02 2018"
  Tue Jul 31 17:02:00 JST 2018

わかったこと
========================
時刻がずれ過ぎているひとも出禁！( `403 Forbidden` )

- 常日頃からまわりの人に「エラーメッセージをよく読みましょう」と言ってもらっているし、自分でも「エラーをよく見るエラーをよく見る・・・」と散々唱えているのに、ちっとも見られていなかった。落ち込んだ。
- エラーメッセージが xml 形式で戻ってきたら、おおちゃくせずに整形してみようと思った。見やすくすると、大事なことも見逃しにくくなるかも。
- そういえば、ローカルVMから、3月はアクセスできていたけど、4月の中旬くらいからアクセスできなくなってた気がするので、そのタイミングで時刻ずれが先方の許容値を超えてたのかもしれない。

宿題
=========================
- ``storages.backends.s3boto.S3BotoStorage`` が何かあんまりわかっていない。
- ホストマシンとゲストマシンの時刻の自動同期設定ができるらしい。調べて設定しよう。

  - https://pc-karuma.net/virtualbox-install-guest-additions/

ありがとうございました。
