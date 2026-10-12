#データ構造 #UnionFind

ライブラリ：[potentialized_unionfind.hpp](https://github.com/winter-2521/cp-library/blob/main/data_structure/potentialized_unionfind.hpp)

## 何ができるアルゴリズムか？

各要素 $u$ に未知の値 $a_u$ があり、「$a_u - a_v = w$」という情報を追加していく。

- 新しい情報がこれまでと矛盾するかの判定
- 同じ連結成分の 2 要素の差 $a_u - a_v$ の取得

をならし $O(\alpha(n))$ で処理する（重み付き Union-Find）。$a_u$ は加法群に限らず **任意の群**（xor、置換、可逆行列など非可換でもよい）にできる。

>[!問題] Unionfind with Potential (Library Checker)
>`0 u v x`：$a_u \equiv a_v + x$ という情報。矛盾しなければ採用して 1、矛盾すれば 0 を出力
>`1 u v`：$a_u - a_v$ が定まるなら出力、定まらないなら $-1$

非可換版は「$a_u = a_v x$（$2 \times 2$ 行列）」で、$a_v^{-1} a_u$ を答える。

## 計算量

各操作ならし $O(\alpha(n))$（群演算を $O(1)$ として）。

## 証明・仕組み

普通の Union-Find に加えて、各要素 $x$ に $p_x = a_{\mathrm{par}(x)}^{-1} a_x$（親から見た相対値）を持つ。根では単位元。

- **根からの相対値**：$x$ から根 $r$ までの経路の $p$ を、根側から順に掛けると $a_r^{-1} a_x$ になる（$a_r^{-1} a_{p_1} \cdot a_{p_1}^{-1} a_{p_2} \cdots$ が打ち消し合う）。経路圧縮で $x$ を根に直接つなぎ替えるとき、$p_x$ をこの積に更新しておけば、以後 $p_x = a_r^{-1} a_x$。
- **差の取得**：同じ根 $r$ なら $a_v^{-1} a_u = (a_r^{-1} a_v)^{-1} (a_r^{-1} a_u)$。
- **併合**（$a_u = a_v w$）：根 $r_u, r_v$ が違えば、$a_u = a_{r_u} P_u$, $a_v = a_{r_v} P_v$（$P$ は根からの相対値）より
  $$a_{r_v}^{-1} a_{r_u} = P_v \, w \, P_u^{-1}$$
  これを $r_u$ を $r_v$ の子にするときの $p_{r_u}$ にする（逆向きにつなぐなら逆元）。根同士がつながっていなかったので、この 1 本の制約で矛盾は起きない。
- **矛盾判定**：根が同じなら $a_v^{-1} a_u$ が既に決まっているので、$w$ と比べる。

**非可換で注意すること**：積の順序。経路圧縮で掛ける順は「根側が左」、差は「$v$ 側の逆元が左」。加法群のつもりで書くと非可換のときに壊れる。

union by size と経路圧縮を併用するのでならし $O(\alpha(n))$ で、再帰の深さも $O(\log n)$。

## 例題・類題

- [Unionfind with Potential (Library Checker)](https://judge.yosupo.jp/problem/unionfind_with_potential)
- [Unionfind with Potential (Non-Commutative Group) (Library Checker)](https://judge.yosupo.jp/problem/unionfind_with_potential_non_commutative_group)
- [AOJ DSL_1_B - Weighted Union Find Trees](https://onlinejudge.u-aizu.ac.jp/courses/library/3/DSL/1/DSL_1_B)
- [ABC087 D - People on a Line](https://atcoder.jp/contests/abc087/tasks/arc090_b)：「$x_r - x_l = d$」が全て両立するか。
- [ABC328 F - Good Set Query](https://atcoder.jp/contests/abc328/tasks/abc328_f)：矛盾しない制約だけを採用していく。まさに `merge` の戻り値。
- **xor の群**：「$a_u \oplus a_v = w$」（偶奇・2 色塗り分けの制約）。二部グラフ判定の一般化。
