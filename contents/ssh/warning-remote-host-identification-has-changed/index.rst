.. article::
   :date: 2018-08-05
   :title: サーバーに ssh 接続しようとしたら、 WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED! と言われた
   :category: ssh
   :tags:
   :canonical_url: /ssh/warning-remote-host-identification-has-changed/
   :draft: false


何が起こったか
=========================
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

よく鍵を間違えるので、また間違えたのかと思ったら・・・


違ってた！
==========
チームの方が教えてくれました。

- 接続先サーバーの RSA 公開鍵のフィンガープリントが変わってますよ、というエラー。
- サーバー入れ替えとかすると、RSA 公開鍵のフィンガープリントは変わる。
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
3. 汝が接続しようとしているサーバーの fingerprint はこちらか？と尋ねられるので、 ``yes`` と答える。

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


ssh-keygen のおさらい
================================

何者か
--------------------------
認証鍵の生成と管理・変換が役割。


秘密鍵・公開鍵のペアを作る
--------------------------

.. code-block:: bash

  $ ssh-keygen -t rsa

- ``~/.ssh/id_rsa`` (秘密鍵), ``~/.ssh/id_rsa.pub`` (公開鍵) にできあがる。

  - ``id_rsa`` が、秘密鍵(自分で持っておく、復号用)
  - ``id_rsa.pub`` が、公開鍵(サーバに置く、暗号用)

- ``-t type`` で、生成する鍵のタイプを指定できる。
- ``-f output_keyfile`` で、出力先ファイルを指定できる。
- ``-N new_passphrase`` で、パスフレーズ（鍵の鍵みたいなやつ）を指定できる。後からでもつけられる。


秘密鍵から公開鍵を取り出す
--------------------------

.. code-block:: bash

  $ ssh-keygen -y -f private_key_file


ためしてみる
~~~~~~~~~~~~~~~~~~~~~~
1. カレントディレクトリに、↓で rsa 鍵のペアを作成する。

  - ファイル名: ``id_rsa_test1``
  - コメント: ``fumi23``
  - パスフレーズ: ``fumi23``

    .. code-block:: bash

      $ ssh-keygen -t rsa -f id_rsa_test1 -C fumi23
      Generating public/private rsa key pair.
      Enter passphrase (empty for no passphrase): (ここで ``fumi23`` と入力した)
      Enter same passphrase again:  (ここで ``fumi23`` と入力した)
      Your identification has been saved in id_rsa_test1.
      Your public key has been saved in id_rsa_test1.pub.
      The key fingerprint is:
      SHA256:1WfZC2LtB0bkllFW1bTmSpleRbR+QSnARuYFvT7kLC0 fumi23
      The key's randomart image is:
      +---[RSA 2048]----+
      |          o===.*@|
      |          o+=o==+|
      |          o+.O*=o|
      |         .. ==O +|
      |        S   *= *.|
      |           Eo*+ .|
      |            oo.  |
      |                 |
      |                 |
      +----[SHA256]-----+

2. できあがった。

    .. code-block:: bash

      $ ls -la
      -rw-------   1 fumi23  staff   1766  8  5 16:40 id_rsa_test1
      -rw-r--r--   1 fumi23  staff    388  8  5 16:40 id_rsa_test1.pub

3. 公開鍵の中身を見てみる。

    末尾にコメントとして指定した文字列が付与されている。

    .. code-block:: bash

      $ cat id_rsa_test1.pub
      ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDfq5BPUHIRnrxBX1b3sp8OFTzFh3k9e1VZ3OXlRQRAKPTJlwdMR0apIatgq4KocFTTc4EKBksOVxOJShG1iVcUNFkhQ0kxpHTMyPHMyQdgpWAqaF5REOKMCI111xWgEC166zLUwZ1SdOHi/p2+5oDFhElsyjprro66o+uVluCD1VmfWORYYZlrMyUTtbdzHOO8xyT4k+yVMnuDJSLgfSGkCA/gXUi9vCqJf0p5iRt1owf520DSLLnkE5Cu9QxIdGDEBbS8lq53oJm5DyOcSXn+V2vKBv6pfjh+TJJNZ6PClrRI7Zk/aZFAkB/9XgqErbhU6mkHWWO9vmRavJh8Wspd fumi23

