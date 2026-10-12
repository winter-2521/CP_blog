#データ構造 #セグ木 #ならし計算量

ライブラリ：[segment_tree_beats.hpp](https://github.com/winter-2521/cp-library/blob/main/data_structure/segment_tree_beats.hpp)

## 何ができるアルゴリズムか？

区間 chmin（$a_i \gets \min(a_i, x)$）・区間 chmax・区間加算と、区間和・区間 min・区間 max を、ならし $O(\log^2 n)$ で処理する。区間 chmin と区間和は普通の遅延セグ木には乗らない。

>[!問題] Range Chmin Chmax Add Range Sum (Library Checker)
>区間 chmin / 区間 chmax / 区間加算 / 区間和 を $Q$ 回処理せよ。$N, Q \le 2 \times 10^5$、$|a_i| \le 10^{12}$

## 計算量

ならし $O((n + q) \log^2 n)$。加算がなければ $O((n + q) \log n)$。

## 証明・仕組み

### なぜ遅延セグ木に乗らないか

区間和だけ持っていると、chmin($x$) で和がいくつ減るかが分からない（$x$ より大きい要素がいくつあり、いくつ大きいかは和からは分からない）。

### 最大値と 2 番目の最大値を持つ

各ノードに **最大値 $m_1$、2 番目に大きい値 $m_2$（狭義）、最大値の個数 $c$**、和を持つ。chmin($x$) をノードに適用するとき：

- $m_1 \le x$：何も変わらない。終了。
- $m_2 < x < m_1$：変わるのは最大値の要素だけで、それらが全て $x$ になる。和は $(m_1 - x) \cdot c$ 減り、$m_1 = x$。**このノードで止められる**（子にはタグとして「最大値は $x$ 以下」を後で伝える）。
- $x \le m_2$：子に降りて再帰する。

chmax も最小値側を同様に持てば対称。

### ならし計算量（加算なし）

ポテンシャル $\Phi$ = 「各ノードの部分木に含まれる値の種類数」の総和、とする。初期値は $O(n \log n)$。

3 つ目の場合（$x \le m_2$ で降りる）、そのノードでは $m_1$ と $m_2$ が両方 $x$ になるので種類数が 1 以上減る。つまり「余計に降りたノード」ごとに $\Phi$ が 1 以上減る。区間の境界の経路（普通のセグ木と同じ $O(\log n)$ 個）以外で降りるノードは全てこの場合なので、全体で $O((n + q) \log n)$。

加算があると境界の経路上で種類数が増えうる（1 回で $O(\log n)$ ノード × 各 $O(\log n)$）ので、全体 $O((n + q) \log^2 n)$ になる（Jiry 氏の解析）。

### タグの伝え方

push では、子の最大値が親の最大値より大きければ「子の最大値を親の最大値にする」（子では必ず 2 つ目の場合になる）。加算は通常の遅延評価。最大値と最小値が同じ要素（ノードの値が 1 種類・2 種類のとき）は、最大値側を変えたら最小値側も更新するなど、細かい場合分けが要る。

## 例題・類題

- [Range Chmin Chmax Add Range Sum (Library Checker)](https://judge.yosupo.jp/problem/range_chmin_chmax_add_range_sum)
- [HDU 5306 - Gorgeous Sequence](https://acm.hdu.edu.cn/showproblem.php?pid=5306)：区間 chmin・区間 max・区間和。Segment Tree Beats の元ネタ。
- [ABC256 Ex - I like Query Problem](https://atcoder.jp/contests/abc256/tasks/abc256_h)：区間を $x$ で割る（切り捨て）・区間代入・区間和。割り算は「区間の値が全て同じなら代入にできる」で止め、値の種類数のポテンシャルで同じように抑えられる。
- 「区間の各要素に、値によって効き方が変わる操作（割る・mod をとる・平方根）」を、区間内の値がそろっていれば一括で処理し、そろっていなければ降りる、という発想は Beats 全般に共通。
