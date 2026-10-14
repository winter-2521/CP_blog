#データ構造 #永続 #UnionFind

ライブラリ：[persistent_array.hpp](https://github.com/winter-2521/cp-library/blob/main/data_structure/persistent_array.hpp)、[persistent_unionfind.hpp](https://github.com/winter-2521/cp-library/blob/main/data_structure/persistent_unionfind.hpp)

## 1. 概要

本記事では、**永続配列** と、それを使った **永続 Union-Find** を解説します。

- 永続配列：どの過去の版からでも一点更新でき、更新すると新しい版を作る。過去の版はそのまま残る。
- 永続 Union-Find：どの過去の版にも辺を足して新しい版を作れ、どの版でも連結判定ができる。

>[!問題] 問題 1：Persistent UnionFind（Library Checker）
>$G_{-1}$ は辺のない $N$ 頂点のグラフです。$i = 0, 1, \ldots, Q-1$ について次を処理してください。（$N, Q \le 2 \times 10^5$）
>- `0 k u v`：$G_k$ に辺 $(u, v)$ を足したものを $G_i$ とする
>- `1 k u v`：$G_k$ で $u, v$ が連結なら 1、そうでなければ 0 を出力する

## 2. 前提知識

- Union-Find（union by size、経路圧縮）：[[ポテンシャル付きUnion-Find]] の前半
- 永続データ構造の考え方（変わった部分だけ作り、残りは共有する）：[[永続キュー]]

## 3. 素朴な方法

版を作るたびに `par` 配列全体をコピーすると、1 回 $O(N)$、全体 $O(NQ) = 4 \times 10^{10}$ で無理です。

## 4. 永続配列

### 4.1. 配列を木で持つ

配列を **8 分木** で持ちます（葉が配列の要素）。長さ $n$ の配列なら、高さは $\lceil \log_8 n \rceil$ です。

- **取得**：根から葉まで降りるだけです（各段で、添字の 3 ビットずつを見て子を選ぶ）。
- **更新**：根から葉までの道のノードだけを **コピーして作り直し**、それ以外の子は前の版と共有します（path copying）。新しい版の根を返します。

例として、長さ 8 の配列を 2 分木（分かりやすさのため）で持ち、版 0 の `a[5]` を 9 に更新すると、次のようになります。

```
版0の根                      版1の根（新）
 ├ [0,4) ────────共有───────┤
 └ [4,8)                     └ [4,8)（新）
    ├ [4,6)                     ├ [4,6)（新）
    │  ├ a[4] ──────共有───────┤  ├ a[4]
    │  └ a[5]=7                 │  └ a[5]=9（新）
    └ [6,8) ────────共有────────└ [6,8)
```

新しく作ったのは道の上の 4 ノードだけで、残りは版 0 と共有しています。版 0 の根から降りれば、今まで通り `a[5] = 7` が読めます。

### 4.2. 分岐数の選び方

$2^{\mathrm{LOG}}$ 分木にすると、高さは $\log_{2^{\mathrm{LOG}}} n$ ですが、更新で作るノードは子を $2^{\mathrm{LOG}}$ 個持つので、メモリは $2^{\mathrm{LOG}} \times$ 高さ になります。

| 分岐数 | 高さ（$n = 2 \times 10^5$） | 更新 1 回のメモリ（int の個数） |
| --- | --- | --- |
| 2 | 18 | 36 |
| 8 | 6 | 48 |
| 16 | 5 | 80 |

ライブラリでは、取得が速く（高さ 6）メモリもそれほど増えない 8 分木をデフォルトにしています。

### 4.3. 実装

```cpp
T get(int v, int i) const {
    int t = roots[v];
    for (int h = height - 1; h >= 0; h--) t = ch[t][(i >> (LOG * h)) & (K - 1)];  // 3 ビットずつ
    return vals[t];
}

int set_rec(int t, int h, int i, int leaf) {
    array<int, K> c = ch[t];                         // ノードをコピー
    int k = (i >> (LOG * h)) & (K - 1);
    c[k] = h == 0 ? leaf : set_rec(c[k], h - 1, i, leaf);   // 道の上の子だけ差し替える
    ch.push_back(c);
    return ch.size() - 1;
}
```

## 5. 永続 Union-Find

### 5.1. 方針

Union-Find の親の配列 `par` を永続配列で持ちます（`par[x] < 0` なら $x$ は根で、$-$`par[x]` が連結成分の大きさ）。

- **経路圧縮は使いません**。find のたびに `par` を書き換えると、そのたびに新しい版ができてしまいます。また、経路圧縮の計算量は「ならし」で、過去の版に何度も戻って同じ長い道をたどると成り立ちません。
- **union by size だけ使います**。小さい方の根を大きい方の根の子にします。これだけで木の高さは $O(\log n)$ に保たれます。

### 5.2. 操作

- **leader(v, x)**：版 $v$ で、`par` を根までたどる。高さ $O(\log n)$ × 永続配列の取得 $O(\log_8 n)$。
- **merge(v, x, y)**：版 $v$ で $x, y$ の根を求め、違えば、大きい方の根の `par`（大きさ）を更新した版を作り、さらに小さい方の根の `par` を大きい方の根にした版を作る（永続配列の更新 2 回）。

### 5.3. 例

4 頂点で、版 0 から辺 $(0, 1)$ を足した版 $v_1$、$v_1$ に $(2, 3)$ を足した版 $v_2$、**$v_1$ に** $(1, 2)$ を足した版 $v_3$（$v_2$ とは別の枝）を作ります。ライブラリで出力した `par` は次の通りです。

| 版 | `par` | 連結成分 |
| --- | --- | --- |
| 0 | $[-1, -1, -1, -1]$ | {0} {1} {2} {3} |
| $v_1$ | $[-2, 0, -1, -1]$ | {0,1} {2} {3} |
| $v_2$ | $[-2, 0, -2, 2]$ | {0,1} {2,3} |
| $v_3$ | $[-3, 0, 0, -1]$ | {0,1,2} {3} |

- $v_3$ を作るとき：$v_1$ で 1 の根は 0（大きさ 2）、2 の根は 2（大きさ 1）なので、2 を 0 の子にします。
- $v_2$ では 0 と 3 は非連結、$v_3$ では 0 と 2 が連結で 0 と 3 は非連結です。$v_1$ は 2 つの版の元になりましたが、書き換わっていません。

### 5.4. 実装

```cpp
int leader(int v, int x) const {
    while (true) {
        int p = par.get(v, x);
        if (p < 0) return x;
        x = p;
    }
}
int merge(int v, int x, int y) {
    x = leader(v, x), y = leader(v, y);
    if (x == y) return par.set(v, x, par.get(v, x));      // 変化はないが新しい版を作る
    int sx = -par.get(v, x), sy = -par.get(v, y);
    if (sx < sy) swap(x, y), swap(sx, sy);
    int w = par.set(v, x, -(sx + sy));                    // 大きい方の根の大きさを更新
    return par.set(w, y, x);                              // 小さい方の根を付け替え
}
```

## 6. 計算量

- 永続配列：取得 $O(\log_8 n)$、更新 $O(8 \log_8 n)$（時間・メモリ）。
- 永続 Union-Find：連結判定・辺の追加とも $O(\log n \cdot \log_8 n)$。

**union by size で高さが $O(\log n)$ になる理由**：根でない頂点 $x$ の深さが 1 増えるのは、$x$ を含む成分が、それ以上の大きさの成分の下にくっつくときだけです。そのとき $x$ を含む成分の大きさは 2 倍以上になります。大きさは $n$ 以下なので、深さは $\log_2 n$ 以下です。

## 7. 問題

【問題 1】は、版の番号の配列 `ver` を作り（`ver[0] = uf.initial_version()`）、$G_i$ の版を `ver[i + 1]` に入れていけば、`merge` と `same` で解けます。

【問題 2】クエリを先読みしてよいとき、永続 Union-Find を使わずに解いてください。
（ヒント：版の関係は木になります（各版は 1 つの親の版から作られる）。この木を DFS しながら、子に降りるときに辺を足し、戻るときに取り消す「undo 付き Union-Find」を使えば、各版の質問にその版にいるときに答えられます。）

### 関連問題

- [Persistent UnionFind (Library Checker)](https://judge.yosupo.jp/problem/persistent_unionfind)

## 8. まとめ

- 永続配列は、配列を木で持ち、更新では根から葉までの道だけをコピーする（path copying）。
- 永続 Union-Find は、親の配列を永続配列にし、経路圧縮を使わず union by size だけで高さを $O(\log n)$ に保つ。

## 9. 参考文献

- J. R. Driscoll, N. Sarnak, D. D. Sleator, R. E. Tarjan, "Making data structures persistent", Journal of Computer and System Sciences, 1989.
- [Library Checker - Persistent UnionFind](https://judge.yosupo.jp/problem/persistent_unionfind)
