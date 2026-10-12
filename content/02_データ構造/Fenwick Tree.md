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

## 具体例で理解する

### 持っているもの

長さ $n$ の配列 $d_1, \ldots, d_n$（1-indexed）だけ。$d_i$ は区間 $(i - \mathrm{lowbit}(i),\ i]$ の和。$\mathrm{lowbit}(i)$ は「$i$ を 2 進数で書いたときの最下位の 1 が表す数」（`i & -i`）。

$n = 8$ の場合：

| $i$ | 2 進 | lowbit | $d_i$ が持つ区間 |
| --- | --- | --- | --- |
| 1 | 001 | 1 | $(0, 1]$ = $a_1$ |
| 2 | 010 | 2 | $(0, 2]$ = $a_1 + a_2$ |
| 3 | 011 | 1 | $(2, 3]$ = $a_3$ |
| 4 | 100 | 4 | $(0, 4]$ = $a_1 + \cdots + a_4$ |
| 5 | 101 | 1 | $(4, 5]$ |
| 6 | 110 | 2 | $(4, 6]$ |
| 7 | 111 | 1 | $(6, 7]$ |
| 8 | 1000 | 8 | $(0, 8]$ |

### セグ木との関係

区間和は「$[0, r)$ の和 − $[0, l)$ の和」なので、**左端が 0 の区間（prefix）の和だけ求まれば十分**。prefix の和だけなら、セグ木の **右の子は要らない**（右の子の和は「親 − 左の子」で分かる）。各ノードから右の子を捨てると残りはちょうど $n$ 個で、それを並べたのが上の表。

### 累積和 $a_1 + \cdots + a_r$ の手順

`s = 0` として、$r > 0$ の間「`s += d[r]`、$r$ から最下位ビットを消す（`r &= r - 1`）」。

**例：$r = 7$**。$7 = 111_2 \to 110_2 = 6 \to 100_2 = 4 \to 0$ なので $d_7 + d_6 + d_4$。表を見ると $(6,7] + (4,6] + (0,4] = (0, 7]$ で、区間がちょうど隙間なくつながっている。足す回数はビットの個数（$\le \log_2 n$）。

### 一点加算 $a_i \mathrel{+}= x$ の手順

$i \le n$ の間「`d[i] += x`、$i$ に最下位ビットを足す（`i += i & -i`）」。

**例：$i = 3$**。$3 \to 4 \to 8$。表を見ると、$a_3$ を含む区間を持つのはちょうど $d_3, d_4, d_8$ の 3 つ。

### lower_bound（累積和が $w$ 以上になる最初の位置）の手順

全要素が非負のとき。位置 $x = 0$ から始め、$k = $（$n$ 以下の最大の 2 冪）, …, 2, 1 の順に「$x + k \le n$ かつ $d_{x+k} < w$ なら、$x \mathrel{+}= k$、$w \mathrel{-}= d_{x}$」。最後の $x + 1$ が答え（1-indexed）。

**例**：$a = [1, 0, 2, 3, 0, 1, 4, 1]$（$a_1 \ldots a_8$）、$w = 5$。累積和は $1, 1, 3, 6, \ldots$ なので答えは 4 番目。

| $k$ | 見るマス | 値 | 判定 | $x$ | 残り $w$ |
| --- | --- | --- | --- | --- | --- |
| 8 | $d_8$ | 12 | $\ge 5$ → 進まない | 0 | 5 |
| 4 | $d_4$ | 6 | $\ge 5$ → 進まない | 0 | 5 |
| 2 | $d_2$ | 1 | $< 5$ → 進む | 2 | 4 |
| 1 | $d_3$ | 2 | $< 4$ → 進む | 3 | 2 |

$x = 3$ まで（$a_1 + a_2 + a_3 = 3$）は 5 未満 → 答えは位置 4。$d_{x+k}$ がちょうど「$x$ の次から $k$ 個の和」になっているので、累積和を毎回計算し直さなくてよい。

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
