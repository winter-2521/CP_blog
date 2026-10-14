#グラフ #DFS #lowlink #橋

ライブラリ：[two_edge_connected_components.hpp](https://github.com/winter-2521/cp-library/blob/main/graph/two_edge_connected_components.hpp)

## 1. 概要

本記事では、無向グラフの **橋**（取り除くと連結成分が増える辺）を求め、橋で切った連結成分（**二重辺連結成分**）に分解する方法を解説します。DFS で **lowlink** という値を求めるのが鍵で、全体 $O(n + m)$ です。lowlink は関節点・二重頂点連結成分（[[二重頂点連結成分分解]]）や強連結成分分解（[[強連結成分分解（Tarjan）]]）でも使う、DFS の基本的な道具です。

>[!問題] 問題 1：Two-Edge-Connected Components（Library Checker）
>$N$ 頂点 $M$ 辺の無向グラフ（多重辺あり）を二重辺連結成分に分解してください。（$N, M \le 2 \times 10^5$）

## 2. 前提知識

- 深さ優先探索（DFS）、DFS 木
- 無向グラフの DFS では、木の辺以外の辺（**後退辺**）は必ず祖先と子孫を結ぶこと

## 3. 素朴な方法

辺を 1 本ずつ取り除いて、連結成分が増えるか BFS で調べれば橋が分かります。辺ごとに $O(n + m)$、全体 $O(m(n + m)) = 4 \times 10^{10}$ で間に合いません。

## 4. lowlink

### 4.1. 定義

DFS で頂点を訪問した順番を $\mathrm{ord}[v]$ とします。

> $\mathrm{low}[v]$ ＝ $v$ の部分木（DFS 木で $v$ の子孫と $v$ 自身）から、**後退辺を高々 1 本** 使って行ける頂点の $\mathrm{ord}$ の最小値

$\mathrm{low}[v]$ は、DFS の帰りがけに

$$\mathrm{low}[v] = \min\left(\mathrm{ord}[v],\ \min_{\text{後退辺 } v - w} \mathrm{ord}[w],\ \min_{\text{子 } c} \mathrm{low}[c]\right)$$

で計算できます。

### 4.2. 橋の判定

DFS 木の辺 (親 $p$, 子 $c$) が橋 ⇔ $\mathrm{low}[c] > \mathrm{ord}[p]$

$c$ の部分木から後退辺で $p$ 以上（$p$ かその祖先）に行けないなら、辺 $(p, c)$ を取り除くと部分木が切り離されるからです。逆に、後退辺で $p$ 以上に行けるなら、その後退辺を通って迂回できます。後退辺は閉路に含まれるので、橋になりません。

**多重辺に注意**：$p$ と $c$ の間に辺が 2 本あるとき、2 本目は後退辺として $\mathrm{low}[c] \le \mathrm{ord}[p]$ にするので橋になりません。「来た辺を飛ばす」ときは、頂点ではなく **辺の番号** で区別します。

### 4.3. 例で追う

辺 0: $\{0,1\}$、1: $\{1,2\}$、2: $\{2,0\}$、3: $\{2,3\}$、4: $\{3,4\}$、5: $\{4,5\}$、6: $\{5,3\}$、7: $\{5,6\}$。三角形 $\{0,1,2\}$ と三角形 $\{3,4,5\}$ を辺 3 でつなぎ、5 に 6 をぶら下げたグラフです。

DFS は $0 \to 1 \to 2 \to 3 \to 4 \to 5 \to 6$ と進みます（実際にトレースした値）。

| 頂点 | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| $\mathrm{ord}$ | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
| $\mathrm{low}$ | 0 | 0 | 0 | 3 | 3 | 3 | 6 |

- 2 は後退辺 $\{2, 0\}$ で ord 0 に行けるので $\mathrm{low}[2] = 0$、それが 1, 0 に伝わります。
- 5 は後退辺 $\{5, 3\}$ で ord 3 に行けるので $\mathrm{low}[5] = 3$、4, 3 に伝わります。
- 6 は後退辺がないので $\mathrm{low}[6] = 6$。

橋の判定：

| 木の辺 | 判定 | 橋？ |
| --- | --- | --- |
| (5, 6)（辺 7） | $\mathrm{low}[6] = 6 > \mathrm{ord}[5] = 5$ | **橋** |
| (2, 3)（辺 3） | $\mathrm{low}[3] = 3 > \mathrm{ord}[2] = 2$ | **橋** |
| その他 | $\mathrm{low}[c] \le \mathrm{ord}[p]$ | 橋でない |

## 5. 二重辺連結成分

橋を全部取り除いた連結成分が二重辺連結成分です。BFS をし直してもよいですが、DFS 中にスタックを使うと 1 回で求まります。

- 頂点を訪問したらスタックに積む。
- 帰りがけに $\mathrm{low}[v] = \mathrm{ord}[v]$ なら（$v$ と親の間が橋か、$v$ が根）、スタックの上から $v$ までを取り出す。それが 1 つの成分。

例では、6 の帰りがけに $\{6\}$、3 の帰りがけに $\{5, 4, 3\}$、0 の帰りがけに $\{2, 1, 0\}$ が取り出されます（ライブラリの出力と一致）。

## 6. 実装

再帰を使わず、頂点ごとに「次に調べる辺」`it[v]` を持って DFS します。

```cpp
while (!call.empty()) {
    int v = call.back();
    if (it[v] < g.start[v + 1]) {
        int e = g.elist[it[v]++];
        if (e == par_edge[v]) continue;                  // 来た辺 (辺番号で区別)
        int to = edges[e].first ^ edges[e].second ^ v;
        if (ord[to] < 0) {                               // 木の辺
            ord[to] = low[to] = now++, it[to] = g.start[to], par_edge[to] = e;
            call.push_back(to), stk.push_back(to);
        } else {                                         // 後退辺
            low[v] = std::min(low[v], ord[to]);
        }
        continue;
    }
    call.pop_back();                                     // 帰りがけ
    if (!call.empty()) low[call.back()] = std::min(low[call.back()], low[v]);
    if (low[v] == ord[v]) {                              // v の上の辺は橋 → 成分を取り出す
        ...
    }
}
```

## 7. 問題

【問題 1】は `two_edge_connected_components(n, edges)` で解けます。引数で `is_bridge` を渡すと、各辺が橋かどうかも分かります。

【問題 2】無向グラフで、各辺を「この辺を取り除いたら、頂点 0 と頂点 $n - 1$ が非連結になるか」判定してください。
（ヒント：そのような辺は橋で、しかも 0 と $n-1$ の間の（二重辺連結成分を縮約した木の上の）道に乗っています。）

### 関連問題

- [Two-Edge-Connected Components (Library Checker)](https://judge.yosupo.jp/problem/two_edge_connected_components)
- [ABC075 C - Bridge](https://atcoder.jp/contests/abc075/tasks/abc075_c)

## 8. まとめ

- $\mathrm{low}[v]$ は「$v$ の部分木から後退辺 1 本で行ける最も浅い頂点の ord」で、DFS の帰りがけに計算できる。
- 木の辺 $(p, c)$ は $\mathrm{low}[c] > \mathrm{ord}[p]$ のとき橋。多重辺は辺番号で区別する。
- 橋で切った成分は、DFS のスタックから $\mathrm{low}[v] = \mathrm{ord}[v]$ のときに取り出せる。

## 9. 参考文献

- R. E. Tarjan, "Depth-first search and linear graph algorithms", SIAM Journal on Computing, 1972.
- [Library Checker - Two-Edge-Connected Components](https://judge.yosupo.jp/problem/two_edge_connected_components)
