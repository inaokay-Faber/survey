## Matching Cross Network for Learning to Rank in Personal Search  
Zhen Qin, Zhongliang Li, Mike Bendersky, Don Metzler  
Proceedings of the 2020 World Wide Web Conference (2020)  
https://research.google/pubs/pub48899/

### 一言まとめ
クエリと文書のテキスト情報だけでなく、ユーザのロケーションやコンテキストのような広範囲の情報を効率的に組み合わせたランキング学習についての研究。

はじめに、以下の2つの知見を実験により示している。
- 際立った素性（クエリと文書の埋め込み等）の相互作用を学習する際に、他の有用だが目立たない側面的情報が存在すると、ニューラルネットワークの学習が非効率になる
- 高次の特徴量生成のための最新手法をそのまま用いたとしても、重要な相互作用を学習するのは非効率である

これらの知見に基づいて、側面的情報を用いたランキング学習のためのシンプルで効果的なMatching Cross Network (MCN) 手法を提案している。

### 従来手法の限界の実証
#### データセット
擬似的にデータセットを生成して実験を行うことで従来手法の限界を実証している。以下にその手順を示す。

1. 一様分布(-1 +1)で初期化された20次元のベクトルを3つ生成する（それぞれf_1, f_2, f_3とする）
1. f_1とf_2のコサイン類似度を計算し、0より大きければy=1, 違えばy=0の正解ラベルを付与する
1. 上のセットを1件とし、1,000件ずつ訓練データ、検証データ、テストデータとする

f_1とf_2がそれぞれクエリと文書の埋め込み、f_3が目立たない側面的情報とすれば、一般的な検索と同様の設定になる。

#### 手法
上のデータセットを使って以下の3つの手法を比較する。いずれも3層ニューラルネットの入力となる。

1. f_1, f_2, f_3を連結(Concatenation)する
1. Latent Crossを用いる（f_2とf_3を入力とし、f_1を出力と乗ずる）
1. f_1とf_2のアダマール積（ベクトルの要素ごとに積を取る）を取ってからf_3と連結する

##### Latent Cross
Latent Crossはユーザタイプやプラットフォームといった潜在的な素性をニューラルネットへ組み込む手法である。
数式では以下のように表される。
w_latentは潜在的な素性、h_outは最終の隠れ層の出力を指す。
これによりw_latentがh_outの各次元のマスクとして機能する。

<img src="https://user-images.githubusercontent.com/61494140/79727726-c6c53e80-8327-11ea-9e1e-995facbacd7d.png" alt="Latent Cross" width="40%">


#### 結果
以下が3つの手法を比較した結果である。
結果から、単純な連結とLatent Crossはどちらも学習効率が悪いことがわかる。

<img src="https://user-images.githubusercontent.com/61494140/79729429-6be11680-832a-11ea-8d9d-e6d879ba8ca5.png" alt="result of sub-experiment" width="60%">

Latent Crossを直接適用できないことについて著者らは、f_1とf_2がモデル構造において離れすぎており、相互作用の学習が困難であることが原因であるという仮説を立てている。
一方で3つ目の手法は、2つの際立った素性を早い段階でマッチングさせることにより、学習が容易になっているとしている。

### 提案手法
#### Embedding Features Matching
クエリと文書の埋め込みにはいずれも単語埋め込みの平均を使用する。
その単語埋め込みもクエリと文書で同じものを用いる。
そのため2つの埋め込みは次元が同じになり、またモデルのパラメータ数を減らすことができる。

事前の実験によって得られた知見から、クエリと文書を早い段階でマッチングさせる。
マッチングの手法には、事前の実験と同じくアダマール積を用いる。
もちろんそれ以外のより複雑で意味を捉えたマッチング手法を使うといったこともできる。
アダマール積は単純に積を取るだけなので実装が簡単で、学習や推論時に追加でかかる時間やメモリが最小限に抑えられるというメリットがある。

#### Matching Cross Network (MCN)
MCNの構造を以下に示す。

<img src="https://user-images.githubusercontent.com/61494140/79733549-46570b80-8330-11ea-945d-9ed0868120a3.png" alt="MCN architecture" width="60%">

### 実験設定
#### データ
データにはGoogle Gmail, DriveのClick-throughデータを用いる。期間は2018年8月から12月で、クリック数が数億のクエリ。
Gmailクエリは平均して6個、Driveクエリは5個の候補文書がある。

