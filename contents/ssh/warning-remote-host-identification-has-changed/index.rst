.. article::
   :date: 2018-08-05
   :title: サーバーに ssh 接続しようとしたら、 WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED! と言われた
   :category: ssh
   :tags:
   :canonical_url: /ssh/warning-remote-host-identification-has-changed/
   :draft: true


===========================================================
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED! と言われた
===========================================================

TODO: 残りを整理する。


事象
====
サーバーに ssh 接続しようとしたら、こんなエラーが出た。

  - エラーメッセージ(だいたいこんな感じ)

    .. code-block:: bash

      $ ssh {接続先サーバー}
      @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
      @    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
      @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
      IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
      Someone could be eavesdropping on you right now (man-in-the-middle attack)!
      It is also possible that a host key has just been changed.
      The fingerprint for the RSA key sent by the remote host is
      SHA256:{ここに fingerprint }.
      Please contact your system administrator.
      Add correct host key in /Users/fumi23/.ssh/known_hosts to get rid of this message.
      Offending RSA key in /Users/fumi23/.ssh/known_hosts:3
      RSA host key for {ここに 接続先サーバーの HostName } has changed and you have requested strict checking.
      Host key verification failed.
      ssh_exchange_identification: Connection closed by remote host

原因
====
チームの方が教えてくれました。 (ありがとうございました。)

- 接続先サーバーの RSA 公開鍵のフィンガープリントが変わっていたため。

  - 上記のエラーは、接続先サーバーの RSA 公開鍵のフィンガープリントが変わってますよ、というエラー。
  - サーバーの入れ替えなどすると、 RSA 公開鍵のフィンガープリントは変わる。
  - `エラーメッセージちゃんと読むと、MITM攻撃ありえるから、気を利かしてエラーにしてる感じ` なのだそうです。


解決方法
==========
1. known_hostsファイルから、接続先サーバーの HostName に属する鍵をすべて取り除く。

    .. code-block:: bash

      $ ssh-keygen -R {ここに 接続先サーバーの HostName }
      # Host {ここに 接続先サーバーの HostName } found: line 3
      /Users/fumi23/.ssh/known_hosts updated.
      Original contents retained as /Users/fumi23/.ssh/known_hosts.old

2. ssh でアクセスし直す。
3. 「あなたが接続しようとしているサーバーの fingerprint はこれか？」と尋ねられるので、 ``yes`` と答える。

    .. code-block:: bash

      $ ssh {接続先サーバー}
      The authenticity of host '{ここに 接続先サーバーの HostName } ({ここに 接続先サーバーの IPアドレス })' can't be established.
      ECDSA key fingerprint is SHA256:(ここに 新しい fingerprint ).
      Are you sure you want to continue connecting (yes/no)? yes
      Warning: Permanently added '{ここに 接続先サーバーの HostName },{ここに 接続先サーバーの IPアドレス }' (ECDSA) to the list of known hosts.
      The authenticity of host '{ここに 接続先サーバーその2の HostName } (<no hostip for proxy command>)' can't be established.
      RSA key fingerprint is SHA256:(ここに 新しい fingerprint その2 ).
      Are you sure you want to continue connecting (yes/no)? yes
      Warning: Permanently added '{ここに 接続先サーバーその2の HostName }' (RSA) to the list of known hosts.
      Last login: Fri Jul 13 15:43:09 2018 from 172.31.7.69

4. 接続できました。


fingerprint とは何ですか
------------------------
- 流れとしては、

  1. 秘密鍵は自分持ってる
  2. 公開鍵はサーバーに書く
  3. ssh 1回目に、「あなたが接続しようとしているサーバー（が持ってるRSA公開鍵の）フィンガープリントこれでいい？」って聞かれる
  4. 「いいよ」っていうと自分のマシンの  ``known_hosts`` に書かれる。

      - ``known_hosts`` ファイルに、

        .. code-block:: bash

          サーバーのHostName 鍵のタイプ 謎の文字列

        の形式で書かれる。

      - ``known_hosts`` は覚書きみたいなもの。このサーバーは知ってるひと、伊原に住んでる fumi さんでしょ、合言葉は「おとといきやがれ」。みたいな。
      - ``known_hosts`` に書かれる謎の文字列は、公開鍵の文字列と似ているけれど、違う。なんか暗号化とかしてるの？フィンガープリントを。

    5. ssh 2回目以降は、 ``known_hosts`` に書いておいた覚書きを照会して、知り合いか否かを判定する。

TODO: ここから整理中

- なんのために、フィンガープリントを送ってくるかというと、「あなたが接続しようとしているサーバーはこちらでよろしいですか？」という確認のため。
- 公開鍵をそのまま送っちゃうと危険だから、 ``SHA256`` (ハッシュ) して、送ってくれる。
- サーバー入れ替えとかすると、フィンガープリントは変わる (とのこと)。
- でも鍵が変わっているわけではない
- ``SHA256`` はハッシュだから、同じ元値からは必ず同じハッシュ値が生成されるはず
- ということは、単純に公開鍵から生成しているわけではなさそう
- 公開鍵 ( +α ) から生成されるんだろう
- 送り主のサーバーは公開鍵しか持ってないしな
- でも変えられちゃったら、念のため手元にとっておいた公開鍵に対応するフィンガープリントなのかわからなくなっちゃう・・
- いやたぶん、一番最初、サーバーに公開鍵を置いたばかりのタイミングでは、`$ ssh-keygen -l -f id_rsa_test1.pub` したやつと同じフィンガープリントを
  送りつけてくるんだろう
- 試したい・・・

あ、わかった
ここだ

http://www.unixuser.org/~euske/doc/openssh/book/chap3.html
3.2.3. なりすましを防ぐしくみ

公開鍵、って言ってるやつは、ふたつに分かれていたんだ、ホスト公開鍵とホスト秘密鍵

通常はこのようなことが起こらないよう、 クライアントはサーバに接続した瞬間に、まず暗号化された通信を介して そのサーバのホスト鍵 (host key) を確認し、
それが本当に自分のログインしたいサーバであるかどうか確かめます。
ホスト鍵は ``ホスト公開鍵`` と ``ホスト秘密鍵`` に分かれており、 クライアント上には通常 known_hosts と呼ばれるファイルがあり、
ここには特定の IPアドレス (とホスト名) をもつサーバの ``ホスト公開鍵`` が登録されています。
ホスト秘密鍵はサーバマシン内のディスクに格納されており、 ネットワーク上に持ち出されることはありません。
クライアントは、まずこの known_hosts ファイル内に登録されているホスト公開鍵と、
サーバから送られてくるホスト公開鍵を照合し (図 what-is-host-authentication)、
サーバが実際にこのホスト公開鍵に対応するホスト秘密鍵をもっているかどうか確認します。
この確認には公開鍵暗号技術が使われており、 サーバは実際のホスト秘密鍵をネットワーク上に送信することなく、
ホスト秘密鍵の所有を クライアント側に証明できるようになっています (コラム - 公開鍵をつかった認証のしくみ 参照)。

TODO: あした、↑たしかめよう


宿題
====
- ``中間者攻撃`` とは何ですか。


参考サイト
==========
- `SSH-KEYGEN(1) <https://www.freebsd.org/cgi/man.cgi?query=ssh-keygen&apropos=0&sektion=1&manpath=CentOS+6.5&arch=default&format=html>`_
- `秘密鍵/公開鍵の基本的な設定 <http://note.crohaco.net/2014/public-key-basic-config/>`_
- `第3章 OpenSSH のしくみ <http://www.unixuser.org/~euske/doc/openssh/book/chap3.html>`_


ありがとうございました。
