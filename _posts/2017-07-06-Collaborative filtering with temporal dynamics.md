---
layout: post
title:  "論文紹介: Collaborative Filtering with Temporal Dynamics"
date: 2017-07-06 05:27:02 +0900
categories: jekyll update
---

## この論文を選んだ理由

今から約10年ほど前、2006年から2009年にかけて[Netflix Prize](https://en.wikipedia.org/wiki/Netflix_Prize)と呼ばれる100万ドルの賞金をかけたコンペティションが行われていた時期があった。
最終的にBellKorを筆頭にしたチームが優勝することになるが、本論文の著者であるYehuda Korenはその優勝チームメンバーの一人である。

開催を通してMatrixFactorizationやアンサンブルによる精度向上といったアルゴリズムの進化と同時に、結果に大きく影響したのがこの論文で提唱されるような時間経過による嗜好の変化を予測モデルに取り入れることだった。

この結果の解釈は様々だとは思うが、情報推薦の対象は常に人間であるということ、人間の行動や認知特性を理解することがに推薦システムにおいてもブレイクスルーにつながること、ということを気づかせてくれるという意味で示唆に富んだ論文であると思う。

## 概要

ユーザの興味や嗜好は常に変化し続け、プロダクトに対する消費者の評価は時間と共に揺れ動く。
このような時間的ダイナミクスは推薦システムのモデルをデザインする際に重要な要素の一つである。
既存の協調フィルタリングは時不変的なモデルを前提としており、時間経過による評価の揺れや嗜好性の変化を捉えることができなかった。
また直近のアイテムにのみ重みを付けるような時間経過モデルはあまりに多くのシグナルを無視しているためうまく動作しない。より洗練されたアプローチが求められている。
ここではアイテムとユーザ評価の時間経過に伴う変化とユーザの嗜好性の変化を取り込んだ新しい協調フィルタリングのモデルを提案する。
またNetflixの評価データセットを用いたRMSEを指標とした実験では、これまで報告されたモデルよりも優れていたことが示された。

## ACM Refs
```
Yehuda Koren. 2009. 
Collaborative filtering with temporal dynamics. 
In Proceedings of the 15th ACM SIGKDD international conference on Knowledge discovery and data mining (KDD '09). 
ACM, New York, NY, USA, 447-456. 
DOI: https://doi.org/10.1145/1557019.1557072
```

## 目的と背景

### 時間経過に伴う嗜好性の変化

データマイニングにおいて時間経過に伴った対象モデルの変化を捉えることは重要な観点である。Netflixの評価データセットにおいてもこれは同様であった。

特徴的なものとして、例えば下図のように2004年初頭からなぜか急にユーザの平均レーティング値が高まる現象や、過去の映画であればあるほど平均レーティングが高まる現象が知られている。

![](https://gyazo.com/b0a537e722ce44f493a32a43eac2c301.png)

またレーティングという意味では時間経過につれて評価の意味が変化したりする。
一年前の評価3と現在の評価3は別物かもしれない。
また時間とともにあるジャンルから他のジャンルへ興味が移ったり、共演者や同じ監督の他作品に興味が移っていくこともある。

このような時間変化に伴う嗜好性の変化は既存の協調フィルタリングでは捉えきれない。なぜならナイーブな協調フィルタリングは時不変的なモデルを前提としているからである。短期間のうちに見ても、一年後に見ても評価の上では同じことになっている。

このような背景を踏まえ、本論文では時間経過に伴うユーザとアイテムの嗜好性の変化を推薦予測値に取り込む、新しい協調フィルタリングのモデルを提案している。


### 推薦手法

#### 予測評価値のベースモデル

まずベースとなるモデルは以下を参考にするとわかりやすい。

![](https://gyazo.com/08e7f8dac910da97f8c110d53f935f9c.png)
 - 引用:http://videolectures.net/kdd09\_koren\_cftd/ のスライド17枚目

r\_uiは評価予測値、μはすべてのアイテムの平均評価、b\_iはアイテムバイアス(アイテムの評価が全体と比べてどのくらい高いか/低いか)、b\_uはユーザバイアス(高い点を付けやすいか低い点を付けやすいか)、q\_i以降はMatrixFactorizationを用いたCF評価項(ユーザとアイテム間の関連を次元fで圧縮)、を示している。


またCF評価項に関しては以下のようなSVD++と呼ばれる拡張も施している。

![](https://gyazo.com/b95f99e03cd9a2cbf1030bd5f25f3ef6.png)

ここで面白いのは末項にある

>|R(u)|^(-1/2)Σ(y\_j)

の部分で、ここは著者が前年に出しているKDDの論文[1]に詳細があるが、過去に閲覧した映画といった直接評価していない暗黙的なユーザフィードバックをCFのファクターモデルに加味するというもの。
|R(u)|^(-1/2)は正規化部分であり、y\_jは各アイテムに対するユーザの何かしらのレーティングを示す。


#### 時間経過とベースライン

この評価予測値(r\_ui)の右辺のμとb\_i, b\_uの項をこの論文ではベースラインと呼んでいる。これらに対して時間tで補正をかける。最終的なモデルは以下のようなものになる。

![](https://gyazo.com/ffbe0788c4d67f0fb079302ff32a1c58.png)

各項の具体的な詳細は4.2節を参考してもらうとして、要は基準となる時間をtとしたときに、その時間に近い評価データに重みを付けるという実装をしていると思えばよい。
例えばユーザが評価した時期に近いデータに対して重みが付くというイメージになる。
また末項のb\_i_Bin(t)は全アイテムを10週ごとに30個のグループに分けた際に各アイテムが所属するグループから算出されたバイアス値である。

このような対処によってアイテムとユーザの単体での時間的なバイアスを補正することが可能になる。

RMSEを用いた評価実験では各ユーザとアイテムごとの評価値に時間バイアスを加味しただけでも約0.02の精度向上が見られた。


#### 時間経過とファクターモデル

前節はアイテムとユーザのそれぞれ単体での時系列バイアスを焦点としていたが、ユーザのアイテムへの嗜好性の変化を捉えることはできていなかった。
このような時間経過による興味の変化を捉えるため、以下のようにファクターモデルの部分に時間要素を取り入れる。

![](https://gyazo.com/f32b8b1845822b5ff96c78a5372df1b6.png)

p\_u(t)はユーザの嗜好をベクトル化したもので、各次元が期間ごとの時間バイアスを加味しているのが特徴である。

ユーザとアイテムに関連する興味の変化を評価値予測に取り入れたところ以下のように時間の次元を増やすほど評価値の予測精度の改善が見られた。

![](https://gyazo.com/dd7f24ffde9224c7fd3173695753e683.png)



#### 時間経過と近傍モデル

また本論文で提案する時間経過モデルは、MatrixFactorizationを用いない近傍モデルに対して適用することもできると主張している。具体的には以下のようなモデルになる。

![](https://gyazo.com/22030c53e57e9f8050774c47a50db5c9.png)
 - 引用: http://videolectures.net/kdd09\_koren\_cftd/ のスライド26枚目


w\_ijは、CF的な評価値から求められたweightである。論文ではc\_ijというアイテムの特徴を用いた類似度?と思われるweightもあるが、スライド上には載っていないのが個人的には気になった。

ちなみにRMSEは既存のstaticなモデルと比べて0.02ほどの改善が見られたとのこと。


## 実験研究からの知見

前述したようにNetflixの評価セットでは2004年頃に急激に平均レーティングが上昇するという現象が起きている。その原因として仮説を三つほど立てそれぞれ検討している。

1. 2004年からNefflixが提供している推薦エンジンであるCinematchとGUIの精度が向上したため、良質のマッチングが良い評価につながっている
2. 五段階評価の意味が昔の作品と今の作品で異なっている。例えば最近の作品の5点評価は「とても好きだ」なのに対し、昔の作品は「一流の作品である」といったように、より客観的になる
3. 2004頃からユーザ層に変化がみられた可能性。例えばアーリーアダプターからマジョリティ層へ遷移したことで評価値の傾向が変わった。

解析の結果として、仮説3は2003年以前のユーザに焦点を合わせても同様の傾向があったことで棄却された。
仮説2については、本論文が提唱するbaselineの部分のみを評価モデルとして検証したところ、古いものになるほどユーザの平均評価が上昇していく傾向がみられた。
仮説1についても、interaction項を加味した評価検証で2004年頃から大きく評価値が向上するという結果を得た。（詳細については第6章を参照のこと）

さらなる仮説としては、新しい映画を探すときと比較して過去の映画を探す際にユーザはよりセレクティブになる傾向があり、注意深く選別した上で観る映画を決定している可能性があげられる。


## 所感

Netflixの100万ドルチャレンジは下図のように、第一段階としてMatrixFactorizationなどのアルゴリズムによる改善が行われ、次に本論文のような時間的な影響を加味した予測評価モデルの出現によって10%の精度向上が達成されたとされている。

![](https://gyazo.com/31bd5b0aa03d0ae692884dca936494ac.png)
 - 引用:http://videolectures.net/kdd09\_koren\_cftd/ のスライド5枚目

しかしNetflixのその後の顛末をみるとどうも優勝したアルゴリズムを実用化したわけではないらしい。

[なぜNetflixはNetflix Prizeで優勝したアルゴリズムを実用化していないのか？ - 射撃しつつ前転](http://d.hatena.ne.jp/tkng/20120517/1337221259)

億単位のデータセットではスケーラビリティの問題があったり、ストリーミング配信をメインとしたビジネスモデルへの転換といったことが要因となったようだ。

Netflix的には推薦の精度を上げることは手段の一つに過ぎず、ユーザが継続的に課金してくれるようなビジネスモデルが成り立つことが最も重要なわけで、推薦アルゴリズムだけに囚われるのは本末転等ということだろう。

さて、現実的な推薦システムやアプリケーションを考えると常にユーザやアイテムの評価は追加されるわけであり、特にユーザの評価を即時に反映できるようなモデルが望まれる。
また膨大なアイテムとユーザセットにおいて計算量を必要とするモデルは計算マシンの確保や時間的制約もありなかなか採用しにくい。
Staticな評価値データセットからの単なる予測評価の精度だけでなく、コールドスタートの状態でいかにユーザに満足させられる推薦を行えるか、大規模なデータセットにおいて現実的な時間で予測モデルを構築できるか、といった部分に焦点を当てるべきだろう。次回はこの観点から最近の動向を探ってみたい。


## 引用
[1]  Y. Koren. Factorization meets the neighborhood: a multifaceted collaborative filtering model. Proc. 14th ACM SIGKDD Int. Conf. on Knowledge Discovery and Data Mining (KDD’08), pp. 426–434, 2008.