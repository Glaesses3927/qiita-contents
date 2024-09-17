---
title: C言語で回転する立方体をつくる
tags:
  - 'C'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
まずは見てみましょう．
<iframe width="560" height="480" src="https://www.youtube.com/embed/p09i_hoFdd0?start=1229" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" loading="lazy" allowfullscreen></iframe>
一応，この動画を最初から見ればどのような流れでコーディングをしているのかわかるのですが，めちゃ高速なので理解に苦しみます．したがって，実際に手順を踏んでコーディングし，その背後にある数学についても解説します．



# 前提知識
今回は3次元の回転する立体を2次元に落とし込んで表示させます．したがって，3次元での回転を考える必要があります．ここで便利なのが，**回転行列**です．

簡単のため，2次元の回転を考えることにします．点$(x,y)$を原点中心に角度$\theta$だけ回転した後の点の座標を求めてみましょう．回転には極座標を用いると簡単です．点$(x,y)$の動径を$r$，偏角を$\alpha$とすると，
$$ x = r\cos\alpha ,\ y = r\sin\alpha$$
となります．これを$\theta$回転した後の点$(x',y')$は，
$$ x' = r\cos(\alpha+\theta) ,\ y' = r\sin(\alpha+\theta)$$
となります．加法定理を用いると，
```math
\begin{align}
x'  &= r\cos(\alpha+\theta)\\
    &= r(\cos\alpha\cos\theta-\sin\alpha\sin\theta)\\
    &= x\cos\theta-y\sin\theta\\
y'  &= r\sin(\alpha+\theta)\\
    &= r(\sin\alpha\cos\theta+\cos\alpha\sin\theta)\\
    &= y\cos\theta+x\sin\theta
\end{align}
```
ここで行列を用いると，
```math
\begin{align}
\left(\begin{array}{c}
  x' \\
  y' \\
\end{array}\right)
&=
\left(\begin{array}{cc}
  \cos\theta & -\sin\theta\\
  \sin\theta & \cos\theta\\
\end{array}\right)
\left(\begin{array}{c}
  x \\
  y \\
\end{array}\right)\\
&= R(\theta)\left(
\begin{array}{c}
  x \\
  y \\
\end{array}\right)
\end{align}
```
と表せます．この$R(\theta)$が2次元における回転行列です．

