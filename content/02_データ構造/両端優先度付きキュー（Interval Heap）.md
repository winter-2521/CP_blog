#データ構造 #ヒープ

ライブラリ：[interval_heap.hpp](https://github.com/winter-2521/cp-library/blob/main/data_structure/interval_heap.hpp)

## 何ができるアルゴリズムか？

最小値と最大値の **両方** を取り出せる優先度付きキュー（Double-Ended Priority Queue）。挿入・最小値の削除・最大値の削除が $O(\log n)$、最小値・最大値の参照が $O(1)$。

>[!問題] Double-Ended Priority Queue (Library Checker)
>整数の多重集合 $S$ に対して、挿入 / 最小値を取り出す / 最大値を取り出す を $Q$ 回処理せよ。$N, Q \le 5 \times 10^5$

## 計算量

| 操作 | 計算量 |
| --- | --- |
| 挿入 | $O(\log n)$ |
| min / max | $O(1)$ |
| pop_min / pop_max | $O(\log n)$ |

## 証明・仕組み

### よくある代替との比較

- `std::multiset`：平衡二分木なので定数倍が重い。
- `priority_queue` を 2 本（最小用・最大用）持ち、取り出したものを「削除予定」として遅延削除：メモリが 2 倍で、削除予定の管理も重い。

Interval Heap は 1 本の配列で済み、普通の二分ヒープとほぼ同じ速さで動く。

### 構造

完全二分木の各ノード $k$ が 2 つの値 $d_{2k} \le d_{2k+1}$（区間 $[d_{2k}, d_{2k+1}]$）を持つ（最後のノードだけ 1 個のことがある）。

**不変条件**：子ノードの区間は親ノードの区間に含まれる。

これにより、左端 $d_{2k}$ だけを見ると min-heap、右端 $d_{2k+1}$ だけを見ると max-heap になる。根の左端が最小値、右端が最大値。

- **挿入**：末尾に追加。新しいノードの区間が親の区間の左にはみ出していれば min-heap として上げ、右にはみ出していれば max-heap として上げる。
- **pop_min**：根の左端を取り出し、末尾の要素を根の左端に置いて、min-heap として下げる。下げる途中で各ノードの「左端 $\le$ 右端」が崩れたら交換する（交換しても区間の包含は保たれる）。pop_max も対称。

どれも木の高さぶん $O(\log n)$。

## 例題・類題

- [Double-Ended Priority Queue (Library Checker)](https://judge.yosupo.jp/problem/double_ended_priority_queue)
- 「容量 $K$ の集合で、あふれたら最大（最小）を捨てつつ最小（最大）も取り出す」ような、両端を使う貪欲。
- 区間の中央値の管理（下半分の max と上半分の min を見る）は通常のヒープ 2 本で足りることが多い。両端からの削除が混ざると DEPQ が要る。
