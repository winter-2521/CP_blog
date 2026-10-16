#グラフ #UnionFind #セグメント木 #オフライン

ライブラリ：[offline_dynamic_connectivity.hpp](https://github.com/winter-2521/cp-library/blob/main/graph/offline_dynamic_connectivity.hpp)

## 1. 概要

グラフに辺を **追加・削除** しながら「2 頂点がつながっているか」「連結成分の値の和」などを答える問題を **動的連結性** と呼びます。Union-Find は辺の追加しかできませんが、**クエリを全部先読みできる（オフライン）** なら、

- 各辺が存在する時刻の区間を **時刻のセグメント木** に載せる
- セグメント木を DFS しながら **巻き戻し可能な Union-Find** で辺を足したり戻したりする

ことで $O(Q \log Q \log n)$ で処理できます。

>[!問題] 問題 1：Dynamic Graph Vertex Add Component Sum（Library Checker）
>$N$ 頂点の空グラフ（頂点 $v$ に値 $a_v$）に、$Q$ 個のクエリ「辺 $u$–$v$ を追加」「辺 $u$–$v$ を削除」「$a_v$ に $x$ を足す」「$v$ の連結成分の値の和を出力」を順に処理してください。（$N, Q \le 3 \times 10^5$）

## 2. 前提知識

- Union-Find（大きさで併合）
- セグメント木（区間を $O(\log n)$ 個のノードに分ける）

## 3. 素朴な方法

質問のたびに、その時点の辺だけで BFS すれば答えは出ますが、1 回 $O(n + m)$ で全体 $O(Q(n + m))$ です。Union-Find は「辺を消す」ができないので、そのままでは使えません。

## 4. 辺の存在区間とセグメント木

### 4.1. 辺は時刻の区間に存在する

クエリに時刻 $0, 1, \dots, Q-1$ を振ると、各辺は「追加された時刻 $l$ から、削除された時刻 $r$ の手前まで」の区間 $[l, r)$ に存在します（最後まで消えなければ $r = Q$）。

### 4.2. 区間をセグメント木に載せる

時刻のセグメント木を作り、各辺の区間 $[l, r)$ をセグメント木の $O(\log Q)$ 個のノード（区間をちょうど覆うノード）に載せます。すると、

**時刻 $t$ に存在する辺 = 根から葉 $t$ までの道の上のノードに載っている辺**

になります。

### 4.3. DFS しながら足して戻す

セグメント木を根から DFS し、

1. ノードに入るとき、そのノードの辺を Union-Find で併合する
2. 葉（時刻 $t$）に来たら、その時刻の質問に答える
3. ノードから出るとき、1. で併合した分を **巻き戻す**

とします。葉 $t$ にいるときは、ちょうど時刻 $t$ に存在する辺が全部併合されています。

## 5. 巻き戻し可能な Union-Find

**経路圧縮をしない**（大きさで併合だけ）Union-Find では、併合は「根 $b$ の親を根 $a$ にする」だけなので、併合した $b$ をスタックに積んでおけば、`par[b] = -1` とするだけで戻せます。経路圧縮なしでも、大きさで併合すれば木の高さは $O(\log n)$ です。

### 5.1. 値の和を持つ

各頂点に「Union-Find の木での部分木の値の和」`sum` を持ちます。

- 併合（$b$ を $a$ の下に）：`sum[a] += sum[b]`。戻すとき `sum[a] -= sum[b]`。
- 質問：根の `sum`。
- **加算** $a_v \mathrel{+}= x$：$v$ から根までの **全ての頂点** の `sum` に $x$ を足す（高さ $O(\log n)$）。

加算を根だけに足すと、後で併合を戻したときに値が間違った側に残ってしまいますが、道全体に足しておけば、`sum[a] -= sum[b]` で $b$ 側の分（$x$ を含む）が正しく引かれます。加算は戻しません（それ以降もずっと有効なので）。

## 6. 例で追う

3 頂点、$a = (1, 10, 100)$ で、次のクエリを処理します。

| 時刻 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| クエリ | 追加 0–1 | 追加 1–2 | 質問 0 | 削除 0–1 | 質問 0 | $a_2 \mathrel{+}= 5$ | 質問 1 | 追加 0–2 | 質問 1 |

辺の存在区間は 0–1：$[0, 3)$、1–2：$[1, 9)$、0–2：$[7, 9)$ です。

- 時刻 2：0–1, 1–2 が存在。0 の成分は {0,1,2} で $1 + 10 + 100 = 111$。
- 時刻 4：1–2 だけ。0 の成分は {0} で $1$。
- 時刻 5：$a_2 = 105$。
- 時刻 6：1 の成分は {1,2} で $10 + 105 = 115$。
- 時刻 8：1–2, 0–2 が存在。$1 + 10 + 105 = 116$。

ライブラリの出力も `111 1 115 116` です。例えば時刻 6 の葉では、区間 $[1, 9)$ を覆うノード（$[1,2), [2,4), [4,8), [8,9)$）のうち $[4, 8)$ に載った 1–2 だけが併合されています。

## 7. 実装

```cpp
auto dfs = [&](auto self, int k, int l, int r) -> void {
    if (l >= T) return;
    size_t mark = hist.size();
    for (auto [u, v] : seg[k]) {                    // このノードの辺を併合
        int a = find(u), b = find(v);
        if (a == b) continue;
        if (size_[a] < size_[b]) swap(a, b);
        par[b] = a, size_[a] += size_[b], sum[a] += sum[b];
        hist.push_back(b);
    }
    if (r - l == 1) {                               // 葉: 時刻 l の操作
        auto &o = ops[l];
        if (o.type == 2) for (int x = o.u; x >= 0; x = par[x]) sum[x] += o.x;
        else if (o.type == 3) res.push_back(sum[find(o.u)]);
    } else {
        int m = (l + r) / 2;
        self(self, 2 * k, l, m), self(self, 2 * k + 1, m, r);
    }
    while (hist.size() > mark) {                    // 巻き戻し
        int b = hist.back(), a = par[b];
        hist.pop_back();
        par[b] = -1, size_[a] -= size_[b], sum[a] -= sum[b];
    }
};
```

## 8. 正しさ・計算量

- **正しさ**：葉 $t$ ではちょうど時刻 $t$ に存在する辺が併合されていて、`sum` は 5.1 節のとおり正しく保たれます。
- **計算量**：各辺は $O(\log Q)$ 個のノードに載り、併合は `find` が $O(\log n)$ なので、全体 $O(Q \log Q \log n)$。

## 9. 問題

【問題 1】は `OfflineDynamicConnectivity` に操作を登録して `solve()` すれば解けます。

【問題 2】辺の追加・削除と「グラフの連結成分の個数」の質問を処理してください。
（ヒント：成分の数は「$n$ − 併合に成功した回数」。併合と巻き戻しで回数を増減させます。）

【問題 3】辺の追加・削除と「グラフが二部グラフか」の質問を処理してください。
（ヒント：Union-Find に「親との色の偶奇」を持たせます（ポテンシャル付き Union-Find の巻き戻し版）。矛盾する辺を足した回数を数えます。）

### 関連問題

- [Dynamic Graph Vertex Add Component Sum (Library Checker)](https://judge.yosupo.jp/problem/dynamic_graph_vertex_add_component_sum)
- [[ポテンシャル付きUnion-Find]]

## 10. まとめ

- 辺が存在する時刻の区間を時刻のセグメント木に載せ、DFS で根から葉までの辺を併合すると、葉でちょうどその時刻のグラフになる。
- 経路圧縮なしの Union-Find は併合を $O(1)$ で戻せる。
- 加算は葉から根までの道全体に足すと、巻き戻しても値が正しく残る。

## 11. 参考文献

- [Library Checker - Dynamic Graph Vertex Add Component Sum](https://judge.yosupo.jp/problem/dynamic_graph_vertex_add_component_sum)
- [CP-Algorithms - Dynamic Connectivity (Offline)](https://cp-algorithms.com/data_structures/deleting_in_log_n.html)
