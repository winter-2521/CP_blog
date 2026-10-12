#データ構造 #BIT

ライブラリ：[fenwick_tree.hpp](https://github.com/winter-2521/cp-library/blob/main/data_structure/fenwick_tree.hpp)

## 何ができるアルゴリズムか？

数列に対して、一点加算と区間和を $O(\log n)$ で処理する（Binary Indexed Tree, BIT）。要素が非負なら「累積和が $w$ 以上になる最初の位置」も $O(\log n)$ で求まる。

>[!問題] Point Add Range Sum (Library Checker)
>長さ $N$ の数列 $a$ に対して、`0 p x`：$a_p \mathrel{+}= x$、`1 l r`：$\sum_{i=l}^{r-1} a_i$ を $Q$ 回処理せよ。$N, Q \le 5 \times 10^5$

## 計算量

| 操作 | 計算量 |
| --- | --- |
| 構築（配列から） | $O(n)$ |
| 一点加算・累積和・区間和 | $O(\log n)$ |
| `lower_bound(w)` | $O(\log n)$ |

## 証明・仕組み

1-indexed で、$d_i$ に区間 $(i - \mathrm{lowbit}(i), i]$ の和を持つ（$\mathrm{lowbit}(i)$ は $i$ の最下位の 1 のビット、`i & -i`）。

- **累積和** $a_1 + \cdots + a_r$：$r$ から最下位ビットを消していくと $(r - \mathrm{lowbit}(r), r]$, … と区間が隙間なくつながって $(0, r]$ を覆う。ビットの個数ぶんなので $O(\log n)$。
- **一点加算** $a_i \mathrel{+}= x$：$i$ を含む区間を持つのは $i, i + \mathrm{lowbit}(i), \ldots$（最下位ビットを足していく）。これも $O(\log n)$ 個。
- **$O(n)$ 構築**：$d_i$ に $a_i$ を入れ、$i$ の小さい順に $d_{i + \mathrm{lowbit}(i)} \mathrel{+}= d_i$ とすると、各 $d$ が子の区間の和を受け取って正しくなる。

### 区間和を少し速く

$\mathrm{sum}(r) - \mathrm{sum}(l)$ を 2 回計算する代わりに、$l$ と $r$ から交互に最下位ビットを消していき、値が一致したら（共通部分なので）打ち切る。短い区間ほど早く終わる。

### lower_bound（BIT 上の二分探索）

全要素が非負なら累積和は単調。上のビットから順に「$x + 2^k$ まで進んでも累積和が $w$ 未満か」を見て進む。$d_{x+2^k}$ がちょうど $(x, x + 2^k]$ の和なので、各ビット $O(1)$ で全体 $O(\log n)$（素朴な二分探索 + 累積和だと $O(\log^2 n)$）。

## 例題・類題

- [Point Add Range Sum (Library Checker)](https://judge.yosupo.jp/problem/point_add_range_sum)
- [AtCoder Library Practice Contest B - Fenwick Tree](https://atcoder.jp/contests/practice2/tasks/practice2_b)
- **転倒数**：左から順に「自分より大きい値がこれまでにいくつあったか」を BIT で数える。[ABC190 F - Shift and Inversions](https://atcoder.jp/contests/abc190/tasks/abc190_f)
- [ABC256 F - Cumulative Cumulative Cumulative Sum](https://atcoder.jp/contests/abc256/tasks/abc256_f)：累積和の累積和を、$i^2 a_i, i a_i, a_i$ の 3 本の BIT で表す。
- 座標圧縮した値の個数を持てば「$k$ 番目に小さい値」が `lower_bound` で求まる（[[順序付き集合（ブロックAVL木）]] のオフライン版）。
