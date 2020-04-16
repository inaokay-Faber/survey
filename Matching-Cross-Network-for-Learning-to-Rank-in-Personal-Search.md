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
際立った2つの素性とそうでない素性を含んだ合成データセットを使って、従来手法の限界を実証する。

各要素が-1～+1の一様分布で初期化された20次元のベクトルを3つセットにしたものを1サンプルとし、1,000サンプルずつ訓練セット、検証セット、テストセットとする。



### 関連研究
- An Introduction to Neural Information Retrieval (https://www.microsoft.com/en-us/research/publication/introduction-neural-information-retrieval)
