# maple初心者用ノート

## このドキュメントでは自身の研究で実装した内容を通して、maple初心者として躓いた内容をまとめます。

## 目次

- [pade近似の基本(関数とパッケージ)](pade近似の基本(関数とパッケージ))
- [pade近似を関数化(式を関数化する方法)](pade近似を関数化(式を関数化する方法))
- [指数関数のパデ近似を用いた三角関数の近似](指数関数のパデ近似を用いた三角関数の近似)
- [関数の保存と読み込み](関数の保存と読み込み)


## pade近似の基本(関数とパッケージ)

Mapleでは `pade(f, x, [m,n])` を使って、関数 `f` を点 `x=0` 周辺で Pade 近似できます。<br>
`pade(f,x,[m,n])`は`numapprox`というパッケージに属している関数です。<br>
このような関数を使用する場合は`numapprox:-pade(f,x,[m,n])`としなくてはなりません。<br>
また`with(numapprox)`というコマンドを打てば、`numapprox`内の関数がすべてトップレベル(直接コードを書いている場所)で使用することができるようになります。

Pade近似の基本的な使い方：
```maple
with(numapprox);
pade(exp(x),x,[2,2]);
```

パッケージの関数をバインドしない場合：
```maple
numapprox:-pade(exp(x),x,[2,2]);
```
この `:-` 記法は「numapproxパッケージの中にあるpade関数」という意味になります。


## pade近似を関数化(式を関数化する方法)

研究では指数関数のPade近似を多用しています。<br>
そのため指数の肩`x`とPade近似の次数`[m,n]`を入力すれば、それに応じた指数関数のPade近似が出力されるような関数(このような複雑な関数のことをMapleではプロシージャといいます)を作りたいです。<br>
次のようにするとよいです。<br>
式の関数化とプロシージャの定義：
```maple
pade_exp := proc(xval, l1, l2)
  local x, tmpFunc;
　# 式pade()を関数化
  tmpFunc := unapply(pade(exp(x), x, [l1, l2]), x);
  return tmpFunc(xval);
end proc
```

`local`は関数内でローカル変数を生成しています。<br>
`unapply(f(x),x)`は式`f(x)`をxの関数にするコマンドです。
`tmpFunc:=x->pade(exp(x), x, [l1, l2])`でも関数化することは可能ですが、式が変数に依存している場合や評価順序が複雑な場合にはエラーが生じることがあるようです。

`unapply`によって関数化したのち、その関数に入力`xval`を代入しています。<br>
なぜこの操作が必要なのでしょうか。<br>
直接`f:=(x,l1,l2)->pade(exp(x),x,[l1,l2])`としたとします。<br>
この場合、`x=0.1`を代入したとすると関数内では`pade(exp(0.1),0.1.[l1,l2])`となっており、`pade()`に関数ではなく定数が入力されていることがわかります。<br>
そのため一旦ローカル変数`x`の式を生成したのち、それを関数化。その関数に`xval`を代入した値を返すといった手順にしなくてはならないようです。<br>


## 指数関数のパデ近似を用いた三角関数の近似

続いて、Pade近似を用いて三角関数の近似関数を作ります。`sin(x) = (e^(ix) - e^(-ix)/2i)`の指数関数をパデ近似に置き換えます。<br>
三角関数を直接パデ近似すれば良いのでは？と無駄な操作に思えるかもしれませんが、これが研究に必要でした。<br>
指数関数のパデ近似を用いた三角関数の近似を出力するプロシージャ：
```maple
sin_by_padeExp := proc(xval, l1, l2)
  local x, sinapprox;
  sinapprox := unapply(simplify(-1/2*I*(pade_exp(x*I, l1, l2) - pade_exp(-I*x, l1, l2))));
  return sinapprox(xval);
  end proc
```

基本的な構造(関数化したのちに代入)は先ほどと同様です。<br>
注意点は`simplify`(式を簡単にする関数)の位置です。<br>
`uapply`は関数オペレータを返します。`simplify`は「関数」ではなく「式」に作用するため`simplify(unapply())`では誤りです。<br>
したがって`simplify`で簡単な式にしたのちに`unapply`で関数にする必要があります。<br>


## 関数の保存と読み込み
自分で制作した関数は、`save` コマンドを使って関数を `.m` ファイルに保存し、必要なときに `read` コマンドで読み込むことで他のドキュメントでも使用することができます。<br>
MATLABを使っている人は拡張子`.m`だとMATLABのコードと認識されてしまうので拡張子`.mpl`にすればOKです。

関数(プロシージャ)の保存方法：
```maple
# 関数(プロシージャ)の定義
pade_exp := proc(xval, l1, l2)
  local x, tmpFunc;
  tmpFunc := unapply(pade(exp(x), x, [l1, l2]), x);
  return tmpFunc(xval);
end proc

# 関数の保存
save pade_exp, "C:\\Users\\あなたの名前\\Documents\\proc_pade_exp.mpl"
```


関数の読み込み：
```maple
read "C:\\Users\\あなたの名前\\Documents\\proc_pade_exp.mpl"
```

ユーザーマニュアルには`"ファイルの名前.mpl"`だけで保存できるとありましたが、私はエラーが出ました。<br>
調べたところ上のコードのように完全なパスを指定した方が確実です。<br>
パスを指定する場合`/`ではなく`\\`であることに注意してください。<br>
また複数の関数を保存することも可能です。

複数の関数を保存：
```maple
# 関数の定義
pade_exp := proc(xval, l1, l2)
  local x, tmpFunc;
  tmpFunc := unapply(pade(exp(x), x, [l1, l2]), x);
  return tmpFunc(xval);
end proc

sin_by_padeExp := proc(xval, l1, l2)
  local x, tmpFunc;
  tmpFunc := unapply(simplify(-1/2*I*(pade_exp(x*I, l1, l2) - pade_exp(-I*x, l1, l2))));
  return tmpFunc(xval);
end proc

# 関数の保存
save pade_exp,sin_by_padeExp, "C:\\Users\\あなたの名前\\Documents\\proc_pade_exp.mpl"
```

関数名の後に必ず`,`をつけるのを忘れないようにしましょう。