4. 秘密鍵から公開鍵を取り出してみる。

    パスフレーズを聞かれる。 3 と同じ公開鍵が取り出せた。

    .. code-block:: bash

      $ ssh-keygen -y -f id_rsa_test1
      Enter passphrase: (ここで ``fumi23`` と入力した)
      ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDfq5BPUHIRnrxBX1b3sp8OFTzFh3k9e1VZ3OXlRQRAKPTJlwdMR0apIatgq4KocFTTc4EKBksOVxOJShG1iVcUNFkhQ0kxpHTMyPHMyQdgpWAqaF5REOKMCI111xWgEC166zLUwZ1SdOHi/p2+5oDFhElsyjprro66o+uVluCD1VmfWORYYZlrMyUTtbdzHOO8xyT4k+yVMnuDJSLgfSGkCA/gXUi9vCqJf0p5iRt1owf520DSLLnkE5Cu9QxIdGDEBbS8lq53oJm5DyOcSXn+V2vKBv6pfjh+TJJNZ6PClrRI7Zk/aZFAkB/9XgqErbhU6mkHWWO9vmRavJh8Wspd

5. 公開鍵のほうは、RSA認証で接続したいサーバーの ``~/.ssh/authorized_keys`` に追記する。(サーバーを持っていないので、した気になる。)


公開鍵から fingerprint を表示
------------------------------------
fingerprint は、公開鍵にくっついているものらしい。

- カレントディレクトリ配下の公開鍵ファイルを指定して表示

  .. code-block:: bash

    $ ssh-keygen -l -f id_rsa_test1.pub
    2048 SHA256:1WfZC2LtB0bkllFW1bTmSpleRbR+QSnARuYFvT7kLC0 fumi23 (RSA)

- 秘密鍵を指定すると、ペアとなる公開鍵を探してその fingerprint を表示してくれる。

  .. code-block:: bash

    $ ssh-keygen -l -f id_rsa_test1
    2048 SHA256:1WfZC2LtB0bkllFW1bTmSpleRbR+QSnARuYFvT7kLC0 fumi23 (RSA)


known_hosts file からキーを削除
------------------------------------

.. code-block:: bash

  $ ssh-keygen -R hostname [-f known_hosts_file]


fingerprint とは何ですか
------------------------------------
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
- いやたぶん、一番最初、サーバーに公開鍵を置いたばかりのタイミングでは、`$ ssh-keygen -l -f id_rsa_test1.pub` したやつと同じフィンガープリントを送りつけてくるんだろう
- 試したい・・・

あ、わかった
ここだ

http://www.unixuser.org/~euske/doc/openssh/book/chap3.html
3.2.3. なりすましを防ぐしくみ

公開鍵、って言ってるやつは、ふたつに分かれていたんだ、ホスト公開鍵とホスト秘密鍵

通常はこのようなことが起こらないよう、 クライアントはサーバに接続した瞬間に、まず暗号化された通信を介して そのサーバのホスト鍵 (host key) を確認し、 それが本当に自分のログインしたいサーバであるかどうか確かめます。 ホスト鍵はホスト公開鍵とホスト秘密鍵に分かれており、 クライアント上には通常 known_hosts と呼ばれるファイルがあり、 ここには特定の IPアドレス (とホスト名) をもつサーバのホスト公開鍵が登録されています。 ホスト秘密鍵はサーバマシン内のディスクに格納されており、 ネットワーク上に持ち出されることはありません。 クライアントは、まずこの known_hosts ファイル内に登録されているホスト公開鍵と、 サーバから送られてくるホスト公開鍵を照合し (図 what-is-host-authentication)、 サーバが実際にこのホスト公開鍵に対応するホスト秘密鍵をもっているかどうか確認します。 この確認には公開鍵暗号技術が使われており、 サーバは実際のホスト秘密鍵をネットワーク上に送信することなく、ホスト秘密鍵の所有を クライアント側に証明できるようになっています (コラム - 公開鍵をつかった認証のしくみ 参照)。

TODO: あした、↑たしかめよう


宿題
=========================
- ``中間者攻撃`` とは何ですか。
- ssh-agent のほうもあやしいので(自分の理解が)、復習しておこう


参考にしたサイト
===================
- `SSH-KEYGEN(1) <https://www.freebsd.org/cgi/man.cgi?query=ssh-keygen&apropos=0&sektion=1&manpath=CentOS+6.5&arch=default&format=html>`_
- `秘密鍵/公開鍵の基本的な設定 <http://note.crohaco.net/2014/public-key-basic-config/>`_
- `第3章 OpenSSH のしくみ <http://www.unixuser.org/~euske/doc/openssh/book/chap3.html>`_



ありがとうございました。
