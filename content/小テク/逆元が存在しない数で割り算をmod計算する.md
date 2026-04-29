#数学 #mod #小テク

## 内容
$A$ が $B$ で割り切れるような条件下で $A/B \bmod X$ を求めたいとする。

このとき、 $\bmod X$ 上で $B$ の逆元が存在しなくても、 $A \bmod BX$ を求めてから $B$ で割ることで $A/B \bmod X$ を計算できる。

## 証明
(略証)
$A/B = QX + R \ (0 \le R < X)$ と表せて、 $R$ の値を知りたい。

両辺 $B$ をかけると $A = QBX + BR \ (0 \le BR < BX)$ となり、 $A \bmod BX$ の値は $B$ の倍数かつ $A/B \bmod X$ の $B$ 倍になっていることが分かる。
## 例題

https://atcoder.jp/contests/abc293/editorial/5966
