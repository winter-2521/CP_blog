
## 問題名

## 解説

#### よく使っているテンプレート

```cpp
#include <bits/stdc++.h>

using namespace std;

/*

 winter's kyopro template

*/

//*/

#include <atcoder/all>

using namespace atcoder;

//*/

//                                  ......Jgg+.

//                            ..jQHHMHHHHMMHHMNHHmaJ+&&ga&&J-....          ..(-.

//                         .JdMHMM@@@@@MMHyyyWHHWMHHMMM@MMMHHHMMMNmmm&-..&HMMMMMm.

//                       .(WHM@@@@@@@@MMWyyWWWH@@@@@g@@@@@MH@@@@@@@@MMHMMMHVyyVHMNHHHm.

//                      .dMM@@@@@@@@@MHVWWWMMM@@@@@@@@@@@@@@@@@@@@@@@@@@@MkWWWHHHWyyHMn.

//                     .dH@@@@MMMMHHHWWHWH@@@@@@@@@@@g@@@@g@@@@@@@@@@@@@@@MMHyyyyyyyWHMNma..

//                    (WH@@@@NWyyyWWHUuuuXWH@@@@@@@@@@@g@@@@@@M@@@@@@g@@@@@HHkyyyyyVH@@@MMHNm,

//                   (WH@@@@@@HyyWHUuuuuuuuuXWHHHH@@@@@@@g@@@@@MM@@@@@@@@@@HUWkyyyyWM@@@@@@MMNm-

//                  .dH@@@@@@@MkWHkkkkkkXuuuuuuuuXWUWHHHHHHHg@@g@MHHg@@HHHUuuuWkyyyyWM@@@@@@@MMH+

//                 .dMM@@@g@@MHHM@@@@@@@HHkkkXuuuWSuuuuuuuuuuuuXXXUHkXuuuuuuuuXHkyyyyWHM@@@@@@MHNe.

//                 (MM@@@@@@@NW@@@@@@MM@@@@@@@HHWHkkXuuuuuuuuuuuuuuuUHkuuuuuuuuXHWyyyyyVHM@@@g@@MNl

//                .WH@@@@@@@@@M@@@@@MM@@@@@@@M9=d@@@@gHHHHkkkkkkQQQQQkHkQkkHHHkkWkyyyyyyyWH@@@@@MHH-

//               .dH@@@@@g@@@H@@@@@MM@@@@@@H9! (W@@@@@@@@MM@@@@@@@@@@@@MM@@@@@@@@HVyyyyyWWW@@@@@@MNr

//               (HM@@@@@@@@MM@@@@@M@g@@@@B:``.(@@@@@@@@@MH@@@@@@@@@@@@@MM@@@@@@@MHWWkWqH@@@@@@@@MHH_

//              .dHM@@@@g@@@H@@@@@H@@@@@M3_`.` J@@@@@@@@@H?H@@@@@@@@@@@@@HM@@@g@@@@HHWyyVH@@@@@@@@HH~

//              dMM@@@@@@@@HM@@@@@H@@g@#>..... dM@@@g@@@@D.?H@@@@@g@@g@@@@M@@@@@@@@HVWWWH@@@@@g@@@HN-

//            .dMM@@@@g@@gMH@@@@@M@@@@#><?zw&. j@@@@@@g@M>` ?M@@@@@@@@@@@@H@@@@@@@@MH@M@@@@@@@@@@@HM{

//           .dMM@@@@@@@@@M@@@@g@H@@@#!```` _?IjH@@@@@@M$_``.?H@@@@@@g@@@@M@@@g@@@@M@@@@@@@@@@@@@@HM>

//        ..jHH@@@@@g@@g@MM@@@@@@HM@MC......`.`_W@@@@@@D_..-J+dMMM@MM@@@g@H@@@@@@@MH@@@@@@@@@g@@@MHD`

//  .gHHQHMMM@@@@@@@@@@MHM@@@@g@@HM@D~.(ggNQma,.jH@MM@#~.-~~_.. ?WMMHM@@@@H@@@@g@@HHM@@@@@g@@@g@@MMD

//   ZMHM@@@@@@@@@@@@@MHM@@@@@@@MHMMI+MB=?HNHHMn-U@MMM{.````.``.. ?TMMM@@@M@g@@@@@H@H@@@g@@@@@@@@HM$

//    ?WMHMHM@@@@@@g@MM@@@Nc?H@MHMMNd#:+-.dMMNZ>.(WN?U:`.`.``.```.`._!_TMM@@@@g@@MM@MM@@@@@g@@@@@H#:

//    .+MHM@@@@@@@@@MM@@@@@No(HHHVUWN{(MMMMMMMb_`.-?!`.`...-((-..`.`.`. (M@@@@@@@H@@M#M@@@@@g@@@MHR

//   (dMM@@@@@@@@@MHMHWHM@@M81zHD_dMN<(MMMMMMMD_``.``.`. (JgggJ.<!.``. (H@@@g@@@M@@@MMH@@g@@@@@@MHR

//  +MM@@@@@@@@MMHHM@Nm._79!<-(>`(wI?3-?MHWWM@!.`.`.`.`.(TYYYHMMNe..`.(H@@@@@@MHM@@MM@@@@@@@@g@@MHR

// jMHMMM@@@@MMM@@HM@@@@H+ <i(w> jK>:~~__<?7!_``.`.`.`.``.``.` _7MMs(d@@@@@@@MWM@@MHM@@@@@@@g@@@MHD

// dHH#M@@@MMM9>?WH@@@MM@N;. ~!.jMH<:::::_`.``.`..`.``.`.`.`...  .(WHMM@@@@HY! dHHHHM@@@@@g@@@@@MMr

// dHHM@@@MM9!   dHM@MHM@#Nx `` ?MN<_____.`.`......`.``.`.`._~:::((ud@@HH9=``` jH@HH@@@@@@@@@@@@MMR.

// vMHM@MH#>     dMM@HHHMMHM2 `.`?MN, ````.`..jUwz?>++. ``.`_~~:?TY9Y=<~_``...dM9TMH@@M@@@@g@@g@@HH_

//  OMM@H#:      .WMHHH@@MMHN;.`.`_TMm. `.```.jZ=====zw:.``.`..`````..-JJJggHHY! .HH@MHM@@@@@@@@@HN{

//  .dHMHR        _WMHHMC!?TMN+.`.`._vHW&...``.?====zz>_`.`.``````..JdMBYTY^`    .WH@HHMM@@@g@MM@MMR.

//   (MMHR_         _7T=`   (MN+``.` (wXWUUO+-.. _~<<`.`.`..` .-JgHMHHH,          dMM@HHHM@@@@@MHMHNe.

//    (TH9!                  (HNx..`.wdyykX<(HWH4Z1++(((JJ&zwXWWWHHWWXMNo          TMH@MHH@@@@@HHHH#MH+

//                            (WMm&(wwXyyyXXdkW0`.`.`. dWyR+uwyyyyyyyWwHN{          ?THMMH@@@@@HMY7TMHC

//                             jMSXSwyyyyyyWXUWH......(dHWUXXyyyyyyyyykXMHe.            dMM@@@MH#~

//                             dMHXXyyyyyyyWWyXkwXZuuuwXwuUyyyyyyyyyyykwvdMm.           J@M@@@MMD`

//                             (TMM@HHQQkkkWHyyyyyyyyyy0<:;zyyyyyyyyyXXf~_WN-          .dH@@@HM@!

//                                `??TYYYWHMMWyyyyyyyyyWA&uwyyWHkyyyUwVC((dMMMH_       .HH@@MM8!

//                                         vMNWyyyyyyyyyyyyyyyyyWHkQHmdWWyyWHMD`      .dMMMM#=`

//                                          7MMkyyyyyyyyyyyyyyyyyWWHMMkWWWWMMHH+      JMMM9=`

//                                           (TMMHkWyyyyyyyyyyyyyWHWkWH9Y<zUV<WN;      ?!`

//                                            .(d@HMHWkkWyyyyyyyVWHHM8w+ ` _.(HNo.

//                                       .(gkHHMMMMNWyyWWWWVyyypHpW0z;<zO++zV9?WN;

//                                      (WMM9UOOTUv?WWkWVVVWWWWU0CzXz;;;;;;+Z:-dM{

//                                      (MR~1+-<.(w++<?TH9TC1<;;;;;?1;;>;;+v:(dMD`

//                                       ?MH+?1-.._<zz;vC;;;;>++;;>;>;;;;+wC<&dN:

//                                       `(M#!_?u-..(z&++>;;<jV<;>;;;>>>+jZ~(dM=

//                                        .TMm+JZ1--_._~?zz&u9Twzz&&zZ77<!(WMB^

//                                          (WHB1-_(z_..(-____..~_~~_.._-JW#>

//                                        .gHMB! <<.(1(JXQe+((+<<<1+(+1WMMY`

//                                      .dM9!_~```-?1-.(dMD<<<<<(-....dMC

//                                     (W#! ````````` (dH@~````` _?<.dM=

//                                    (H8`````````  .(HHM}` `````` (WB!

//                                   .d#:`` ````  .JHMHM#~``` `` .(M9!

//                                  `(MD ` ````.JkM9=`(H$``````  jM$

//                                   (@{``` ` (HB>`   (@l ` `` `.WK~

//                                  .dMx  ``.jM3`     JM{``` ```.MK`

//                                  .WHHA<+(dMC       dM>````` `(M$

//                                   dHH$_(WM3        dN: ``` ` dN{

//                                   ?MHmJdH#!       .d#~` ````-WD`

//                                     ?TTB9!        (Mb- ` ` .dM>

//                                                 .dH9!(A-. .dM%

//                                                (MNI-.dWyVQd#>

//                                                .TMm+?UVWWM#>

//                                                  ?WMHmdMM9!

//                                                    _?7T=!

// #pragma GCC optimize("Ofast,unroll-loops,no-stack-protector,fast-math")

// #pragma GCC target("avx2,bmi,bmi2,lzcnt,popcnt")

// #pragma GCC target("avx,avx2,fma")

// #pragma GCC target("sse,sse2,sse3,ssse3,sse4,popcnt,abm,mmx,avx,avx2,fma")

#define rep(i,n) for(int i=0;i<n;i++)

#define Rep(i,a,b) for(int i=a;i<b;i++)

#define ALL(x) (x).begin(),(x).end()

#define dbgv(x); for(auto now : x) cout << now << " "; cout << endl;

//using P = pair<int,int>;

using ll = long long;

using ull = unsigned long long;

//*/

template<class T> inline bool chmax(T& a, T b) { if (a < b) { a = b; return 1; } return 0; }

template<class T> inline bool chmin(T& a, T b) { if (a > b) { a = b; return 1; } return 0; }

typedef pair<int, int> pii;

typedef pair<ll, ll> pll;

typedef vector<ll> vll;

typedef vector<int> vint;

random_device rnd;

mt19937 rng(rnd());

const int dx[4] = {1,0,-1,0};

const int dy[4] = {0,1,0,-1};

  

const int dx2[8] = {1,1,1,0,-1,-1,-1,0};

const int dy2[8] = {0,1,-1,1,0,1,-1,-1};

  
  

void naive(){

}

  

void solve(){

}

  

int main() {

  ios::sync_with_stdio(false);

  cin.tie(nullptr);

  cout << fixed << setprecision(25);

  int t = 1; //cin >> t;

  rep(testcase,t) solve();

}
```