80%を訓練データ、10%を検証データ、10%をテストデータとして用いる。
これらはrandom splitではなく、時系列的に古い方から訓練データ→検証データ→テストデータとなるように分割をしている。

またクエリと文書は、複数のユーザにまたがって高頻度に使用される単語と文字n-gramのみを使用する。
これはユーザのプライバシーを保護する目的で行なっている。

データに含まれる側面的情報は例えば以下のようなものがある。
- 時間帯などの状況的なもの
- ユーザの以前のクリックなどユーザに関するもの
- ドキュメントの古さなどの非テキストな文書属性

データはPosition Bias（より上に位置するものがクリックされやすい）を含むので、以下の研究で提案されている補正手法を用いて緩和させる。  
Xuanhui Wang, Michael Bendersky, Donald Metzler, and Marc Najork. Learning to rank with selection bias in personal search. In ACM Conference on Research and Development in Information Retrieval (SIGIR), pp. 115–124, 2016.

#### 評価尺度
オフライン評価にはWeighted Mean Reciprocal Rank(WMRR)を用いる。オンライン評価にはClick-Through Rate(CTR), Average Clicked Position(ACP)を用いる。WMRR, CTRは大きい方が良く、ACPは小さい方が良い尺度である。
それぞれの詳細は論文を参照。

#### オフライン実験
以下の手法を比較する。

- ベースライン：はじめに全ての素性を連結し、3つの全結合層を持つニューラルネットに入力する手法
- Decision Tree
- Convolutional Neural Network (CNN)
- Latent Cross
- Multiplication First
- MCN
- Kernel MCN：クエリと文書の積を取る際に線形射影行列を用いることで埋め込みの次元間の相互作用を可能にする。
<img src="https://user-images.githubusercontent.com/61494140/80078629-26228900-858a-11ea-8bcf-2fda27aba416.png" alt="Kernel MCN" width="40%">

以下に結果を示す。

<img src="https://user-images.githubusercontent.com/61494140/80083037-1d34b600-8590-11ea-9ff3-034e006aa832.png" alt="Result of offline experiment" width="60%">

結果から次のことがわかる。
- Decision Treeは埋め込みの学習なしではうまく機能しない。これは意味的マッチングの強さを示している。
- CNNでは良い結果が得られなかった。評価に使ったデータセットにおいて、一般的なテキスト埋め込みによる利点は限定的である。また学習時間が標準の構造よりも大幅に長くなる。
- Multiplication Firstは良い結果となった。早い段階でのマッチングは単純ながら性能が有意に改善する。逆に連結の方法は学習に非効率なことがわかる。
- Latent Crossは改善が見られなかった。これはクエリを単に潜在的な素性として処理するだけではうまくいかないことを示している。クエリと文書のマッチングを明示的にモデル化する必要がある。
- MCNとKernel MCNは最もうまく動作している。早い段階でのマッチングと、その出力と側面的情報の相互作用の促進に意味があることを示している。
- Kernel MCNとMCNは僅かな差に留まった。積を取るだけのマッチングだけで十分に機能することを示している。

#### オンライン実験
MCNの有効性をさらに確認するために、大規模なオンライン実験を実施する。
オフライン実験ではGoogle Driveにおいて最も大きな改善が見られたので、オンライン実験でもDriveの検索を行う。
2019年3月に一部のユーザに対してA/Bテストを実施し、1000万回の検索セッションから結果を収集した。

結果を以下に示す。
Abandon Rateは、結果をクリックせずに検索セッションを放棄したユーザの割合である。
Long Click per Sessionは、クリックした文書に5秒以上滞在した数である。
Time to Clickは、ユーザが検索セッションを開始してから最初のクリックまでにかかった時間である。

<img src="https://user-images.githubusercontent.com/61494140/80085994-01331380-8594-11ea-8435-e3994a8eab0c.png" alt="Result of online experiment" width="60%">

今の本番環境と比較して、CTRが1.9%、ACPが1.57%改善した。
また他の指標においても良い傾向にあることがわかる。
この結果を受けて、MCNはDrive検索に完全に導入された。



### 関連研究
- An Introduction to Neural Information Retrieval (https://www.microsoft.com/en-us/research/publication/introduction-neural-information-retrieval)
- Alex Beutel, Paul Covington, Sagar Jain, Can Xu, Jia Li, Vince Gatto, and Ed H. Chi. Latent Cross: Making Use of Context in Recurrent Recommender Systems. In Proceedings of the Eleventh ACM International Conference on Web Search and Data Mining (WSDM ’18), pp. 46-54,  2018.
