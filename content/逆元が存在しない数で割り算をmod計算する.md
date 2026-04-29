#数学 #mod #小テク

## 内容

AがBで割り切れるような条件下でA/B mod Xを求めたいとする。
このとき、mod X上でBの逆元が存在しなくても、A mod BXを求めてからBで割ることでA/B mod Xを計算できる。

## 証明

(略証)
A/B = QX + R (0<= R < X) と表せて、Rの値を知りたい。
両辺Bをかけると A = QBX + BR (0 <= BR < BX) となり、A mod BXの値はBの倍数かつA/B mod XのB倍になっていることが分かる。

## 例題

https://atcoder.jp/contests/abc293/editorial/5966
