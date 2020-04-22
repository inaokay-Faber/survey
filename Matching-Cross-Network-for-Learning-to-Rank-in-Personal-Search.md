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
結果から、単純な連結とLatent Crossはどちらもコサイン類似度の学習効率が悪いことがわかる。

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

#### Matching Cross Network (MCN)
MCNの構造を以下に示す。

<img src="https://user-images.githubusercontent.com/61494140/79733549-46570b80-8330-11ea-945d-9ed0868120a3.png" alt="MCN architecture" width="60%">

### 関連研究
- An Introduction to Neural Information Retrieval (https://www.microsoft.com/en-us/research/publication/introduction-neural-information-retrieval)
- Alex Beutel, Paul Covington, Sagar Jain, Can Xu, Jia Li, Vince Gatto, and Ed H. Chi. Latent Cross: Making Use of Context in Recurrent Recommender Systems. In Proceedings of the Eleventh ACM International Conference on Web Search and Data Mining (WSDM ’18), 2018.
