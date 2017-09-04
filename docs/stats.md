# Python stats

## StatsModels
`endog`は目的変数、`exog`は説明変数を意味する。

### Linear Regression
線形回帰モデルを利用できる。

+ OLS: ordinary least squares 普通の自乗誤差
+ WLS: weighted least squares
+ GLS: generalized least squares

`sm.OLS(y,X).fit()`で自乗誤差を最小化する`slope`と`bias`を計算する。
計算結果を知りたい場合は`sm.OLS(y,X).fit().summary()`を出力する。

デフォルトの設定では`OLS`は`intercept`を考慮しない。ので、
データに`statsmodels.tools.add_constant`して`hasconst`を`True`にする必要がある。

### Generalized Linear Models
一般化線形モデルを利用できる。
母数が1つの分布族の推定に対応している。具体的には次のような目的変数の分布を仮定できる。

これらはだいたい`sm.families.Gamma()`などして指定することができる。

+ Binomial (二項分布)
+ Poisson (ポアソン分布)
+ Normal (正規分布)
+ Gamma (ガンマ分布)

### Example
線形モデルの場合とだいたい同じで、`目的変数`, `説明変数`, `仮定する分布`を渡せば良い。

```Python
import statsmodels.api as sm
data = sm.datasets.scotland.load()
data.exog = sm.add_constant(data.exog)
gamma_model = sm.GLM(data.endog, data.exog, family=sm.families.Gamma())
gamma_results = gamma_model.fit()
```

## Pandas tips
[Pandas GroupBy](https://pandas.pydata.org/pandas-docs/stable/groupby.html)

### Group By: split-apply-combine
+ Splitting
+ Applying
    + Aggregation
        + グループごとの要約統計量の集計
    + Transformation
        + グループ内での正規化 !!
        + グループごとの統計量での欠損値の補完 !!
    + Filtration
        + 行数の少ないグループを書落とす
        + グループの要約統計量に基づいてフィルタリング
+ Combining

#### Splitting
行あるいは列を基準として分割を実行できる。
分割したあとのオブジェクトは`GroupBy`であり、`DataFrame`ではなくなる。

```Python
# default axis = 0, 行方向
# 0は中線がヨコ, 1はタテに伸びている 覚える！
grouped = df.groupby(key)
grouped = df.groupby(key, axis=1)
grouped = df.groupby([key1, key2])
```

分割して作成した`GroupBy`オブジェクトから、
特定のグループを取得したい場合は`get_group(key)`メソッドを利用する。

```Python
# 集計に利用したkeyという属性の値がAであるものを取得する
grouped.get_group("A")
```

### GroupBy with MultiIndex
インデックスが階層化されたGroupByオブジェクトでは、指定した階層で集計を行いたい場合があるはず。

```Python
# 階層化されたインデックスを持つオブジェクトを作成
array = [["a","b","c","c","d"], ["x", "x", "x", "y", "z"]]
index = pd.MultiIndex.from_array(array, names=["first", "second"])
s = pd.Series(np.random.randn(4)), index=index)

grouped = s.groupby(level=0) # first で集計を行う
grouped.sum() # グループごとの合計

s.groupby(level="second").sum() # second で集計を行う
```

### Aggregation
GroupByオブジェクトには、グループデータを集計するための様々なメソッドが提供されている。
基本的には、`GroupBy`オブジェクトの`aggregate(func)`を呼び出すことで集計できる。
ちなみに、`aggregate()`は`agg()`でも呼び出すことができる。

```Python
grouped = df.groupby("A")
grouped.aggregate(np.sum) # <= 関数として渡すのがポイント
grouped.size() # それぞれのグループのデータ数
grouped.describe() # グループの基本統計量
```

### DataFrameのマージ
`pd.merge(df1, df2, on="col")`で実行できる。
df1とdf2で列名が異なる場合は、`pd.merge(df1,df2,left_on="col1",right_on="col2")`とする。

また、複数の列でマージを行いたい場合は`pd.merge(df1,df2,on=["col1","col2"])`のようにリストを利用する。

### DataFrame.iloc & DataFrame.loc
データフレームのサブセットを取得したい場合に便利なプロパティの紹介。

`.iloc`は*明示的に行or列の位置番号を指定して取得したい*ときに利用する。
その一方で、`.loc`は*ラベルやインデックスの名前を指定して取得したい*ときに利用する。

いずれのプロパティを利用しても、返されるのは`DataFrame`となる。
*iは数字、無印はラベル*と覚える。
