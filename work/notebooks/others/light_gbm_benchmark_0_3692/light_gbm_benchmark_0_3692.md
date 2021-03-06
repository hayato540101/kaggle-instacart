https://www.kaggle.com/paulantoine/light-gbm-benchmark-0-3692/code

### 特徴

- submitを

```注文番号 注文した商品番号の文字列の足し合わせ
2774568	17668 21903 39190 47766 18599 43961 23650 24810
1528013	21903 38293
```
の順で予測する必要がある

各商品の再購入確率を出し、order_idごとにそれが閾値を超えた場合にその商品番号を取得して、
それらを文字列結合したのがsubmitファイルになる

Fscoreらしいが、どうやって計算するのか？
→正解か間違いかをkaggle側で0,1として持っておいて、あっちの内部処理でスコアを出してそう
なので出せそう、処理はブラックボックスのままだと思う

#### メモ
- 同一注文IDで同一商品を複数個購入した例が一つもないらしい

dataframe.iterrows()とdataframe.itertuples()の違い

### 特徴量
ユーザベース
- usr['average_days_between_orders'] 注文間隔の平均
- usr['nb_orders'] 注文数

- users['total_items'] 重複ありでの総購入商品数
- users['all_products'] user_idごとの商品IDの集合=注文したことがある全商品 
- users['total_distinct_items'] users['all_products']の数(長さ)
- users['average_basket'] 1回当たり購入商品数
- df['days_since_ratio'] = df.days_since_prior_order / df.user_average_days_between_orders
    - 前回の注文からの日数と平均の注文感覚の比率

ユーザ×商品ベース？
- df['UP_average_pos_in_cart'] = (df.z.map(userXproduct.sum_pos_in_cart) / df.UP_orders).astype(np.float32)
    - 平均購入個数

商品IDベース
- 注文回数
- そのうちの再注文回数
- 注文回数/再注文回数




#### 個人的な新しい知見

- int 16で3万、32で21億まで
```
int8	-128 ～ 127

int16	-32,768 ～ 32,767
int32	-2,147,483,648 ～ 2,147,483,647
int64	-9,223,372,036,854,775,808 ～ 9,223,372,036,854,775,807
uint8	0 ～ 255
uint16	0 ～ 65,535
uint32	0 ～ 4,294,967,295
uint64	0 ～ 18,446,744,073,709, 551,615
```

- dict→dataframeでメモリ使用量が1/3に
単位はbitだろうか

```
a = d.__sizeof__()
d = pd.DataFrame.from_dict(d, orient='index')
b = d.__sizeof__()
a,b
(671088720, 212697024)
```
- forのイテレータに対し
```
order_id = row.order_id
user_id = row.user_id```

のように各要素を変数に格納すると可読性が増す
```
- その他文字起しに適さない事柄多数

#### コードリーディング
- 目的がわかる場合
- 関数
    - 返り値の個数や型を確認しておくと、目的がわかりやすいかも

- 手掛かりになる場合
- 複雑なオブジェクトの各要素の大小がどう変化するか

- スクショは原始的だが非常に役に立つ

#### 感想
- もう少し早くできるはず。itertupleとかとってforで回す操作が多すぎたので、かなり待ち時間が発生してしまった。
- これを解決しようと並列化ライブラリを探したが、細かい仕様の問題で複数ある中で今の自分に使えそうなライブラリはなかった。

- 全体的にきれいなコードで上手い実装だと思った。itertupleとかlistを繰り返すところはなかった発想。