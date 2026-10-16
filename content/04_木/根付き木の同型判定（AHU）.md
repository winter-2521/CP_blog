#木 #同型 #ハッシュ

ライブラリ：[rooted_tree_isomorphism.hpp](https://github.com/winter-2521/cp-library/blob/main/tree/rooted_tree_isomorphism.hpp)

## 1. 概要

2 つの根付き木が **同型**（子の順番を並べ替えると同じ形になる）かを判定します。**AHU 法** では、葉から順に「子の番号を並べた列」に番号を振り、同じ列には同じ番号を付けます。これで木の全ての部分木を同型ごとに分類でき、$O(n \log n)$ です。

>[!問題] 問題 1：Rooted Tree Isomorphism Classification（Library Checker）
>$N$ 頂点の根付き木（根は 0）の各頂点 $v$ を根とする部分木を、同型なものが同じ番号になるように $0, 1, \dots, K-1$ で分類してください。（$N \le 5 \times 10^5$）

## 2. 前提知識

- 根付き木、葉から順の処理（BFS 順の逆）
- ソート（列の辞書順比較）

## 3. 素朴な方法

部分木を「括弧列」で表す方法があります。葉は `()`、子を持つ頂点は「子の括弧列を辞書順にソートしてつなぎ、外側を括弧で囲む」とすると、**同型 ⇔ 括弧列が等しい** です。しかし括弧列の長さは部分木の大きさに比例し、一直線の木では合計 $O(n^2)$ の文字を扱うことになります。

## 4. AHU 法

### 4.1. 列を番号に置き換える

括弧列の代わりに、**子の部分木の番号** を使います。

1. 葉から順に処理する。
2. 頂点 $v$ について、子の番号を昇順に並べた列 $\mathrm{sig}(v)$ を作る。
3. $\mathrm{sig}$ が同じ頂点には同じ番号、違えば新しい番号を振る。

同型な部分木は $\mathrm{sig}$ が等しく、逆に $\mathrm{sig}$ が等しければ子の部分木が（順番を並べ替えて）1 対 1 で同型なので同型です。列の長さは子の数なので、合計 $O(n)$ です。

### 4.2. 高さごとにまとめて番号を振る

同型な部分木は **高さ**（葉までの最大距離）が等しいので、高さ 0, 1, 2, … の順に、その高さの頂点の $\mathrm{sig}$ をまとめてソートし、隣と比べて番号を振ります。子の高さは親より小さいので、子の番号は先に決まっています。

### 4.3. 例で追う

9 頂点、親が $(-, 0, 0, 1, 1, 2, 2, 6, 3)$ の木です（0 の子は 1, 2；1 の子は 3, 4；2 の子は 5, 6；6 の子は 7；3 の子は 8）。

| 高さ | 頂点 | $\mathrm{sig}$ | 番号 |
| --- | --- | --- | --- |
| 0 | 4, 5, 7, 8 | （空） | 0 |
| 1 | 3, 6 | [0] | 1 |
| 2 | 1 | [0, 1]（子 4 → 0、子 3 → 1） | 2 |
| 2 | 2 | [0, 1]（子 5 → 0、子 6 → 1） | 2 |
| 3 | 0 | [2, 2] | 3 |

部分木 1 と 2 はどちらも「葉 1 つ」と「葉を 1 つぶら下げた頂点」を子に持つので同型です。種類数は 4、番号は頂点 0〜8 の順に $3, 2, 2, 1, 0, 0, 1, 0, 0$（ライブラリの出力と一致）。

## 5. 実装

```cpp
for (int h = 0; h <= H; h++) {
    // 高さ h の各頂点の子の番号を昇順に
    for (int i = hs[h]; i < hs[h + 1]; i++) {
        int v = byh[i];
        for (int j = start[v]; j < start[v + 1]; j++) sig[j] = id[ch[j]];
        sort(sig.begin() + start[v], sig.begin() + start[v + 1]);
    }
    auto less = [&](int a, int b) {     // sig の辞書順
        return lexicographical_compare(sig.begin() + start[a], sig.begin() + start[a + 1],
                                       sig.begin() + start[b], sig.begin() + start[b + 1]);
    };
    sort(byh.begin() + hs[h], byh.begin() + hs[h + 1], less);
    for (int i = hs[h]; i < hs[h + 1]; i++) {
        int v = byh[i];
        if (i > hs[h] && !less(byh[i - 1], v)) id[v] = id[byh[i - 1]];   // 前と同じ列
        else id[v] = K++;
    }
}
```

子の列は CSR（1 本の配列）に置き、頂点ごとの `vector` を作らないようにしています。

## 6. 正しさ・計算量

- **正しさ**：高さに関する帰納法で「番号が等しい ⇔ 同型」が示せます。
- **計算量**：子の列のソートが合計 $O(n \log n)$、高さごとのソートは比較が列の長さに比例し、全体 $O(n \log n)$ 程度です。

## 7. 問題

【問題 1】は `rooted_tree_isomorphism(par)` の返り値を出力すれば解けます。

【問題 2】2 つの **根なし** 木が同型か判定してください。
（ヒント：木の中心（1 つか 2 つ）を根にして、2 つの木を 1 つの森にまとめて分類し、根の番号を比べます。中心が 2 つなら両方を試します。）

### 関連問題

- [Rooted Tree Isomorphism Classification (Library Checker)](https://judge.yosupo.jp/problem/rooted_tree_isomorphism_classification)
- [[木の直径]]（中心は直径の真ん中）

## 8. まとめ

- 部分木を「子の番号の昇順の列」で表し、同じ列には同じ番号を振る（AHU 法）。
- 同型な部分木は高さが等しいので、高さごとにソートして番号を付ける。$O(n \log n)$。

## 9. 参考文献

- [Library Checker - Rooted Tree Isomorphism Classification](https://judge.yosupo.jp/problem/rooted_tree_isomorphism_classification)
- A. V. Aho, J. E. Hopcroft, J. D. Ullman, "The Design and Analysis of Computer Algorithms", 1974.
