---
layout: post
title: すごいHASKELLたのしく学ぼう！ を読んだ
categories: ['programming']
---


### TL;DR
- 何となく Haskell 勉強してみようと思って「すごいHASKELLたのしく学ぼう！」を読んだ
- 文章とか挿絵とかかなり独特だったが、内容はとても良く書けていてかなり面白かった
- せっかくなので Functor から Monad に関する現在の理解をブログに残してみる
- 今後も圏論とか勉強しつつ趣味で触っていこうかな、と思えるくらいには好きになった
<br>

気まぐれで Haskell を勉強してみようと思って、二週間ほど前に触り始めてみた。
せっかくなので本の一冊でも読みながら勉強しようと思って選んだのが「[すごいHASKELLたのしく学ぼう！](https://www.amazon.co.jp/dp/B009RO80XY/ref=dp-kindle-redirect?_encoding=UTF8&btkr=1)」である。
これは原著で「[Learn You a Haskell For Great Good!](http://learnyouahaskell.com/)」というものがあり自分は大体原著の方を読んでいくタイプだが、翻訳本訳者の一人である村主氏に追悼の意を示すという意味で翻訳本の方を読んでみることにした。

この本は謎の挿絵や明らかに遊んでいる節見出しなどからして面白いが、訳者序文で書いてあるように中身は相当しっかりしていると思う。
自分が何かしらのプログラミング言語の初学者向けの本を読むと「表層的な文法とかが分かるだけで、それ以上の興味が持てない」ということが多いが、この本は違った。
事前知識をほぼ仮定せず（と言ってもさすがに型って何？とかのレベルではキツいと思う）、小難しい概念は導入せず、かと言ってごまかして説明するのではない、というのは凄いと思った。

この本を読んだのとちょっと公式ドキュメントを読んだくらいでまだ大して理解はしてないのだが、せっかく読んだので全体を通しての感想と特に印象的だった `Functor` ~ `Monad` 部分をブログに残しておく。
Haskell がとても圏論的なので何をやりたいのかという気持ちが理解しやすい、とか型クラスとポリモーフィズム、なども感銘を受けたのだが、それらはまた今度の機会があったら書くことにしよう。

この記事を読んでためになるような対象は、 Haskell ちょっと触ってみて何となく書き方とか分かったけど `Functor` とか `Monad` などが結局何者なのか分かってない、というくらいのレベルだと思われる。


### 全体を通して
前半は Haskell の基本的な文法とか他のプログラミング言語でもよく扱うような問題を Haskell で書いてみるという感じ。
型クラスも登場するが最初なので平易な例を並べた肩慣らしという感じだし、それ以外のパターンマッチやガードや再帰などは「Haskell だったらこんな風に書くのね」ということで慣れさせるという感じ。

第5章の高階関数は Haskell に本格的に足を踏み入れるところ。
個人的にはこの辺から俄然面白くなってくる。
まず最初のカリー化関数のところが良かった。
具体的な例を見てみる。

```
Prelude> :t max
max :: Ord a => a -> a -> a
```

最初の `Ord a =>` は型クラス制約で、これは `max` 関数を適用する引数 `a` が `Ord` 型クラスのインスタンスであることを要請するもの。
ごくごく簡単に説明すると、型クラスは型ではなく、型クラスのインスタンスが型になっていて、メソッドが複数の異なる型に適用できるようになっている（ポリモーフィズムや！）。
`Ord` 型クラスのインスタンスであれば順序づけられるようになっていて、例えば `Int` や `Char` は `Ord` 型クラスのインスタンスなので `max 2 3` や `max 'a' 'z'` ができる。

型シグネチャの残りの部分は `a -> a -> a` という形で、これは 2 つの引数を取って 1 つの結果（ここでは `max` なので大きい方）を返すという風に読む。
カリー化を考えるまではこの型シグネチャはちょっと読みづらい（素直に気持ちを表すなら `max (a, a) -> a` なんだからそう書いた方が分かりやすい）と思っていたが、これは自分が浅はかすぎた。
この型シグネチャは `a -> (a -> a)` と読むこともできて、これは 1 つの引数を取って「`a` 型の値を引数に取り `a` 型の値を返す関数」を返す関数だと読むこともできる（高階関数や！）。
max の例で言うと `(max 2) 3` と書くことができて、これは `(max 2)` が「引数を 1 つ取ってそれを 2 と比べて大きい方を返す関数」を返していて、それに 3 を適用しているということになる。
これは関数の部分適用であり、常に引数を 1 つだけ取るように考えることができるのでカリー化である。

これだけ言われても「あっそ」って感じかもしれないが、これはとても重要で、型シグネチャを眺めると実に色々なことが理解できるようになっている。
その後は `map` や `filter` やラムダ式といったものを導入している。
この辺も書き方が上手くて、なんとなく型シグネチャの読み方が分かった段階で高階関数として他の言語でもよく使う `map` などを出しておいて、後の章で `Functor` として捉え直すという書き方になっている。
多分ここでいきなり `Functor` とかを出すとまあまあの数の読者が脱落してまうのではないかと思う。
ちなみに `map` は以下のように定義されていて、この段階の知識でも `(a -> b)` という関数とリスト `[a]` を取り、関数を適用させたリスト `[b]` を返すと読むことができる。

```
Prelude> :t map
map :: (a -> b) -> [a] -> [b]
```

この後の `($)` を使った関数適用や `(.)` による関数の合成やポイントフリースタイルなどもイカしているのだが、まあ書き方というか見た目の範疇というところもあるので今回は割愛する。

第5章に関して長々と書いてしまった。
その後の第6章のモジュールはまあいいとして、第7章の型と型クラスのところはこの本のポイントの一つだと思う。
ここで　Haskell の型と型クラスについてもう少し理解を深めるが、`Functor` とかを読み飛ばさずにしっかりと理解して先に進んでいかないと `Applicative Functor` とかに到達した時についていけなくなる可能性がありそう。

第8,9章の入出力のところはいかにして Haskell が副作用を持たない純粋な関数とそうでない副作用を持つ純粋でない関数とを共存させるかという話。
これは Haskell が実用的な言語として在るためには本質的に重要だと思うが、実用性とか自分はよく分かってないのであまり言えることがない（これは自分が分かってないだけということで、記述自体は分かりやすい）。
ここで `do` 記法とか `return` とか思いっきり `Monad` の話が出てくるが、この段階ではそういうもんだという感じで話が複雑になりすぎないように進めている。

第10章の具体的なアルゴリズムを実装してみるという話は普通に楽しいけど置いといて、その後の `Applicative Functor` と `Monoid` と `Monad` がこの本の本領発揮というところ。
流し読みで理解できるようなものでは決してないが、個人的にはしっかりと読めば理解できるように書いていると思う。
実際のところ、これを関数型言語に馴染みのないプログラマが読んだとしてどこまで理解できるのかは自分にはよくわからないが（なので、読んでどうだったという感想がある人は教えてください）。
しかし、読者にあまり馴染みのないであろう概念をできるだけ噛み砕いて真正面から説明しているという感じがして、自分としては好印象だった。
文脈という言葉を、圏論的には別の圏での対象、という感じの意味で使っていて、こういうところは自分としてはもっと数学っぽく説明してくれた方が理解はしやすかった気がするが、この本のコンセプトとしてはそういうものを持ち出さずに説明しきるというものだと思うし、それは結構成功していると感じる。

`Functor` と `Applicative Functor` と `Monad` についてここまで何も具体的な話をしてこなかったが、以降で少し詳しく現時点での理解を残しておくことにする。

全体を通してよく書けていて読んでいて楽しかった。
Haskell でこれを作りたい、みたいな気持ちはまだないけど、今後も趣味的に色々やっていきたいなと思えるくらいに好きにはなった。


### Functor
以降では定義から初めて具体的な例を基に理解をして、抽象的に捉え直してみるという進め方をする。
具体例としては全てリスト `[]` を使うことにして、どんなことができるのかを見ていく(`Maybe` とか使った方がもうちょっと Haskell っぽい感じもするが、多くの人にとってはリストの方が分かりやすいだろう）。
定義はこれ。

```
Prelude> :i Functor
class Functor (f :: * -> *) where
  fmap :: (a -> b) -> f a -> f b
  (<$) :: a -> f b -> f a
...
```

まず `(f :: * -> *)` という部分だがこれは `kind` と呼ばれる型の種類を示すようなもので、ここでの意味は「`f` とは 1 つの具体型（例えば `Int` でこれの kind は `Int :: *`）を取って 1 つの具体型を返す型コンストラクタ（型引数を取って具体型を生み出すもの）であれ」と指定している。
次いで `fmap` と `(<$)` なる関数が定義されていることが分かる。

抽象的な定義では感じが掴みづらいと思うので、具体的な例を考えよう。
型コンストラクタ `f` をリスト `[]` に限定して考えてみる。
この `[]` は 1 つの具体型 `a` を取って 1 つの具体型 `[a]` を返すものになっているので、`kind` がお望みのものになっている。
ちなみに、Haskell における `[a]` は `a : []` の syntactic sugar である。

これで `fmap` の定義を書き直すと `fmap :: (a -> b) -> [a] -> [b]` となる。
これは... `map` だ。

実際に `Functor instance` としての `[]` の定義を見てみる、つまり `fmap` が `[]` に対してどう作用するのかの定義をちゃんと見てみると、これは [GHC.Base 内で定義](https://github.com/ghc/ghc/blob/076f5862a9e46eef762ba19fb7b14e75fa03c2c0/libraries/base/GHC/Base.hs#L964-L966) されていて、 確かに `map` そのものになっている。

```
instance Functor [] where
    fmap = map
```

ということで `map` とは `fmap` を `[]` の場合に適用する場合だったということが分かる。
使い方としては `(a -> b)` なる関数と `[a]` を渡せば `[b]` を返してくれるというものなので例えば以下のように使える。

```
Prelude> fmap (>2) [1..3]
[False,False,True]
```

別段難しいところはないと思うが、返ってきている `[]` の中身が `Bool` であることは注目してもよいかもしれない。
これは `(a -> b)` がここでは `(>2)` という数字を取って `Bool` という異なる型を返す関数であり、それを `[1..3]` の中の要素それぞれに適用したものとなるためである。
当たり前のこと言ってるのだが、例えば `filter` は `filter :: (a -> Bool) -> [a] -> [a]` を見てみると、これは返すのが元々の `[a]` であるため、上の例で `fmap` を `filter` に置き換えると返ってくるのが `[3]` になる。
それも当たり前じゃんと言いたくなるところだが、他のプログラミング言語でもこんな感じだったよねと読み飛ばさずに、ジッと型を眺めて Haskell 的に捉え直すのが後々を理解していくためには必要な気がしている。

`(<$)` の方も見てみよう。
これは `[]` に当てはめると `a -> [b] -> [a]` となっている。
ということで `a` と `[b]` を渡せば `[]` の中身を `b` から `a` に変更してくれるものと読める。
実際にそうなっていることを確かめよう。

```
Prelude> (<$) "outside" ["inside"]
["outside"]
```

ということで `Functor` を具体的に `[]` の場合で見てきたが、ここでいくぶん抽象的に捉え直してみる。
本の言葉を借りれば `Functor` とは文脈を持った値だということになっていて、`[]` の場合は非決定性（複数の値のどれを取るかは定まってないという任意性のようなもの）という文脈を持つ。
`a -> b` という関数を文脈を持っている `Functor` 値 `f a` に適用して `Functor` 値 `f b` を返すことができる、というのが `fmap` である。

`Functor` という型クラスのインスタンスであればこのように `fmap` が適用されることが保証されている。
なので `Maybe` であれば `fmap (*2) (Just 3)` が `Just 6` になるという具合である。
あとは `Functor` 型クラスのインスタンスとして満たすべき性質を明らかにすれば安心して `fmap` を使えるわけだが、それが `fmap id = id` と `fmap (f . g) = fmap f . fmap g` という `Functor` 則である。
これらは恒等写像や関数の合成が `Functor` 値においても成り立つことを要請するもので、例えば `[]` の簡単な場合で成り立つことはすぐに確認できます。

```
Prelude> fmap id [2]
[2]
Prelude> id [2]
[2]
Prelude> fmap ((*2) . (*3)) [1..3]
[6,12,18]
Prelude> (fmap (*2) . fmap (*3)) [1..3]
[6,12,18]
```

この `Functor` 則こそが我々が興味がある多くの具体的対象において成立するものであり、そして `Functor` 則さえ成り立っていれば `fmap` が期待通りに機能することを信じてよいのである。
ということで一旦抽象的に理解した後は、様々な具体例が出てきても一旦ここまで立ち戻って考え直すことで包括的に理解できるようになっている（はずである）。
お勉強としてはどういう場合に `Functor` 則を満たさないかというものを考えるのは重要だが、本にも書いてあるのでここでは詳しい話はしない。
関数を適用する部分以外にカウンタみたいなものを合わせて定義しておいて、`fmap` するたびにカウンタが増えていくというものを作れば当然 `Functor` 則を満たさないというものである。

ということで `Functor` でした。
これは数学の言葉で言えば圏論における関手でありもっと厳密に議論できるものであるが、自分は詳しく説明するほど理解してないのでまたのお楽しみである。


### Applicative Functor
次いで `Applicative` である（以降では後ろの `Functor` を省略して `Applicative` のみで書いていく。意味的には `Functor` を付けないとよく分からないものだが、Haskell 内では `Applicative` のみで定義されている）。
こいつがどんなことをしたいのか理解するには定義を見るのが良い。

```
Prelude> :i Applicative
class Functor f => Applicative (f :: * -> *) where
  pure :: a -> f a
  (<*>) :: f (a -> b) -> f a -> f b
  liftA2 :: (a -> b -> c) -> f a -> f b -> f c
  (*>) :: f a -> f b -> f b
  (<*) :: f a -> f b -> f a
...
```

まず型クラス制約として `Functor f =>` がある。
これは `Applicative` 型クラスに属するインスタンスは `Functor` 型クラスに属するインスタンスでなければならないことを要請するものになっている。
これで `fmap` が使えるものに限るということになっているのが分かる。

定義されているメソッドは `pure` と `(<*>)` と `liftA2` と `(*>)` と `(<*)` である。
`pure` は定義そのままで、値を取って `Applicative` 値を返すものになっている。
ちょっとどう使うか想像できないかもしれないので後で確認しよう。
`(<*>)` は `fmap` に似ているが最初の引数が `f (a -> b)` となっていて、関数 `(a -> b)` が入っている `Applicative` 値であると言っている。
つまり `Applicative` 値に入っている関数を取り出して、それを `Applicative` 値 `f a` の `a` に適用して、結果として `Applicative` 値 `f b` を返すというものである。
`liftA2` はなかなか面白い。2 引数関数と `Applicative` 値 `f a` と `f b` を与えると `Applicative` 値 `f c` を返すと言っている。
残りの `(*>)` と `(<*)` は個別の `instance` の実装を見ないとなかなか理解し難いので一旦置いておこう。

`instance` の定義を覗くと以下のように定義されていることが分かる。

```
instance Applicative [] where
    pure x    = [x]
    fs <*> xs = [f x | f <- fs, x <- xs]
    liftA2 f xs ys = [f x y | x <- xs, y <- ys]
    xs *> ys  = [y | _ <- xs, y <- ys]
```

リスト内包表記に馴染みがある人なら何をやっているかを読み解くのは難しくないと思うので解説は省略する。
とは言え矢継ぎ早に定義を並べられてしんどい？
しんどくなってきたら具体例を見よう。
`Functor` の時と同じく `[]` を扱う。
`[]` は `Functor` だったが `Applicative` でもある。
まずは `(<*>)` から。

```
Prelude> [(+2), (*2), (^2)] <*> [1,2,3]
[3,4,5,2,4,6,1,4,9]
```

`[]` の中の関数を取り出して `[]` の中身に適用して結果を返していることが見て取れる。
定義と見比べるとやりたいことがしっくりくるんじゃないだろうか？
続いて `pure` の方を見てみる。これは例えば以下のように使える。 

```
Prelude> pure (*2) <*> [1,2,3] 
[2,4,6]
```

`pure (*2)` が `Applictive` 値になっていることがポイントで、この辺がきちんと意識されるとどの型を相手にしているのが分かって、例えば `<*>` は確かに `<*>` でないとダメで `*` ではエラーになるということが自然と理解できる。

やりたいことは分かったけどこんなものを持ち出して何が嬉しいのか、という疑問も出るかもしれない。
この `Applicative` はとても強力でアプリカティブ・スタイルという書き方を我々に提供してくれる。
まずは `Applicative` とかを忘れて普通の値で以下の計算を考える。

```
Prelude> (+) 2 3
5
```

これに何かしらの文脈を付与した上で計算したいということを考える。
これまでやってきたように `[]` で非決定性という文脈を加えることにすると、例えば以下のように書ける。

```
Prelude> pure (+) <*> [2,4,6] <*> [3,5,7]
[5,7,9,7,9,11,9,11,13]
Prelude> (+) <$> [2,4,6] <*> [3,5,7]
[5,7,9,7,9,11,9,11,13]
```

2 つの計算は等価である（`Applicative` ならば `pure f <*> x = fmap f x = f <$> x` と書ける）。
これは強力だ！！！
生の関数を使った計算を `Applicative` 値のものにしたかったら然るべき場所に `<$>` と `<*>` を入れるだけで実現できる。
これは（期待通り） `Applicative` だったらどんなものにも適用できる。
ちなみにこれが `liftA2` の動作そのもので、上の例ならば `liftA2 (+) [2,4,6] [3,5,7]` と等価であることが定義から明らかであろう。

それでは `Applicative` が満たすべき性質 `Applicative` 則を列挙してみよう。

- `pure f <*> x = fmap f x`
- `pure id <*> v = v`
- `pure (.) <*> u <*> v <*> w = u <*> (v <*> w)`
- `pure f <*> pure x = pure (f x)`
- `u <*> pure y = pure ($ y) <*> u`

これは結構難しい。
少なくとも今の自分には `Applicative` 足りうるためにこれらが必要最小限のセットだと言うことが直感的に理解できない。
ここでは `[]` を例にとりそれぞれが成り立っていることを見ていくことにする。

1 つ目はこれ。
これが一番大事で、`pure` で関数を持ち上げてから適用するのと `fmap` を使って適用するのが等価だというもの。

```
Prelude> pure (*3) <*> [1..3]
[3,6,9]
Prelude> fmap (*3) [1..3]
[3,6,9]
```

2 つ目は以下で、`id` を持ち上げて適用すればそれは `Applicative` 値に対する `id` になっていなさいというように読める。
そしてそれならば `pure id <*> v = id v` と書いても良さそうなのだが、どうなんだろう？
もちろん両辺の `id` は違う圏における `id` なんだから同じように書くのは厳密にはイカンと怒られそうだが、既に `Functor` 則で `fmap id = id` を使ってるんだよな。
よく理解できてないので詳しい人教えてください。

```
Prelude> pure id <*> [2]
[2]
Prelude> id [2]
[2]
```

3 つ目は `Applicative` 値の関数における合成則で、合成してから適用するのと部分的に適用していくものが等しいことを要請する。
これは素直だがちょっと読みにくいかもしれないので、左辺は以下のように括弧をつけるともう少し気持ちがはっきりするかもしれない。

```
Prelude> (pure (.) <*> [(*2)] <*> [(+3)]) <*> [4]
[14]
Prelude> [(*2)] <*> ([(+3)] <*> [4])
[14]
```

4 つ目も言ってることは難しくない。
`pure` して持ち上げたもの同士を演算するのと、演算してから `pure` で持ち上げたものが等しいことを要請する。
ただしこれを Haskell の具体的なコードで示すやり方はちょっと工夫が必要だ。
というのも、`pure` がどの文脈で実行されているかを指定してやる必要があるからだ。
一つの例として以下のように型注釈を使うというものがある。

```
Prelude> pure (*3) <*> (pure 4 :: [Int])
[12]
Prelude> pure ((*3) 4) :: [Int]
[12]
```

最後はパッと見では一番分かりづらいかもしれないが、実際ちょっと分かりづらい。
中置関数をセクション記法で部分適用するときに関数適用 `($)` を使うという操作が `pure` で持ち上げた `Applicative` 値でも成り立っていることを要請する。
ちなみに普通の値では `3 ^ 2 = (^ 2) 3 = (^ 2) $ 3 = ($ 3) (^ 2)` という風に書けることに相当する。 
特に、`Applicative` においては明示的に `($)` を書かないとどのように演算するかは分からないということに注意が必要だ（`[3] <*> [(^ 2)]` はエラーで `[($ 3)]` としてやる必要がある）。

```
Prelude> [(^ 2)] <*> pure 3
[9]
Prelude> pure ($ 3) <*> [(^ 2)]
[9]
```

ということで少なくとも `[]` に対して `Applicative` 則が成り立つことは理解した。
もっと抽象的に `Applicative` を理解するには、これらの `Applicative` 則が意味する気持ちとなぜこれが我々にとって都合の良いものなのかをもっと理解してあげる必要があるが、自分の理解はそこまで達してないので現段階ではこれくらいにしておく。


### Monad
`Monoid` とかすっ飛ばしてやるが、気持ちはある程度理解できるはず。
まず定義はこれ。

```
Prelude> :i Monad
class Applicative m => Monad (m :: * -> *) where
  (>>=) :: m a -> (a -> m b) -> m b
  (>>) :: m a -> m b -> m b
  return :: a -> m a
  fail :: String -> m a
...
```

型クラス制約として `Applicative` がついているということで、`Monad` であれば `Applicative` であることが要請される。
これは話の流れから自然なことに思えるが、実は以前（例えばこの翻訳本が書かれた当時）はこの型クラス制約はなかったので、翻訳本の説明では型クラス制約がついてない。
とはいえ全ての `Monad` は `Applicative` という記述にもある通り、この型クラス制約は自然なもので、（いつかは自分は知らないが）つけられるようになった。

いくつかメソッドが定義されているが、まずは `(>>=)` から。
これは `Monad` 値 `m a` と `(a -> m b)` なる関数を取って `m b` を返すものである。
ここまで `Functor` と `Applicative` をやってきたので結構気持ちが分かるようになっているのではないだろうか？
普通の値 `a` を取って `Monad` 値 `m b` を返す関数に `Monad` 値 `m a` を入れたいんや、ということである。

次は `(>>)` で、これは典型的には `x >> y = x >>= _ -> y` という実装（[ここ](https://github.com/ghc/ghc/blob/076f5862a9e46eef762ba19fb7b14e75fa03c2c0/libraries/base/GHC/Base.hs#L660)を参照）になり、1 つ目の引数の `Monad` 値に対してそれを無視した上で 2 つ目の `Monad` 値を返すものになっている。

`return` は見た瞬間に `pure` の `Monad` 版だな！と分かる。
ここからも想像できるように Haskell における `return` は他のオブジェクト指向なプログラミング言語とは一線を画している（他の言語を対して知らないのに言っています）。

`fail` は定義を見てもあまり分からないが、これはパターンマッチに失敗しても失敗を `Monad` の文脈で扱うことを可能にしてくれる便利なものである。
今回はここまで触れないが、本の第13章の綱渡りの例題をやってみるとその威力が理解できると思う。

ここからは例によって具体的なものを見てみる。
お察しの通り、`[]` は `Monad` でもある。

`instance` としての定義も見ておく。
`(>>=)` はリスト内包表記なのでやっていることを理解するのは難しくないだろう。
`(>>)` は初見では？となるかもしれないが、これは `Applicative` の `instance` で定義されているメソッド `(*>)` を使うようになっている。
この辺はちょっとややこしいというか持って回った感じになっているがその理由は [ここ](https://github.com/ghc/ghc/blob/076f5862a9e46eef762ba19fb7b14e75fa03c2c0/libraries/base/GHC/Base.hs#L678-L698) に書いてある。内容の本質ではないので今回は割愛。

```
instance Monad []  where
    xs >>= f             = [y | x <- xs, y <- f x]
    (>>) = (*>)
    fail _              = []
```

ということで `fail` 以外の典型的な例は以下のように書ける（念のため書いておくと `\x -> [x, -x]` はラムダ式である）。

```
\Prelude> [1..3] >>= \x -> [x, -x]
[1,-1,2,-2,3,-3]
Prelude> [1..3] >> [4..6]
[4,5,6,4,5,6,4,5,6]
Prelude> return 1 :: [Int]
[1]
```

これだけ見てもある程度は何をしたいかという気持ちが伝わるのではないかと思う。
`>>=` に関してこれを複数個使ったもう少し複雑な例を考えてみる。

```
Prelude> [1..3] >>= \x -> [x, -x] >>= \y -> [(* 2) y]
[2,-2,4,-4,6,-6]
```

これはまず `Monad` 値 `[1..3]` があって、それを `\x -> [x, -x]` に食わせることで `[1, -1, 2, -2, 3, -3]` を作り、その `Monad` 値を `\y -> [(* 2) y]` に食わせることで結果を算出していることが分かる。
このように `Monad` を使うことで文脈を考慮した演算を繋ぎ合わせていって最終的な返り値を得ることができる、という代物になっている。
`[]` の `fail` は空のリストなので、途中の演算で `fail` が発生すると、エラーで演算が終了するのではなく、その文脈を保って空のリストを返してくれるようになっているのである。
`[]` だとちょっと有り難みが薄いかもしれないが、`Maybe` を使えば途中の演算の失敗が `Nothing` としてきちんと伝播してくれるので、煩わしい場合分けなど書かずにお行儀よく `fail` させたりすることが可能となる（が、ここではそこまで立ち入らない）。

先ほどの例をスクリプトとして書いてみると以下のようになる。
ちょっと書き方を工夫して、`Monad` 値がラムダ式の変数に束縛されているかのように書いてみる。

```
test :: [Int]
test =  [1..3] >>= \x ->
       [x, -x] >>= \y ->
       [(* 2) y]
```

これをみると、繋ぎ合わせをどんどんしていくことを考えた時にもうちょっとラクに書けると嬉しいなという気がしてくる。
そこで登場するのが `do` 記法である。

```
test :: [Int]
test = do
       x <- [1..3]
       y <- [x, -x] 
       [(* 2) y]
```

というわけで、`do` 記法は `Monad` の `>>=` に関する syntactic sugar に他ならないというものでした。
もう一つ典型的なものとしてリスト内包表記を見てみる。
例えば `>>=` を使えば以下のように 2 つのリストから要素をそれぞれ取ってきて新しいリストを作るということが可能になる。

```
Prelude> [1..3] >>= \n -> ['a', 'b'] >>= \ch -> return (n, ch)
[(1,'a'),(1,'b'),(2,'a'),(2,'b'),(3,'a'),(3,'b')]
```

とまあここまで見てきたけど、そもそもリスト内包表記でいいじゃんと思うわけです。

```
Prelude> [(n, ch) | n <- [1..3], ch <- ['a', 'b']]
[(1,'a'),(1,'b'),(2,'a'),(2,'b'),(3,'a'),(3,'b')]
```

それもそのはずで、リスト内包表記も `[] Monad` の syntactic sugar だったのさ、というオチです。
（いや、そもそも `instance` における `(>>=)` の定義としてリスト内包表記使ってたんじゃないの、という疑問が出そうだが、これはコンパイラが desugaring するという形になっている。という理解。）
なんにせよ、ここまで読めた人なら、単に「リスト内包表記は `Monad` の syntactic sugar ですよ」という statement が血肉の通わない空虚なものとしてではなく、なぜ `Monad` を使うことでこのようなことが実現できているのかを明快に説明できるようになっているはず。

ということでこれまで同様 `Monad` が満たすべき `Monad` 則を見ていくことにする。

- `return x >>= f = f x`
- `m >>= return = m`
- `(m >>= f) >>= g = m >>= (\x -> f x >>= g)`

最初の 2 つは `return` の満たすべき性質に関する要請で、`return` で持ち上げたものを `f` に渡したものとそのまま `f` に食わせたものの結果が等しいことと、`Monad` 値を `return` に渡しても変化なしというものになっている。
これは straightforward だろう。

```
Prelude> return 2 >>= (\x -> [x, -x])
[2,-2]
Prelude> (\x -> [x, -x]) 2
[2,-2]
Prelude> [2] >>= return
[2]
```

最後のものは `Monad` における結合則になっている。
右辺がちょっとややこしいかもしれないが、具体的な例で確認することは容易い。

```
Prelude> [2] >>= (\y -> [(+3) y]) >>= (\z -> [(*2) z])
[10]
Prelude> [2] >>= (\x -> ((\y -> [(+3) y]) x) >>= (\z -> [(*2) z]))
[10]
```

やや醜いがやっていることを理解するのは難しくないはず。
特に下の演算では、まず `\x -> (\y -> [(+3) y]) x` で `Num a => a -> [a]` という関数を作り、それを `>>=` で `\z -> [(*2) z]` と結合して `Num b => b -> [b]` という関数を作っている。
最終的にその関数に `Monad` 値 `[2]` を渡しているというものだ。
実に明快。

ということで抽象的な議論に戻ると、結局 `Monad` 則は `Monad` の文脈を与える最も基本的な関数として `return` の性質と結合則から成るものであり、それらを満たす限り `>>=`（すなわち `do` 記法）が使えることを保証しているのである。
そしてここまでの `Functor` 則、`Applicative` 則、`Monad` 則を見直してみると、それらはある種の基本的な関数（恒等写像など）や結合則によって構成されるという構造が見えてくる。
これらは全然別のものではなく（それは `Functor f => Applicative f => Monad f` という関係もあるので必然だが）、関数を適用するという操作において、それをそれぞれの文脈において適切（我々が使いやすいと思うようなもの）に振る舞うように規定しているものなのだと分かる。

なるほど！！！

確かに「どんなことがしたいのか」というのがめちゃくちゃ理解しやすい。
関数型言語が好きになる人の気持ちが ﾁｮｯﾄﾜｶﾙ ようになった。
本を一冊読んでちょっとコードを書いた程度の理解ではあるが、それでも気持ちが分かるぞと思えるのは心地よい。

今後は具体的に何かに使ってみて色々と評価していけたらなぁと考えたりしている。


## まとめ
Haskell に入門してみようと思って「すごいHASKELLたのしく学ぼう！」を読んだ。
なかなかに素晴らしい本で、今後も趣味的に Haskell を勉強していこうと思えるくらいに Haskell が好きになった。

特に `Functor` から `Monad` までの説明を Haskell 内に閉じた形で頑張ってやってみたが、もしご意見などあればください。
「大して難しいことやってるわけじゃないな」という感想を持ったなら成功と言えるのではないかと思う。
相当色々なものを削ったので何が強力なのかとかまで理解するのは難しいと思うけど、まあそれはやむなしということで。

ちなみに自分は今後は圏論と合わせて勉強しながら何か具体的なものに使ってみたいなぁとぼんやり考えている。
圏論とか数学の各々の具体的な対象に詳しくない自分がやっても空虚に終わってしまいそうだなと思っていたが、Haskell という具体的な対象を手に入れたことで楽しめるんじゃないかと期待している。

---
---
<br>