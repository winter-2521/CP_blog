#データ構造 #区間クエリ

ライブラリ：[cumulative_sum.hpp](https://github.com/winter-2521/cp-library/blob/main/data_structure/cumulative_sum.hpp) / [sparse_table.hpp](https://github.com/winter-2521/cp-library/blob/main/data_structure/sparse_table.hpp) / [static_rmq.hpp](https://github.com/winter-2521/cp-library/blob/main/data_structure/static_rmq.hpp)

## 何ができるアルゴリズムか？

**配列が変わらない** とき、区間クエリを前計算して $O(1)$ で答える。

| 手法 | 前計算 | クエリ | 使える演算 |
| --- | --- | --- | --- |
| 累積和 | $O(n)$ | $O(1)$ | 逆元のある演算（和、xor など） |
| Sparse Table | $O(n \log n)$ | $O(1)$ | 冪等な演算（min, max, gcd, and, or） |
| 線形 RMQ（ブロック + ビットマスク） | $O(n)$ | $O(1)$ | min / max（とその位置） |

>[!問題] Static RMQ (Library Checker)
>長さ $N$ の数列に $Q$ 個のクエリ：$\min(a_l, \ldots, a_{r-1})$ を出力せよ。$N, Q \le 5 \times 10^5$

## 証明・仕組み

### 累積和

$s_i = a_0 + \cdots + a_{i-1}$ とすると $\sum_{i=l}^{r-1} a_i = s_r - s_l$。引き算（逆元）が要るので min などには使えない。
2 次元なら $s_{i,j}$ = 左上 $[0,i) \times [0,j)$ の和として、包除原理で長方形の和を 4 項で求める。

### Sparse Table

$t_{k,i} = \mathrm{op}(a_i, \ldots, a_{i + 2^k - 1})$ を全ての $k, i$ について持つ（$t_{k,i} = \mathrm{op}(t_{k-1,i}, t_{k-1,i+2^{k-1}})$ で $O(n \log n)$）。

区間 $[l, r)$ は $2^k \le r - l$ となる最大の $k$ について、$[l, l+2^k)$ と $[r - 2^k, r)$ の **2 つの区間で覆える**（重なりがある）。演算が冪等（$x \cdot x = x$）なら重なった部分を 2 回数えても答えは変わらないので、$\mathrm{op}(t_{k,l}, t_{k, r-2^k})$ が答え。$k$ は `__builtin_clz` で $O(1)$。

和は冪等でないので使えない（重なりを 2 回足してしまう）。

### 線形 RMQ

Sparse Table はメモリが $O(n \log n)$ でキャッシュに乗りにくい。そこで 64 要素ずつのブロックに分ける。

- **ブロック間**：各ブロックの最小値の位置だけで Sparse Table を作る。要素数 $n/64$ なので $O(\frac{n}{64} \log n) = O(n)$。
- **ブロック内**：ブロック先頭から $i$ まで左から順に見たときの **単調スタック**（最小値の候補：右にある自分以下の値がまだ来ていないもの）を、ブロック内の位置のビットマスク $m_i$（64 bit）で持つ。

ブロック内の区間 $[l, i]$ の最小値の位置は、$m_i$ のうち $l$ 以上のビットで **最も下のもの**。理由：スタックに残っている要素は値が左から単調増加していて、$l$ 以上で最も左に残っている要素は、$[l, i]$ の中で自分より右に自分より小さい値がない＝区間の最小値。よって `ctz(m_i >> l)` で $O(1)$。

クエリは「左端のブロックの端数 + 間のブロック (Sparse Table) + 右端のブロックの端数」の 3 つの min。全部 $O(1)$。手元では Sparse Table とほぼ同じ速さで、メモリは $1/10$ 程度。

## 例題・類題

- [Static Range Sum (Library Checker)](https://judge.yosupo.jp/problem/static_range_sum)
- [Static RMQ (Library Checker)](https://judge.yosupo.jp/problem/staticrmq)
- [AOJ DSL_3_D - Sliding Minimum Element](https://onlinejudge.u-aizu.ac.jp/courses/library/3/DSL/3/DSL_3_D)（スライド最小値。deque でも解ける）
- **LCA**：オイラーツアーで頂点の深さを並べると、2 頂点の LCA は「その間で深さが最小の頂点」になる。RMQ で $O(1)$。
- 区間 gcd が何度も必要な問題（Sparse Table）。
