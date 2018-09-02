.. article::
   :date: 2018-09-02
   :title: PyCharm 使うときメモ
   :category: pycharm
   :tags:
   :canonical_url: /pycharm/note/
   :draft: false



定義ジャンプと source root を設定
==========================================

::

  shimizukawa [11:47]
  PyCharm使ってるのであれば、 `dashboard.views.sales_summary_store` にカーソルがある状態でメニューの `Navigate -> Declaration` でそのコードの定義位置にジャンプできるよ
  自分の環境だとショートカットキーは Cmd+B

  fumi23 [11:48]
  （やってみます・・・
  うーん、、can not find declaration go to … になってしまう。。

  shimizukawa [11:51]
  おや。プロジェクトの設定かな。こんど出社したときにでも見るので声かけてー


  fumi23 [11:52]
  モジュールが違ってもできますか？

  shimizukawa [11:52]
  赤い ~~ が出てたらうまくいかない。PyCharmがimportを認識できてない状態なので
  もちろんモジュール違っててもできるよ

  kashew_nuts [11:52]
  (source rootを設定してないとかかな。 > プロジェクトの設定

  fumi23 [11:53]
  赤なみなみが出ている！


  shimizukawa [11:54]
  1. 左ツリーの `apps` を右クリック
  2. メニューの `Mark Directory as` -> `Sources Root`

  fumi23 [11:54]
  あああ！！
  出なくなりました！！！
  わー、全部でなくなった
  おおお、これもできるようになりました
  >https://beproud.slack.com/archives/G038LCD9V/p1531968437000084
  shimizukawa
  PyCharm使ってるのであれば、 `dashboard.views.sales_summary_store` にカーソルがある状態でメニューの `Navigate -> Declaration` でそのコードの定義位置にジャンプできるよ
  Posted in p-booklista-salesJul 19th

  shimizukawa [11:55]
  (´･ω･`):tada:

  fumi23 [11:56]
  この赤なみなみ、rookies のときからずっと「出てるなあ」と思っていたのですが、そういうものだと思い込んでました・・・・・・・・・・
  なんと・・・・・
  shimizukawa kashew_nuts++ Sources Root

  haro APP [11:57]
  Sources Root(shimizukawa: 759GJ)
  Sources Root(kashew_nuts: 257GJ)

  shimizukawa [11:57]
  思い込みよくないね
  マウスカーソル重ねれば、警告内容も表示されるよ
  :yojira:
  じゅにじら APP [11:58]
  :meat_on_bone:  ＼(・ω・＼｀)ミ
  (Powered by slack-reminder)

  fumi23 [11:59]
  おおお、警告が減って、すごくすっきりしました、flake8警告とか見逃さなくなりそう・・

  shimizukawa [12:00]
  割れ窓理論っていうのがあって、
  窓が割れた状態を「そういうものだ」と思うと、窓を割って良いと思う人がやってきて割り始める (edited)

  fumi23 [12:02]
  ！！それきいたことがあります、その理論で壁の落書きを減らした話（落書き放置しとくと落書きされるので、落書きはすぐ消した、という話
  はー、「そういうものだ」よくない・・・（わたしは「そういうものだ」傾向けっこうある・・・・
  shimizukawa++ 割れ窓理論

  haro APP [12:05]
  割れ窓理論(shimizukawa: 760GJ)

  shimizukawa [12:05]
  そうそう > 落書き

  fumi23 [12:06]
  ありがとうございます:pray:
  そして、これ、めちゃくちゃ便利・・・
  >https://beproud.slack.com/archives/G038LCD9V/p1531968452000011
  shimizukawa
  自分の環境だとショートカットキーは Cmd+B
  Posted in p-booklista-salesJul 19th

  shimizukawa [12:08]
  便利ついでに、もうひとつ
  `Edit-> Find -> Find Usages`  （私の環境では Option + F7）も便利

  takanory [12:09]
  夜の校舎 窓ガラス壊してまわった〜♪

  shimizukawa [12:09]
  関数の定義（defの行）にカーソル合わせて `Find Usages` すると、利用箇所を検索してくれる

  fumi23 [12:09]
  ほーー

  shimizukawa [12:10]
  さっきのDeclarationの逆だね

  fumi23 [12:10]
  おおお！
  `Sources Root` 設定したら、ちゃんと使っている箇所だけ、出るようになりました
  （さっきまでは、同じ関数名のところが全部出てきてた
  shimizukawa++ Find Usages

  haro APP [12:13]
  Find Usages(shimizukawa: 761GJ)

  shimizukawa [12:13]
  うむ

  fumi23 [12:14]
  PyCharm 便利・・・

  kashew_nuts [12:16]
  ちなみに `Cmd+b` で定義ジャンプした後は、 `Cmd+[` で戻れる (edited)

  fumi23 [12:16]
  （やってみよう・・・
  ！！
  便利・・便利・・・
  kashew_nuts++ `Cmd+[` で戻れる

  haro APP [12:18]
  `Cmd+[` で戻れる(kashew_nuts: 258GJ)