#データ構造 #区間クエリ #二分探索

ライブラリ：[static_range_frequency.hpp](https://github.com/winter-2521/cp-library/blob/main/data_structure/static_range_frequency.hpp)、[dynamic_range_frequency.hpp](https://github.com/winter-2521/cp-library/blob/main/data_structure/dynamic_range_frequency.hpp)

## 1. 概要

本記事では、数列 $a$ について「$a_l, a_{l+1}, \ldots, a_{r-1}$ の中に値 $x$ が何個あるか」に $O(\log n)$ で答える方法を解説します。値の書き換え（一点更新）がある場合も扱います。

>[!問題] 問題 1：Static Range Frequency（Library Checker）
>長さ $N$ の数列に「$a_l, \ldots, a_{r-1}$ の中の $x$ の個数」を $Q$ 回答えてください。（$N, Q \le 5 \times 10^5$、$a_i, x \le 10^9$）

>[!問題] 問題 2：Point Set Range Frequency（Library Checker）
>問題 1 に「$a_k \gets v$」の更新が加わります。（$N, Q \le 2 \times 10^5$）

## 2. 前提知識

- 二分探索（`lower_bound`：ソート済みの列で「$v$ 以上の最初の場所」を求める）
- 順序付き集合（`std::set` のように要素を順序付きで持ち、「$v$ 未満の要素の個数」も求められるもの）：[[順序付き集合（ブロックAVL木）]]

## 3. 素朴な方法

### 3.1. 区間を全部見る

区間を左から右まで見て、$x$ と等しいものを数えます。1 回 $O(r - l)$ なので、$N, Q = 5 \times 10^5$ では最悪 $2.5 \times 10^{11}$ 回で間に合いません。

### 3.2. 値ごとの累積和

値 $x$ ごとに累積和 $c_x[i]$ =（先頭 $i$ 個の中の $x$ の個数）を作れば、答えは $c_x[r] - c_x[l]$ で $O(1)$ です。しかし表の大きさが（値の種類数）× $n$ になり、種類が $5 \times 10^5$ 個あると $2.5 \times 10^{11}$ マスで、メモリが足りません。

## 4. 値ごとに「出てくる位置のリスト」を作る

累積和の表は、ほとんどのマスが「前のマスと同じ値」で無駄です。値が変わるのは「$x$ が出てくる位置」だけなので、**値 $x$ が出てくる位置を小さい順に並べたリスト** を持てば十分です。

$[l, r)$ の中の $x$ の個数は、「$x$ の位置のリストの中で、$l$ 以上 $r$ 未満のものの個数」です。リストはソートされているので、

$$(\text{$r$ 以上の最初の場所}) - (\text{$l$ 以上の最初の場所})$$

の **二分探索 2 回** で求まります。リストの長さの合計は $n$ なので、メモリも $O(n)$ です。

### 4.1. 例

$a = [3, 1, 3, 2, 3, 1]$（位置 0〜5）のとき、

| 値 | 出てくる位置のリスト |
| --- | --- |
| 1 | $[1, 5]$ |
| 2 | $[3]$ |
| 3 | $[0, 2, 4]$ |

**count(1, 5, 3)**（$a_1, \ldots, a_4 = 1, 3, 2, 3$ の中の 3 の個数）：3 のリスト $[0, 2, 4]$ で、

- 5 以上の最初の場所：リストの 3 番目（末尾の外）
- 1 以上の最初の場所：リストの 1 番目（値 2）

なので $3 - 1 = $ **2** です。

### 4.2. 持っているもの（ライブラリの形）

リストを値ごとに別々の `vector` にすると遅いので、1 本の配列にまとめます。

| 名前 | 意味 |
| --- | --- |
| `vals` | 出てくる値の種類（昇順） |
| `pos` | 位置を「値の昇順、同じ値なら位置の昇順」に並べたもの |
| `start` | 値 `vals[k]` のリストは `pos[start[k]]` 〜 `pos[start[k+1] - 1]` |

例の場合（ライブラリで実際に出力した値）：

```
vals  = [1, 2, 3]
pos   = [1, 5 | 3 | 0, 2, 4]     ← 1 のリスト / 2 のリスト / 3 のリスト
start = [0, 2, 3, 6]
```

`pos` は、位置 $0, \ldots, n-1$ を「値で安定ソート」すれば作れます。

### 4.3. 実装

```cpp
int count(int l, int r, const T &x) const {
    int k = lower_bound(vals.begin(), vals.end(), x) - vals.begin();   // x の番号
    if (k == (int)vals.size() || x < vals[k]) return 0;                // x は出てこない
    auto b = pos.begin() + start[k], e = pos.begin() + start[k + 1];   // x のリスト
    return lower_bound(b, e, r) - lower_bound(b, e, l);
}
```

構築 $O(n \log n)$、クエリ $O(\log n)$ です。

## 5. 書き換えがある場合

### 5.1. 遅い方法

4 節の配列 `pos` のままだと、$a_k$ を書き換えるたびに、古い値のリストから $k$ を消し、新しい値のリストに $k$ を入れる必要があり、配列の挿入・削除で $O(n)$ かかります。

### 5.2. (値, 位置) の組を順序付き集合に入れる

「(値, 位置) の組」を全部まとめて **順序付き集合** に入れます。組は「値で比べ、同じなら位置で比べる」（辞書式順序）ので、集合の中では 4.2 の `pos` と同じ順に並びます。

```
(1,1) (1,5) (2,3) (3,0) (3,2) (3,4)
```

- **count(l, r, x)**：$(x, l)$ 以上 $(x, r)$ 未満の組の個数 ＝「$(x, r)$ 未満の組の個数」−「$(x, l)$ 未満の組の個数」。順序付き集合は「ある値未満の要素の個数」を $O(\log n)$ で数えられます。
- **set(i, v)**：組 $(a_i, i)$ を削除し、$(v, i)$ を挿入します。

**例**：set(1, 3) で $a = [3, 3, 3, 2, 3, 1]$ にすると、$(1, 1)$ を消して $(3, 1)$ を入れます。

```
(1,5) (2,3) (3,0) (3,1) (3,2) (3,4)
```

count(0, 3, 3) は「$(3, 3)$ 未満」の 5 個 −「$(3, 0)$ 未満」の 2 個 = **3** です（$a_0, a_1, a_2$ はすべて 3）。

### 5.3. 実装

```cpp
void set(int i, const T &x) {
    st.erase({a[i], i});
    a[i] = x;
    st.insert({a[i], i});
}
int count(int l, int r, const T &x) const { return l < r ? st.count_range({x, l}, {x, r}) : 0; }
```

更新・クエリともに $O(\log n)$ です。

## 6. 正しさ

位置のリストはソートされているので、「$l$ 以上の最初の場所」から「$r$ 以上の最初の場所」の手前までが、ちょうど位置が $[l, r)$ に入る $x$ の出現です。順序付き集合の版も同じで、$(x, l) \le (y, i) < (x, r)$ となる組は「$y = x$ かつ $l \le i < r$」とちょうど一致します（辞書式順序）。

## 7. 問題

【問題 1】Static Range Frequency は `StaticRangeFrequency<int>` の `count(l, r, x)` で、【問題 2】Point Set Range Frequency は `DynamicRangeFrequency<int>` の `set(k, v)` と `count(l, r, x)` で解けます。

【問題 3】長さ $N$ の数列について、「区間 $[l, r)$ に $x$ が $k$ 回以上出てくるか」を $O(1)$ で答えてください（前計算 $O(n \log n)$）。
（ヒント：$x$ のリストの中で、$l$ 以上の最初の場所を $p$ とすると、リストの $p + k - 1$ 番目が $r$ 未満かどうかで分かります。各位置 $i$ について「$i$ が自分の値のリストの何番目か」を前計算しておけば、$l$ の位置に $x$ があるときは二分探索も要りません。この考え方は [[区間最頻値（平方分割）]] で使います。）

### 関連問題

- [Static Range Frequency (Library Checker)](https://judge.yosupo.jp/problem/static_range_frequency)
- [Point Set Range Frequency (Library Checker)](https://judge.yosupo.jp/problem/point_set_range_frequency)

## 8. まとめ

- 値ごとの累積和は、値が変わる「出現位置」だけを持てば十分。区間の個数は出現位置のリストの二分探索 2 回。
- 書き換えがあるときは、(値, 位置) の組を順序付き集合に入れ、「$(x, r)$ 未満」−「$(x, l)$ 未満」を数える。

## 9. 参考文献

- [Library Checker - Static Range Frequency](https://judge.yosupo.jp/problem/static_range_frequency)
- [Library Checker - Point Set Range Frequency](https://judge.yosupo.jp/problem/point_set_range_frequency)