この3次元ver. も存在し，$x$軸，$y$軸，$z$軸周りの回転を表す回転行列は次にようになります．
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
ここでの回転の方向は，$R_x$は$y$軸を$z$軸に向ける方向，$R_y$は$z$軸を$x$軸に向ける方向，$R_z$は$x$軸を$y$軸に向ける方向です．（[回転行列 - Wikipedia](https://ja.wikipedia.org/wiki/%E5%9B%9E%E8%BB%A2%E8%A1%8C%E5%88%97)）

勝手に自分でプログラミングする際はこれで十分なんですが，今回はちょっと異なります．というのも，動画では点を回転させるのではなく，点は固定して軸（座標・視点）を回転させているためです．詳しい説明は省きますが，“座標を$\phi$回転させた状態の点の位置”は，“もとの状態の座標で点を$-\phi$回転させた位置”と等しいことが分かれば，簡単です．すなわち，$\sin A$を$\sin(-A) = -\sin A$，$\cos A$を$\cos(-A) = \cos A$に変換すればいいんです．したがって，

```math
\begin{align}
R'_x(\theta) = 
\left(\begin{array}{ccc}
  1 & 0 & 0 \\
  0 & \cos\theta & \sin\theta\\
  0 & -\sin\theta & \cos\theta\\
\end{array}\right)\\
R'_y(\theta) = 
\left(\begin{array}{ccc}
  \cos\theta & 0 & -\sin\theta \\
  0 & 1 & 0 \\
  \sin\theta & 0 & \cos\theta\\
\end{array}\right)\\
R'_z(\theta) = 
\left(\begin{array}{ccc}
  \cos\theta & \sin\theta & 0\\
  -\sin\theta & \cos\theta & 0\\
  0 & 0 & 1
\end{array}\right)
\end{align}
```

これを掛けていきます！

<details><summary>補足</summary>
列ベクトル$(x,y,z)$に右から$R_x, R_y, R_z$をかけていっても，座標を回転できます．動画ではこれを用いています．
</details>

以上のことが分かれば，もうコード書けます！



# 実際に書いてみよう！
## 1. 回転後の座標を求める
行列の計算は面倒くさいので，[Linear Algebra Calculator](https://www.symbolab.com/solver/linear-algebra-calculator)というWebツールを用います．

https://www.symbolab.com/solver/linear-algebra-calculator

点$(i,j,k)$を$x$軸回りに$A$，$y$軸回りに$B$，$z$軸回りに$C$だけ回転させた点の座標を求めます．

[結果](https://www.symbolab.com/solver/linear-algebra-calculator/%5Cbegin%7Bpmatrix%7D%5Ccos%5Cleft(C%5Cright)%26%5Csin%5Cleft(C%5Cright)%260%5C%5C%20-%5Csin%5Cleft(C%5Cright)%26%5Ccos%5Cleft(C%5Cright)%260%5C%5C%200%260%261%5Cend%7Bpmatrix%7D%5Cbegin%7Bpmatrix%7D%5Ccos%5Cleft(B%5Cright)%260%26-%5Csin%5Cleft(B%5Cright)%5C%5C%200%261%260%5C%5C%20%5Csin%5Cleft(B%5Cright)%260%26%5Ccos%5Cleft(B%5Cright)%5Cend%7Bpmatrix%7D%5Cbegin%7Bpmatrix%7D1%260%260%5C%5C%200%26%5Ccos%5Cleft(A%5Cright)%26%5Csin%5Cleft(A%5Cright)%5C%5C%200%26-%5Csin%5Cleft(A%5Cright)%26%5Ccos%5Cleft(A%5Cright)%5Cend%7Bpmatrix%7D%5Cbegin%7Bpmatrix%7Di%5C%5C%20%20j%5C%5C%20%20k%5Cend%7Bpmatrix%7D?or=input)はこのようになりました．（動画と同じになるように項順を入れ替えています）

```math
\begin{align}
\left(\begin{array}{ccc}
  \cos C & \sin C & 0 \\ 
  -\sin C & \cos C & 0 \\
  0 & 0 & 1 
\end{array}\right)
\left(\begin{array}{ccc}
  \cos B & 0 & -\sin B \\
  0 & 1 & 0 \\
  \sin B & 0 & \cos B
\end{array}\right)
\left(\begin{array}{ccc}
  1 & 0 & 0 \\
  0 & \cos A & \sin A \\
  0 & -\sin A & \cos A
\end{array}\right)
\left(\begin{array}{c}
  i\\ j\\ k
\end{array}\right)\\
=
\left(\begin{array}{c}
  j \sin A \sin B \cos C - k \cos A \sin B \cos C + j \cos A \sin C + k \sin A \sin C + i \cos B \cos C \\
  j \cos A \cos C + k \sin A \cos C - j \sin A \sin B \sin C + k \cos A \sin B \sin C - i \cos B \sin C \\
  k \cos A \cos B - j \sin A \cos B + i \sin B 
\end{array}\right)
\end{align}
```

以上より，
```c
float calculateX(int i, int j, int k) {
  return j * sin(A) * sin(B) * cos(C) - k * cos(A) * sin(B) * cos(C) +
         j * cos(A) * sin(C) + k * sin(A) * sin(C) + i * cos(B) * cos(C);
}

float calculateY(int i, int j, int k) {
  return j * cos(A) * cos(C) + k * sin(A) * cos(C) -
         j * sin(A) * sin(B) * sin(C) + k * cos(A) * sin(B) * sin(C) -
         i * cos(B) * sin(C);
}

float calculateZ(int i, int j, int k) {
  return k * cos(A) * cos(B) - j * sin(A) * cos(B) + i * sin(B);
}
```
というようにコーディングできます．$A, B, C$は回転角度，$i, j, k$は点の座標です．


## 2. 3Dを2Dに
次に，3次元上にある点を2次元に落とし込みます．動画では，立方体の$z$座標に`distanceFromCam`（初期値100）を足し，$xy$平面に映し出しています．求める式は次の通りです．$height, width$は投影する画面の縦横サイズで，$cubeWidth$は立方体の1辺の半分の長さです．
```math
\begin{align}
xp &= \frac{width}{2} - 2 \times cubeWidth + \frac{K1 \times x \times 2}{z}\\
yp &= \frac{height}{2} + \frac{K1 \times y}{z}
\end{align}
```
両式とも，1項目の$\frac{width}{2}, \frac{height}{2}$は，投影画面の中心を起点に持ってくるためです．上式の2項目$- 2 \times cubeWidth$も位置調整です．すこし左に寄せています．最後の項における$K1$は表示倍率です．大きく表示したければこの値を大きく，小さくしたければこの値を小さくすればできます．遠い点ほど小さく表示させたいので，$z$で割って（動画では$ozz = 1/z$を掛けて）います．$x$座標にだけ$2$を掛けているのは，表示する文字が高さより幅が短いためです．幅を2倍して，正方形に近づくようにしています．

立方体ですから，手前側にある面だけが表示されてほしいです．そのために`zBuffer`という配列に$z$座標の逆数を記録しておきます．もし新たな点の表示位置が重なれば，`zBuffer`の値と比較し，新たな点の方が大き（i.e. $z$座標が小さ，近）ければ更新します．


## 3. 描写！
考え方は以上の通りです．これらを駆使して，描写していきます．

変数は以下の通りです．
```c
float A, B, C;                  // 座標の回転角

float cubeWidth = 20;           // 立方体の1辺の長さの半分
int width = 160, height = 44;   // screenの縦と横の長さ
float zBuffer[160 * 44];        // z座標の逆数を記録するバッファ
char buffer[160 * 44];          // 表示する文字を記録するバッファ
int backgroundASCIICode = ' ';  // 背景の文字
int distanceFromCam = 100;      // カメラ（視点・screen）から立方体までの距離（z座標）
float K1 = 40;                  // 表示倍率（拡大係数）

float incrementSpeed = 0.6;

float x, y, z;
float ooz;
int xp, yp;
int idx;
```

まず，screenをきれいにします．
```c
printf("\x1b[2J");
```
こうすることで，全面がまっさらな状態になります．

次に，配列の初期化です．
```c
memset(buffer, backgroundASCIICode, width * height);
memset(zBuffer, 0, width * height * 4);
```
`zBuffer`の方はchar型なので4倍してあります．値は0，すなわち無限遠として初期化します．

次に，立方体の各点の計算です．
```c
for (float cubeX = - cubeWidth; cubeX < cubeWidth; cubeX += incrementSpeed) {
  for (float cubeY = - cubeWidth; cubeY < cubeWidth; cubeY += incrementSpeed) {
    calculateForSurface(cubeX, cubeY, -cubeWidth, '.');
    calculateForSurface(cubeWidth, cubeY, cubeX, '$');
    calculateForSurface(-cubeWidth, cubeY, -cubeX, '~');
    calculateForSurface(-cubeX, cubeY, cubeWidth, '#');
    calculateForSurface(cubeX, -cubeWidth, -cubeY, ';');
    calculateForSurface(cubeX, cubeWidth, cubeY, '+');
  }
}

void calculateForSurface(float cubeX, float cubeY, float cubeZ, int ch) {
  x = calculateX(cubeX, cubeY, cubeZ);
  y = calculateY(cubeX, cubeY, cubeZ);
  z = calculateZ(cubeX, cubeY, cubeZ) + distanceFromCam;

  ooz = 1 / z;

  xp = (int)(width / 2 - 2 * cubeWidth + K1 * ooz * x * 2);
  yp = (int)(height / 2 + K1 * ooz * y);

  idx = xp + yp * width;
  if (idx >= 0 && idx < width * height) {
    if (ooz > zBuffer[idx]) {
      zBuffer[idx] = ooz;
      buffer[idx] = ch;
    }
  }
}
```
各面を`.$~#;+`で塗分けます．$z$軸を垂直上方向とすると，底面が`.`，上面が`#`という具合です．`idx`は，配列が1次元なのでそのインデックスを求めています．

最後に実際に標準出力に表示させます．
```c
printf("\x1b[H");
for (int k = 0; k < width * height; k++) {
  putchar(k % width ? buffer[k] : 10);
}
A += 0.005;   // 回転
B += 0.005;   // 回転
usleep(1000); // 描画速度調整
```
1行目でカーソルを左上に移動します．上書きするような感じです．screenの幅分表示したら改行（ASCIIで10）しています．



# 参考
https://youtu.be/p09i_hoFdd0
https://rikei-tawamure.com/entry/2019/11/04/184049
https://ja.wikipedia.org/wiki/%E5%9B%9E%E8%BB%A2%E8%A1%8C%E5%88%97
https://www.symbolab.com/solver/linear-algebra-calculator
https://www.mm2d.net/main/legacy/c/c-06.html
https://github.com/servetgulnaroglu/cube.c