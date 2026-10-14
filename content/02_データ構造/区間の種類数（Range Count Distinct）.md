#データ構造 #区間クエリ #ウェーブレット行列 #BIT

ライブラリ：[static_range_count_distinct.hpp](https://github.com/winter-2521/cp-library/blob/main/data_structure/static_range_count_distinct.hpp)

## 1. 概要

本記事では、静的な数列 $a$ について「$a_l, \ldots, a_{r-1}$ に何種類の値があるか」に答える方法を解説します。鍵は「**各値は区間の中で一番左の出現だけ数える**」という言い換えで、これにより長方形の中の点の個数の問題になります。最後に、クエリを先読みしなくてよい（オンラインの）$O(\log n)$ の方法を示します。

>[!問題] 問題 1：Static Range Count Distinct（Library Checker）
>長さ $N$ の数列に「$a_l, \ldots, a_{r-1}$ の種類数」を $Q$ 回答えてください。（$N, Q \le 5 \times 10^5$）

## 2. 前提知識

- BIT（Fenwick Tree）：[[Fenwick Tree]]
- ウェーブレット行列（「区間の中で値が $v$ 未満のものの個数」が $O(\log n)$）：[[ウェーブレット行列]]

## 3. 素朴な方法

区間の値をハッシュセットに入れて大きさを見ます。1 回 $O(r - l)$ で、最悪 $2.5 \times 10^{11}$ 回になり間に合いません。

## 4. 一番左の出現だけ数える

区間の中に同じ値が何個あっても、**その値の区間内で一番左の出現だけ** を数えれば、種類数になります。

位置 $i$ が「区間 $[l, r)$ の中で $a_i$ の一番左の出現」かどうかは、次の値を使うと簡単に判定できます。

> $\mathrm{prev}_i$ ＝ $i$ より前で $a_i$ と同じ値が最後に出てくる位置（なければ $-1$）

「区間内で $i$ より左に同じ値がない」⇔「$a_i$ と同じ値の直前の出現が $l$ より前」⇔ $\mathrm{prev}_i < l$ です。よって

$$\text{種類数} = \#\{\, i \in [l, r) : \mathrm{prev}_i < l \,\}$$

### 4.1. 例

$a = [3, 1, 3, 2, 3, 1]$（位置 0〜5）

| 位置 $i$ | 0 | 1 | 2 | 3 | 4 | 5 |
| --- | --- | --- | --- | --- | --- | --- |
| $a_i$ | 3 | 1 | 3 | 2 | 3 | 1 |
| $\mathrm{prev}_i$ | −1 | −1 | 0 | −1 | 2 | 1 |

- **$[1, 5)$**（中身 1, 3, 2, 3）：$\mathrm{prev}_1, \ldots, \mathrm{prev}_4 = -1, 0, -1, 2$ のうち 1 未満は $-1, 0, -1$ の 3 個 → **3 種類**（1, 3, 2）。位置 4 の 3 は $\mathrm{prev}_4 = 2 \ge 1$ なので数えません（区間内の左の位置 2 にも 3 がある）。
- **$[3, 6)$**（中身 2, 3, 1）：$\mathrm{prev}_3, \ldots, \mathrm{prev}_5 = -1, 2, 1$ のうち 3 未満は 3 個 → **3 種類**。位置 4 の 3 は $\mathrm{prev}_4 = 2$ ですが、位置 2 は区間の外（$2 < 3$）なので、区間内では一番左として数えます。

### 4.2. 長方形の中の点の個数になった

点 $(i, \mathrm{prev}_i)$ を平面に置くと、答えは「$l \le i < r$ かつ $\mathrm{prev}_i < l$ の点の個数」、つまり **長方形の中の点の個数** です。ここからは長方形の点の個数を数える定番の方法が使えます。

## 5. 方法 1（オフライン）：右端でソートして BIT

クエリを全部先に読んでよいなら、次の方法があります。

1. クエリを右端 $r$ の小さい順に並べる。
2. 位置 $i = 0, 1, 2, \ldots$ と右に進みながら、BIT に「各値の **最後の出現位置** にだけ 1」を立てていく（$a_i$ が来たら、$\mathrm{prev}_i \ge 0$ なら位置 $\mathrm{prev}_i$ の 1 を消し、位置 $i$ に 1 を立てる）。
3. 右端が $r$ のクエリは、$i = r - 1$ まで進めた時点で BIT の $[l, r)$ の和が答え。

$[0, r)$ の各値は最後の出現にだけ 1 が立っているので、$[l, r)$ の和は「最後の出現が $l$ 以上の値の数」＝区間の種類数です。全体 $O((n + q) \log n)$ です。ただしクエリを先に全部読む必要があります。

## 6. 方法 2（オンライン）：ウェーブレット行列

$\mathrm{prev}$ の列でウェーブレット行列を作っておけば、「区間 $[l, r)$ の中で値が $l$ 未満のものの個数」は `count_lt(l, r, l)` 1 回、$O(\log n)$ で求まります。クエリごとにすぐ答えられます。

### 6.1. $\mathrm{prev}$ の作り方

位置を「値の昇順、同じ値なら位置の昇順」に並べると、同じ値の位置が連続して並びます。隣り合う 2 つが同じ値なら、前の位置が後ろの $\mathrm{prev}$ です。

### 6.2. 実装

```cpp
StaticRangeCountDistinct(const vector<T> &a) : n(a.size()) {
    vector<int> ord(n), prev(n, -1);
    iota(ord.begin(), ord.end(), 0);
    stable_sort(ord.begin(), ord.end(), [&](int i, int j) { return a[i] < a[j]; });
    for (int k = 1; k < n; k++)
        if (!(a[ord[k - 1]] < a[ord[k]])) prev[ord[k]] = ord[k - 1];   // 同じ値が続いた
    wm = WaveletMatrix<int>(prev);
}
int count(int l, int r) const { return l < r ? wm.count_lt(l, r, l) : 0; }
```

構築 $O(n \log n)$、クエリ $O(\log n)$ です。

## 7. 正しさ

区間 $[l, r)$ に出てくる値 $v$ の、区間内の出現位置を $i_1 < i_2 < \cdots$ とします。

- $i_1$：$i_1$ より前の $v$ の出現は全て $l$ より前なので、$\mathrm{prev}_{i_1} < l$。数えます。
- $i_2, i_3, \ldots$：$\mathrm{prev}_{i_k} = i_{k-1} \ge l$。数えません。

よって各値がちょうど 1 回ずつ数えられます。

## 8. 問題

【問題 1】Static Range Count Distinct は `StaticRangeCountDistinct<int>` の `count(l, r)` で解けます。

【問題 2】各値 $v$ に重み $w_v$ があるとき、区間に出てくる値の重みの合計（同じ値は 1 回だけ数える）を答えてください。
（ヒント：5 節の BIT で、1 の代わりに $w_{a_i}$ を立てます。）

### 関連問題

- [Static Range Count Distinct (Library Checker)](https://judge.yosupo.jp/problem/static_range_count_distinct)
- [ABC174 F - Range Set Query](https://atcoder.jp/contests/abc174/tasks/abc174_f)：同じ問題。

## 9. まとめ

- 各値を「区間内で一番左の出現」だけ数えると、$\mathrm{prev}_i < l$ の個数になる。
- これは点 $(i, \mathrm{prev}_i)$ の長方形の点の個数なので、オフラインなら右端でソート + BIT、オンラインならウェーブレット行列で解ける。

## 10. 参考文献

- [Library Checker - Static Range Count Distinct](https://judge.yosupo.jp/problem/static_range_count_distinct)
- [AtCoder アルゴリズム講座 - Wavelet Matrix 基礎](https://info.atcoder.jp/entry/algorithm_lectures/wavelet_matrix_basic)
