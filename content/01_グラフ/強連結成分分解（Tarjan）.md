#グラフ #DFS #lowlink #強連結成分

ライブラリ：[scc.hpp](https://github.com/winter-2521/cp-library/blob/main/graph/scc.hpp)

## 1. 概要

本記事では、有向グラフを **強連結成分**（互いに行き来できる頂点の極大集合）に分解し、成分を **トポロジカル順**（辺 $u \to v$ があれば $u$ の成分が先）に並べる方法を解説します。2 回の DFS を使う Kosaraju の方法から始め、DFS 1 回で済む **Tarjan の方法** を説明します。どちらも $O(n + m)$ です。

>[!問題] 問題 1：Strongly Connected Components（Library Checker）
>$N$ 頂点 $M$ 辺の有向グラフを強連結成分に分解し、トポロジカル順に出力してください。（$N, M \le 5 \times 10^5$）

## 2. 前提知識

- 深さ優先探索（DFS）
- lowlink（無向グラフ版）：[[lowlinkと二重辺連結成分分解]]

## 3. 素朴な方法

全頂点から BFS をして到達可能性の表を作り、「$u$ から $v$ にも $v$ から $u$ にも行ける」でまとめれば分解できます。$O(n(n + m))$ で、$n = 5 \times 10^5$ では無理です。

## 4. 例

7 頂点、辺 $0 \to 1$, $1 \to 2$, $2 \to 0$, $2 \to 3$, $3 \to 4$, $4 \to 5$, $5 \to 3$, $6 \to 5$, $6 \to 0$。強連結成分は $\{0,1,2\}$, $\{3,4,5\}$, $\{6\}$ で、トポロジカル順は $\{6\} \to \{0,1,2\} \to \{3,4,5\}$ です。

## 5. Kosaraju の方法（DFS 2 回）

1. 全頂点から DFS し、頂点の **帰りがけ順**（全ての辺を調べ終わった順）を記録する。
2. 辺の向きを全部逆にしたグラフで、帰りがけ順の **遅い順** に DFS する。1 回の DFS で訪問した頂点が 1 つの強連結成分で、成分はトポロジカル順に出てくる。

分かりやすいですが、グラフを 2 回なめ、逆向きのグラフも作る必要があります。

## 6. Tarjan の方法（DFS 1 回）

### 6.1. ord と low

DFS で訪問した順番を $\mathrm{ord}[v]$ とし、頂点を訪問したら **スタックに積みます**（成分が確定したら取り出します）。

> $\mathrm{low}[v]$ ＝ $v$ の部分木から、**まだ成分が確定していない（スタックにある）頂点** への辺を高々 1 本使って行ける頂点の $\mathrm{ord}$ の最小値

無向グラフの lowlink と似ていますが、「成分が確定済みの頂点への辺は使わない」点が違います。確定済みの頂点は別の成分で、そこから戻ってくることはできないからです。

### 6.2. 成分の取り出し

帰りがけに $\mathrm{low}[v] = \mathrm{ord}[v]$ なら、$v$ は自分の成分で最初に訪問された頂点（成分の「根」）です。スタックの上から $v$ までを取り出すと、それが 1 つの強連結成分です。

成分は「行き先の成分が先に確定する」順、つまり **トポロジカル順の逆** に取り出されます。最後に番号を反転すればトポロジカル順になります。

### 6.3. 例で追う

4 節の例を頂点 0 から DFS します（実際にトレースした結果）。

| 手順 | 出来事 | スタック | 更新 |
| --- | --- | --- | --- |
| 1 | 0, 1, 2 を訪問 | [0, 1, 2] | ord = 0, 1, 2 |
| 2 | 辺 $2 \to 0$（0 はスタックにある） | | $\mathrm{low}[2] = 0$ |
| 3 | 3, 4, 5 を訪問 | [0, 1, 2, 3, 4, 5] | ord = 3, 4, 5 |
| 4 | 辺 $5 \to 3$（3 はスタックにある） | | $\mathrm{low}[5] = 3$ |
| 5 | 5, 4 の帰りがけ | | $\mathrm{low}[4] = 3$, $\mathrm{low}[3] = 3$ |
| 6 | 3 の帰りがけ：$\mathrm{low}[3] = \mathrm{ord}[3]$ | [0, 1, 2] | **成分 {5, 4, 3} を取り出す** |
| 7 | 2, 1, 0 の帰りがけ | | $\mathrm{low} = 0$ が伝わる |
| 8 | 0 の帰りがけ：$\mathrm{low}[0] = \mathrm{ord}[0]$ | [] | **成分 {2, 1, 0} を取り出す** |
| 9 | 6 を訪問。辺 $6 \to 5$, $6 \to 0$ は確定済みの頂点なので使わない | [6] | |
| 10 | 6 の帰りがけ | [] | **成分 {6} を取り出す** |

取り出した順 {3,4,5}, {0,1,2}, {6} を逆にすると、トポロジカル順 {6}, {0,1,2}, {3,4,5} になります（ライブラリの出力と一致）。

手順 9 で、もし確定済みの 5 への辺も使って $\mathrm{low}[6] = \mathrm{ord}[5] = 5$ としてしまうと、$\mathrm{low}[6] \ne \mathrm{ord}[6]$ になり、6 が成分として取り出されなくなります。

## 7. 実装

再帰を使わず、「次に調べる辺」`it[v]` と呼び出しのスタック `call` で DFS します。

```cpp
while (!call.empty()) {
    int v = call.back();
    if (it[v] < g.start[v + 1]) {
        int to = g.elist[it[v]++];
        if (ord[to] < 0) {                                   // 未訪問
            ord[to] = low[to] = now++, it[to] = g.start[to];
            stk.push_back(to), call.push_back(to);
        } else if (comp[to] < 0) {                           // スタックにある (未確定)
            low[v] = std::min(low[v], ord[to]);
        }
        continue;
    }
    call.pop_back();                                         // 帰りがけ
    if (!call.empty()) low[call.back()] = std::min(low[call.back()], low[v]);
    if (low[v] == ord[v]) {                                  // v は成分の根
        while (true) {
            int u = stk.back();
            stk.pop_back();
            comp[u] = k;
            if (u == v) break;
        }
        k++;
    }
}
// 取り出した順はトポロジカル順の逆なので反転
for (auto &c : comp) c = k - 1 - c;
```

## 8. 問題

【問題 1】は `StronglyConnectedComponents scc(n); scc.add_edge(u, v); scc.build();` の後、`scc.groups` を順に出力すれば解けます。

【問題 2】2-SAT（各変数を真か偽にして、「$x$ または $y$」の形の条件を全部満たす）を解いてください。
（ヒント：「$x$ でないなら $y$」「$y$ でないなら $x$」の辺を張ったグラフで、$x$ と $\neg x$ が同じ強連結成分なら解なし。そうでなければ、トポロジカル順で後ろにある方を真にします。）

### 関連問題

- [Strongly Connected Components (Library Checker)](https://judge.yosupo.jp/problem/scc)
- [Two SAT (Library Checker)](https://judge.yosupo.jp/problem/two_sat)

## 9. まとめ

- Tarjan の方法は、DFS で頂点をスタックに積み、「未確定の頂点への辺」だけで low を計算する。
- 帰りがけに $\mathrm{low}[v] = \mathrm{ord}[v]$ ならスタックから成分を取り出す。取り出し順はトポロジカル順の逆。

## 10. 参考文献

- R. E. Tarjan, "Depth-first search and linear graph algorithms", SIAM Journal on Computing, 1972.
- [Library Checker - Strongly Connected Components](https://judge.yosupo.jp/problem/scc)
