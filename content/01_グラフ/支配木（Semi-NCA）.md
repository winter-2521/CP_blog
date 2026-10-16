#グラフ #有向グラフ #DFS #UnionFind

ライブラリ：[dominator_tree.hpp](https://github.com/winter-2521/cp-library/blob/main/graph/dominator_tree.hpp)

## 1. 概要

有向グラフと始点 $s$ について、「$s$ から $v$ へ行くどの道も必ず通る頂点」を $v$ の **支配点**（dominator）と呼びます。$v$ に最も近い支配点を **直接支配点** $\mathrm{idom}(v)$ と呼び、これを親にすると木になります（**支配木**）。

本記事では、素朴な $O(nm)$ の方法から始め、DFS の番号と「半支配点」を使う Lengauer-Tarjan 系の方法（Semi-NCA 法）で高速に求める方法を解説します。

>[!問題] 問題 1：Dominator Tree（Library Checker）
>$N$ 頂点 $M$ 辺の有向グラフと始点 $S$ が与えられます。各頂点の直接支配点を出力してください（到達できない頂点は $-1$、$S$ は $S$）。（$N, M \le 2 \times 10^5$）

## 2. 前提知識

- DFS、DFS 木、先行順（行きがけ順）の番号
- Union-Find（経路圧縮）

## 3. 素朴な方法

頂点 $d$ を取り除いて $s$ から到達できる頂点を調べると、到達できなくなった頂点 $v$ は $d$ に支配されています。全ての $d$ について行えば $O(n(n + m))$ で全ての支配関係が分かり、直接支配点は「支配点のうち $v$ に最も近いもの」（支配点の数が最も多いもの）です。$n, m = 2 \times 10^5$ では間に合いません。

## 4. 例

7 頂点、$s = 0$、辺 $0 \to 1, 0 \to 2, 1 \to 3, 2 \to 3, 3 \to 4, 4 \to 1, 4 \to 5, 2 \to 5, 5 \to 6, 3 \to 6$ のグラフを使います。答えは

| 頂点 | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| idom | 0 | 0 | 0 | 0 | **3** | 0 | 0 |

です。4 へは $3 \to 4$ からしか入れないので 3 に支配されます。6 へは $3 \to 6$ と $0 \to 2 \to 5 \to 6$ の両方があるので、0 以外に支配点はありません。

## 5. DFS 番号と半支配点

### 5.1. DFS で番号を振る

$s$ から DFS し、訪れた順（先行順）に番号を振ります。例では隣接を辺の順に見て

| 番号 | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 頂点 | 0 | 1 | 3 | 4 | 5 | 6 | 2 |
| DFS 木の親（番号） | — | 0 | 1 | 2 | 3 | 4 | 0 |

となりました（$0 \to 1 \to 3 \to 4 \to 5 \to 6$ と深く進み、戻って $0 \to 2$）。以下、頂点は番号で呼びます。

DFS 木の性質から、$\mathrm{idom}(w)$ は必ず DFS 木で $w$ の祖先です。

### 5.2. 半支配点

$w$ の **半支配点** $\mathrm{sdom}(w)$ を

$$
\mathrm{sdom}(w) = \min \{ v : v \text{ から } w \text{ への道で、途中の頂点が全て } w \text{ より大きい番号のものがある} \}
$$

と定義します。直感的には「DFS 木の外から $w$ に回り込める、最も根に近い祖先」です。

- 辺 $v \to w$ で $v < w$ なら、$v$ は候補（途中の頂点なし）。
- 辺 $v \to w$ で $v > w$ なら、$v$ の祖先のうち $w$ より大きいもの $u$ を通って来られるので、$\mathrm{sdom}(u)$ が候補。

つまり

$$
\mathrm{sdom}(w) = \min\Bigl( \{ v : v \to w,\ v < w \} \cup \{ \mathrm{sdom}(u) : u > w,\ u \text{ は } v \text{ の祖先（} v \text{ 自身を含む）},\ v \to w \} \Bigr)
$$

です。番号の大きい順（逆先行順）に計算すれば、右側に必要な $\mathrm{sdom}(u)$ は求まっています。

### 5.3. 「祖先のうち最小の sdom」を速く

「$v$ の祖先で $w$ より大きいもの」のうち $\mathrm{sdom}$ が最小のものを素朴に探すと遅いので、**処理済みの頂点（番号 $> w$）を DFS 木の親につなぐ Union-Find** を使います。各頂点に「つながっている祖先の中で sdom 最小の頂点（`label`）」を持たせ、経路圧縮のときに一緒に更新します。

```cpp
// v から，処理済み (番号の大きい) 祖先をたどった中で semi が最小の頂点
auto eval = [&](int v) {
    if (anc[v] < 0) return v;                       // 未処理なら v 自身
    int x = v;
    while (anc[anc[x]] >= 0) stk.push_back(x), x = anc[x];
    while (!stk.empty()) {                          // 上から順に経路圧縮
        int y = stk.back(), a = anc[y];
        stk.pop_back();
        if (semi[label[a]] < semi[label[y]]) label[y] = label[a];
        anc[y] = anc[a];
    }
    return label[v];
};
for (int i = k - 1; i >= 1; i--) {                  // 逆先行順
    for (int v : 逆辺(i)) semi[i] = min(semi[i], semi[eval(v)]);
    anc[i] = dpar[i];                               // 処理済みにして親につなぐ
}
```

未処理の頂点 $v$（$v < w$）では `eval` は $v$ 自身を返し、$\mathrm{semi}[v] = v$ なので、1 つ目の場合も同じ式で扱えます。

### 5.4. 例で追う

逆先行順に計算します（括弧内は元の頂点番号）。

| 番号 $w$ | 入ってくる辺の始点（番号） | 候補 | $\mathrm{sdom}(w)$ |
| --- | --- | --- | --- |
| 6（2） | 0 | 0 | 0 |
| 5（6） | 4（5）, 2（3） | $\mathrm{sdom}$ の最小（4 は未処理なので 4）、2 | 2 |
| 4（5） | 3（4）, 6（2） | 3、6 の祖先で 4 より大きいもの = 6 自身の $\mathrm{sdom} = 0$ | 0 |
| 3（4） | 2（3） | 2 | 2 |
| 2（3） | 1（1）, 6（2） | 1、$\mathrm{sdom}(6) = 0$ | 0 |
| 1（1） | 0（0）, 3（4） | 0、3 から祖先をたどった $\mathrm{sdom}$ の最小 = 0 | 0 |

## 6. 半支配点から直接支配点へ（Semi-NCA）

$\mathrm{idom}(w)$ は「DFS 木の $w$ の親から始めて、番号が $\mathrm{sdom}(w)$ 以下になるまで $\mathrm{idom}$ をたどった頂点」になることが知られています（$\mathrm{sdom}(w)$ と $w$ の親の、支配木での最近共通祖先）。番号の小さい順に計算すれば、たどる先の $\mathrm{idom}$ は求まっています。

```cpp
idom[0] = 0;
for (int i = 1; i < k; i++) {
    int d = dpar[i];
    while (d > semi[i]) d = idom[d];
    idom[i] = d;
}
```

| 番号（頂点） | 親 | $\mathrm{sdom}$ | たどり方 | $\mathrm{idom}$（頂点） |
| --- | --- | --- | --- | --- |
| 1（1） | 0 | 0 | $0 \le 0$ で止まる | 0（0） |
| 2（3） | 1 | 0 | $1 > 0$ → $\mathrm{idom}(1) = 0$ | 0（0） |
| 3（4） | 2 | 2 | $2 \le 2$ で止まる | 2（**3**） |
| 4（5） | 3 | 0 | $3 \to \mathrm{idom}(3) = 2 \to \mathrm{idom}(2) = 0$ | 0（0） |
| 5（6） | 4 | 2 | $4 > 2$ → $\mathrm{idom}(4) = 0$ | 0（0） |
| 6（2） | 0 | 0 | 止まる | 0（0） |

4 節の答えと一致します（ライブラリの出力 `0 0 0 0 3 0 0`）。

## 7. 正しさ・計算量

- **正しさ**：半支配点の定義と、「$\mathrm{idom}(w)$ は $\mathrm{sdom}(w)$ と親の支配木での最近共通祖先」という Lengauer-Tarjan の定理によります（証明は参考文献）。
- **計算量**：DFS が $O(n + m)$、半支配点は経路圧縮つき Union-Find で $O(m \log n)$。6 節のたどりは最悪 $O(n^2)$ ですが、実際にはほぼ線形で、Lengauer-Tarjan 法（バケットを使う版）より速く動きます。

## 8. 問題

【問題 1】は `dominator_tree(n, edges, s)` の返り値を出力すれば解けます。

【問題 2】有向グラフで、頂点 $s$ から $t$ への道を全部遮断するために「1 つだけ」取り除けばよい頂点（$s, t$ 以外）を全て求めてください。
（ヒント：$t$ の支配木での祖先（$s$ と $t$ を除く）が答えです。）

【問題 3】有向グラフで、$s$ から各頂点 $v$ への道について「その辺を消すと $v$ に行けなくなる辺」を求めてください。
（ヒント：各辺 $a \to b$ の真ん中に新しい頂点を挟むと、辺の支配が頂点の支配になります。）

### 関連問題

- [Dominator Tree (Library Checker)](https://judge.yosupo.jp/problem/dominatortree)

## 9. まとめ

- $v$ の支配点 = $s$ から $v$ へのどの道も通る頂点。最も近いものを親にすると支配木。
- DFS の先行順で番号を振り、逆順に半支配点を「処理済みの祖先の sdom の最小」で求める（経路圧縮つき Union-Find）。
- 直接支配点は、親から $\mathrm{sdom}$ 以下になるまで $\mathrm{idom}$ をたどれば求まる（Semi-NCA）。

## 10. 参考文献

- [Library Checker - Dominator Tree](https://judge.yosupo.jp/problem/dominatortree)
- T. Lengauer, R. E. Tarjan, "A fast algorithm for finding dominators in a flowgraph", ACM TOPLAS, 1979.
- L. Georgiadis, R. E. Tarjan, R. F. Werneck, "Finding dominators in practice", JGAA, 2006（Semi-NCA）.
