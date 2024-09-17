---
title: C言語で回転するドーナツをつくる
tags:
  - C
private: false
updated_at: '2024-09-17T19:17:06+09:00'
id: 6d91579d904fa88b9061
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
これは，前回の記事[C言語で回転する立方体をつくる](https://qiita.com/Glaesses/items/401b6891703dd919d296)の続編です．

https://qiita.com/Glaesses/items/401b6891703dd919d296

まずは見てみましょう．
<iframe width="560" height="480" src="https://www.youtube.com/embed/DEqXNfs_HhY" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" loading="lazy" allowfullscreen></iframe>
この動画の中に最小限の説明がされているのですが，凡人な自分の頭では理解が苦しかったので，わかる範囲で解説してみます．

# 前提知識
今回も座標上で回転をするため， **回転行列** を用います．前回の記事で説明しているので，結果だけを書きます．詳しくは前回の記事をお読みください．
```math
\begin{align}
R_x(\theta) = 
\left(\begin{array}{ccc}
  1 & 0 & 0 \\
  0 & \cos\theta & -\sin\theta\\
  0 & \sin\theta & \cos\theta\\
\end{array}\right)\\
R_y(\theta) = 
\left(\begin{array}{ccc}
  \cos\theta & 0 & \sin\theta \\
  0 & 1 & 0 \\
  -\sin\theta & 0 & \cos\theta\\
\end{array}\right)\\
R_z(\theta) = 
\left(\begin{array}{ccc}
  \cos\theta & -\sin\theta & 0\\
  \sin\theta & \cos\theta & 0\\
  0 & 0 & 1
\end{array}\right)
\end{align}
```
ただし，今作では$R_x(\theta), R_z(\theta)$しか用いません．



# 背景の数学
## 1. 省略記法
これから実際に書いていくのですが，$\sin A$やら$\cos j$やら面倒くさいので，省略します．
```math
\begin{align}
c = \sin i, l = \cos i \\
f = \sin j, d = \cos j \\
e = \sin A, g = \cos A \\
n = \sin B, m = \cos B
\end{align}
```
以上のように置き換えて記述します．


## 2. ドーナツをつくる
![SpinningDonut1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3884435/faeb9386-ee78-b167-59eb-fd8ecd0e34f5.png)

まず，$xz$平面にドーナツの断面の円を描きます．断面の円の半径は$1$，原点からその円の中心までは$2$です．したがって，

```math
\begin{align}
\left(\begin{array}{c}
  2 + d \\ 
  0 \\
  f 
\end{array}\right)
\end{align}
```
と書けます．ここで，
```math
h = 2 + d
```
として，
```math
\begin{align}
\left(\begin{array}{c}
  h \\ 
  0 \\
  f 
\end{array}\right)
\end{align}
```
とします．

つぎにこの円を$z$軸中心に$1$周回転させ，ドーナツを作ります．
```math
\begin{align}
\left(\begin{array}{ccc}
  l & -c & 0 \\ 
  c & l & 0 \\
  0 & 0 & 1 
\end{array}\right)
\left(\begin{array}{c}
  h\\ 0\\ f
\end{array}\right)\\
\end{align}
```

![SpinningDonut2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3884435/8b6c5c43-2ec6-91aa-d387-0e2eee71c198.png)
左の画像のようにできました！


## 3. ドーナツを回転させる

ドーナツを回転させます．回転方法は，まず$x$軸を$A$回転したのち，$z$軸を$B$回転させます．よって，

```math
\begin{align}
&\left(\begin{array}{ccc}
  m & -n & 0 \\ 
  n & m & 0 \\
  0 & 0 & 1 
\end{array}\right)
\left(\begin{array}{ccc}
  1 & 0 & 0 \\
  0 & g & -e \\
  0 & e & g
\end{array}\right)
\left(\begin{array}{ccc}
  l & -c & 0 \\ 
  c & l & 0 \\
  0 & 0 & 1 
\end{array}\right)
\left(\begin{array}{c}
  h\\ 0\\ f
\end{array}\right)\\
&=
\left(\begin{array}{c}
  lhm - (chg-fe)n \\
  lhn + (chg-fe)m \\
  che + fg
\end{array}\right)\\
&=
\left(\begin{array}{c}
  lhm - tn \\
  lhn + tm \\
  che + fg
\end{array}\right)
\end{align}
```
となります．ただし，簡単のため
```math
t = chg-fe
```
としました．最後に，$z$軸に関して$5$だけ平行移動させます．視点（画面・screen）からドーナツを少し離すためです．
```math
\begin{align}
\left(\begin{array}{c}
  lhm - tn \\
  lhn + tm \\
  che + fg + 5
\end{array}\right)
\end{align}
```
これがドーナツの各点を表します．


## 4. 3Dを2Dに
次に，3次元上にある点を2次元に落とし込みます．遠くにあるものほど小さく投影したいので，$x$座標に$1/z$を掛けて縮小します．$y$座標も同様です．
```math
\begin{align}
D = 1 / (che + fg + 5)\\
x = D \times (lhm - tn)\\
y = D \times (lhn + tm)
\end{align}
```

次に，ドーナツ表面の各点の明るさを計算します．平行な光源を用意し，光に当たるところは明るく，当たりにくいところは暗くします．平行光源のベクトルは，動画では
```math
\begin{align}
\left(\begin{array}{c}
  0 \\ 1 \\ 1
\end{array}\right)
\end{align}
```
とされています．光に当たるか当たらないかの判定はドーナツ表面の各点における法線ベクトルを用います．法線ベクトルは以下の通りです．平行光源なので点の位置は関係ないため，$h$ではなく$d$を用い，$z$座標は$+5$する前のものです．
```math
\begin{align}
\left(\begin{array}{c}
  ldm - (cdg-fe)n \\
  ldn + (cdg-fe)m \\
  cde + fg
\end{array}\right)\\
\end{align}
```
この法線ベクトルと光ベクトルの内積が$-1$に近いほど明るく，$0$になるほど暗く，$0$以上は光が全く当たりません．したがってマイナスをつけて，
```math
\begin{align}
N=
\left(\begin{array}{c}
  0 \\ -1 \\ -1
\end{array}\right)
\cdot
\left(\begin{array}{c}
  ldm - (cdg-fe)n \\
  ldn + (cdg-fe)m \\
  cde + fg
\end{array}\right)
\end{align}
=
(fe - cdg)\times m - cde - fg - ldn
```
この値が$0\sim 1$の間である点が描画すべき点であり，この値がその点の明るさを示します．



# 実際に書いてみよう！
```c
float A = 0, B = 0;
float i, j;
int k;
float z[1760];
char b[1760];
```
まず変数定義です．$i, j, A, B$は前章で考えたものと同じ変数を示します．投影する画面の縦横サイズは$80 \times 22$としています．配列`b`は表示する文字，配列`z`はその点の$z$座標の逆数を記憶しておくためのものです．
```c
printf("\x1b[2J");
```
これでscreenをきれいにします．全面がまっさらな状態になります．

次に，配列の初期化です．
```c
memset(b,32,1760);
memset(z,0,7040);
```
`b`は`32`，すなわち空白文字で初期化しています．`z`の方はchar型なので4倍してあります．値は0，すなわち無限遠として初期化します．
```c
for(j=0; j < 6.28; j += 0.07) {
  for(i=0; i < 6.28; i += 0.02) {
    float c = sin(i);
    float d = cos(j);
    float e = sin(A);
    float f = sin(j);
    float g = cos(A);
    float h = d + 2;
    float D = 1 / (c * h * e + f * g + 5);
    float l = cos(i);
    float m = cos(B);
    float n = sin(B);
    float t = c * h * g - f * e;
    int x = 40 + 30 * D * (l * h * m - t * n);
    int y = 12 + 15 * D * (l * h * n + t * m);
    int o = x + 80 * y; // 配列のインデックスを計算
    int N = 8 * ((f * e - c * d * g) * m - c * d * e - f * g - l * d * n);
    if(22 > y && y > 0 && x > 0 && 80 > x && D > z[o]) {
      z[o] = D;
      b[o] = ".,-~:;=!*#$@"[N > 0 ? N : 0];
    }
  }
}
```
これで各$0\leq i, j<2\pi$，すなわちドーナツ上の各点について以下の処理を実行します．$c, d, e, f, g, h, D, l, m, n, t$は前章での省略と同じ意味です．$x, y$座標に関して，$30, 15$倍しているのは単に大きく写すため（拡大倍率），$40, 12$を足しているのは，画面中央に表示するためです．$N$に8倍しているのは，$0\sim 1$の各明るさに対し，8つのASCII文字`.,-~:;=!*#$@`を割り当てるためです．

最後に標準出力に表示させます．
```c
printf("\x1b[H");
for(k = 0; k < 1761; k++) {
  putchar(k % 80 ? b[k] : 10);
  A += 0.00004; // 回転
  B += 0.00002; // 回転
}
usleep(30000);  // 描画速度調整
```
1行目でカーソルを左上に移動します．上書きするような感じです．screenの幅分表示したら改行（ASCIIで10）しています．



# 参考
https://youtu.be/DEqXNfs_HhY
https://www.a1k0n.net/2011/07/20/donut-math.html
https://ja.wikipedia.org/wiki/%E5%9B%9E%E8%BB%A2%E8%A1%8C%E5%88%97
https://www.dropbox.com/scl/fi/f0qfi0figuhlpskpn08z9/donut_deobfuscated.c?rlkey=n5xig3ndm4kvjgh4zx0tv6qi3&e=1&st=ikc9liex&dl=0
