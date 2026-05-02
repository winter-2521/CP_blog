#典型 
#### 有向グラフで、順方向と逆方向のコストがちょうど $-1$ 倍のとき

- ある頂点 $v$ から二通りの距離でたどり着ける頂点 $v'$  があるとき、その連結成分は正コストの閉路を持つ。
- 正コストの閉路がないとき、$\text{dist}(x,y) = \text{dist}(v,y)-\text{dist}(v,x)$ である。

参考：https://atcoder.jp/contests/abc280/editorial/5303

