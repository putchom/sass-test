こちらは [CSS Advent Calendar 2022](https://qiita.com/advent-calendar/2022/cascading_style_sheets) 22 日目の記事です。

---

こんにちは、[家計簿プリカ B/43](https://b43.jp/)を運営する[株式会社スマートバンク](https://smartbank.co.jp/)でデザイナーをしている putchom です。

CSS Advent Calendar ではありますが、今回は **Sass** のテストコードを書くと便利という話をします。

普段私はデザインシステムを設計する仕事をしています。

家計簿プリカ B/43 は iOS・Android アプリでプロダクトを提供していますが、サービスをプロモーションするページ（ https://b43.jp/ ）の Web アプリケーション向けには、ブランドを反映したコンポーネントライブラリを用意する必要があり、そのためにわりと複雑な Sass の関数を書くことがあります。

そのような場合、より正確なロジックを組み立てたり、何か実装に変更があった場合にバグを検知したくなります。

そんなときに便利なパッケージが [assert-sass](https://github.com/gyugyu/assert-sass) 及び [sassunit](https://github.com/gyugyu/sassunit) です。

assert-sass は Dart Sass 向けの値の一致などを検証してくれるアサーション関数のライブラリ、sassunit は Dart Sass 向けの単体テストフレームワークです。

# 基本的な使い方

公式の [README](https://github.com/gyugyu/sassunit/blob/master/README.md) にあるとおりです。

例えば以下ような 2 つの数を足し合わせて返す Sass の関数があったとします。

```scss
// _functions.scss

@function sum($a, $b) {
  @return $a + $b;
}
```

assert-sass を使うと以下のような 2 + 3 は 5 になるべきという旨のテストコードを書いてこの関数の正しさを検証することができます。

```scss
// _functions.test.scss

@use "functions";
@use "@gyugyu/assert-sass" as assert;

@function test-sum() {
  @return assert.equals(functions.sum(2, 3), 5);
}
```

`assert.equals($expected, $actual)` は `$expected` に渡した値と `$actual` に渡した値が一致する場合にテストが成功します。

`npx sassunit` を叩くと、テストを実行することができます。

```bash
$ npx sassunit

> sass-test@1.0.0 test
> sassunit

.
passed: 1
failed: 0
```

`passed: 1` となっているのでテストが成功しています。

今度はわざと失敗するように、 2 + 3 の計算結果が 10 になるべきといった旨のテストコードに変更してみます。

```scss
// _functions.test.scss

@use "functions";
@use "@gyugyu/assert-sass" as assert;

@function test-sum() {
  @return assert.equals(functions.sum(2, 3), 10);
}
```

`npx sassunit` を実行するとこのようになるはずです。

```bash
$ npx sassunit

> sass-test@1.0.0 test
> sassunit

x
passed: 0
failed: 1
src/example/_functions.test.scss:
": assert-equal failed. expected: 5 actual: 10"
  ╷
5 │     @return assert.equals(
  │ ┌───────────^
6 │ │     functions.sum($a: 2, $b: 3),
7 │ │     10
8 │ │   );
  │ └───^
  ╵
  src/example/_functions.test.scss 5:11  test-sum()
  stdin 12:14                           root stylesheet
```

`: assert-equal failed. expected: 5 actual: 10` とあるので、本当は 5 にならなければならないが、10 にしたことによってテストが失敗していることがわかります。

普段からテストコードを書いている方にはお馴染みだと思いますし、これが Sass でできることにもメリットが感じられるかもしれません。

しかし、「わりと Sass は書いていて、『テストコード』という名前は聞いたことあるけど、 いまいちテストコードを書くメリットがわからないな...」という方のために、どのように Sass でテストコードを書いていくのか、どのようなメリットがあるのか具体例を交えてご紹介します。

# どのように書いていくのか？

今回は Sass で Hex color であるか否かを判定する関数を用意したいとします。

Hex color とはお馴染みの `#a9d3dc` みたいな 16 進数で表現されたカラーコードです。

まず Hex color は先頭の文字が `#` になるはずなのでそれをテストするコードを書きます。

```scss
// _functions.test.scss

@function test-is-hex-color() {
  // 先頭が # なら Hex color である
  @return assert.equals(functions.is-hex-color("#a9d3dc"), true);
}

@function test-is-not-hex-color() {
  // 先頭が # でないなら Hex color ではない
  @return assert.equals(functions.is-hex-color("a9d3dc"), false);
}
```

そして実際に関数を書いていきます。

まずは、どんな値を渡しても true を返す関数を用意します。

```scss
// _functions.scss
@function is-hex-color($value) {
  @return true;
}
```

絶対に true を返すので、false が返ってくることを予期している `test-is-not-hex-color()` のテストは失敗します。

```bash
$ npx sassunit

.x
passed: 1
failed: 1
src/example/_functions.test.scss:
": assert-equal failed. expected: true actual: false"
   ╷
14 │     @return assert.equals(
   │ ┌───────────^
15 │ │     functions.is-hex-color('a9d3dc'),
16 │ │     false
17 │ │   );
   │ └───^
   ╵
  src/example/_functions.test.scss 14:11  test-is-not-hex-color()
  stdin 12:14                             root stylesheet
```

失敗してはいますが、現時点ではこれは失敗するのが正しく、このテストが成功するようになれば要件を満たすような関数になっていると言えそうです。

なので、実際にテストが成功するように関数を修正していきます。

```scss
// _functions.scss

@use "sass:string";

@function is-hex-color($value) {
  @if string.slice("#{$value}", 1, 1) != "#" {
    @return false;
  }

  @return true;
}
```

sass:string の slice 関数を使うと、文字列の一部を取り出すことができます。

- [参考: Sass: sass:string](https://sass-lang.com/documentation/modules/string#slice)

1 文字目を取り出し、`#` に合致しない場合は false、合致する場合は true を返すように関数を修正しました。

無事にすべてのテストが成功し、先頭の `#` の有無によって Hex color かどうか判定できるようになりました。

```bash
$ npx sassunit

..
passed: 2
failed: 0
```

しかし、「先頭が `#` であること」のみでは正確に検証できているとは言えません。

`#a9d3dc00000000000` のように桁数がべらぼうに多い場合も Hex color ではないと判定されてほしいです。

これを検証するテストコードを書いてみます。

```scss
// _functions.test.scss

@function test-is-not-hex-color-when-too-many-digits() {
  // 桁数がべらぼうに多い場合は Hex color ではない
  @return assert.equals(functions.is-hex-color("#a9d3dc00000000000000"), false);
}
```

しかし、このテストは `is-hex-color()` 関数から true が返ってきて（#a9d3dc00000000000000 は Hex color であると判定されて）失敗してしまいます。

```bash
$ npx sassunit

..x
passed: 2
failed: 1
src/example/_functions.test.scss:
": assert-equal failed. expected: true actual: false"
   ╷
19 │     @return assert.equals(
   │ ┌───────────^
20 │ │     functions.is-hex-color('#a9d3dc00000000000000'),
21 │ │     false
22 │ │   );
   │ └───^
   ╵
  src/example/_functions.test.scss 19:11  test-is-not-hex-color()
  stdin 12:14                             root stylesheet
```

この関数はまだ Hex color を上手に判定できていなそうなので、規定の桁数ではない場合は false を返すように関数を修正します。

```scss
// _functions.scss

@function is-hex-color($value) {
  $length: string.length("#{$value}");

  // 先頭の文字列が # かどうかチェックする
  @if string.slice("#{$value}", 1, 1) != "#" {
    @return false;
  }

  // 文字数が # を含めて正しい桁数になっているかをチェックする
  @if $length != 4 and $length != 7 and $length != 9 {
    @return false;
  }

  @return true;
}
```

sass:string の `length($string)` を用いると `$string` の文字数をカウントできます。

- [参考: Sass: sass:string](https://sass-lang.com/documentation/modules/string#length)

今回 Hex color は `#000` のような three-value syntax、 `#000000` のような six-value syntax、または透過を含めた `#000000000` のような eight-value syntax の場合を考慮することにし、それぞれに `#` を含めた桁数に `$length` が一致しなかった場合は false を返すようにしました。

```bash
$ npx sassunit

...
passed: 3
failed: 0
```

これで、テストが無事成功するようになりました。

ちなみに桁数が少ない場合もテストしてみます。

```scss
// _functions.test.scss

@function test-is-not-hex-color-when-few-digits() {
  // 桁数が少ない場合は Hex color ではない
  @return assert.equals(functions.is-hex-color("#a9"), false);
}
```

```bash
$ npx sassunit

....
passed: 4
failed: 0
```

これも無事にテストが成功しました。

これで桁数の問題はクリアできていそうです。しかし、まだこの関数には問題があります。

Hex color で有効な文字列ではない文字列を渡した場合にも Hex color ではないと判断されてほしいです。

`#ああああああ` という日本語のかなを含む文字列を渡して、Hex color ではないことを検証してみます。

```scss
// _functions.test.scss

@function test-is-not-hex-color-when-japanese-chars() {
  // Hex colorとして有効な文字でないものが含まれる場合はHex colorではない
  @return assert.equals(functions.is-hex-color("#ああああああ"), false);
}
```

このテストは `is-hex-color()` 関数から true が返ってきて（#ああああああは Hex color であると判定されて）失敗してしまいました。

```bash
$ npx sassunit

...x
passed: 3
failed: 1
src/example/_functions.test.scss:
": assert-equal failed. expected: true actual: false"
   ╷
26 │     @return assert.equals(
   │ ┌───────────^
27 │ │     functions.is-hex-color('#ああああああ'),
28 │ │     false
29 │ │   );
   │ └───^
   ╵
  src/example/_functions.test.scss 26:11  test-is-not-hex-color-when-japanese-chars()
  stdin 12:14                             root stylesheet
```

そこで、渡した文字列が Hex color で扱える文字に合致する場合のみ Hex color であると判定されるように関数を修正します。

```scss
// _functions.scss

@use "sass:string";
@use "sass:list";

@function is-hex-color($value) {
  $length: string.length("#{$value}");
  $available-hex-chars: "0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "a",
    "b", "c", "d", "e", "f", "A", "B", "C", "D", "E", "F";

  // 先頭の文字列が # かどうかチェックする
  @if string.slice("#{$value}", 1, 1) != "#" {
    @return false;
  }

  // 文字数が # を含めて正しい桁数になっているかをチェックする
  @if $length != 4 and $length != 7 and $length != 9 {
    @return false;
  }

  // Hex color で利用できる文字で構成されているかチェックする
  @for $i from 2 through $length {
    $char: string.slice("#{$value}", $i, $i);

    @if list.index($available-hex-chars, $char) == null {
      @return false;
    }
  }

  @return true;
}
```

渡した値の 2 文字目（ # 以降）から文字数分の文字を取り出し、Hex color で扱える文字に合致するか検証していきます。

ちなみに Hex color で扱える文字は数字の 0〜9、英字の a〜f、A〜F です。

sass:list の `list.index($list, $value)` を使うと `$list` に渡した配列内に `$value` で渡した値が存在すればその値の位置の番号を取得できます。しかし、配列内に合致する値が存在しない場合は null を返します。

- [参考: Sass: Lists](https://sass-lang.com/documentation/values/lists#find-an-element-in-a-list)

そのため、 `$available-hex-chars` の配列に対して値から取り出した文字列を照合していき、番号ではなく null が返ってきた場合は Hex color で扱える文字ではないと判定できます。

もう一度テストを実行してみます。

```bash
$ npx sassunit

.....
passed: 5
failed: 0
```

無事すべてのテストが成功しました 🎉

このように「この場合は正しく動かないかもしれない」という予想をもとに用意した値を渡してテストコードを書き、そのあとにそのテストが成功するように関数を組み立てるといったサイクルを回していくことで、より強度が高い関数を書いていくことができます。

これは人間が手動でチェックするよりも実装の漏れを防ぐことができ、たとえ今後関数の実装が変更されたときも、再度テストを実行することで、壊れたりしていないかを自動でチェックすることができます。

# まとめ

今回は `assert.equals($expected, $actual)` のみ紹介しましたが、assert-sass には他にも null ではないことをチェックする `assert.not-null($value)` や、配列で指定した key が指定した map の key にすべて含まれているかどうかチェックする `assert.contains-all-keys($map, $keys)` などが用意されているので、うまく組み合わせてテストコードを書いていくと便利です。

今回書いたコードは以下の GitHub リポジトリで公開していますのでご参考まで。

- https://github.com/putchom/sass-test

# さいごに

私が働く家計簿プリカ B/43 を運営する株式会社スマートバンクでは各方面で積極採用活動中です。

![recruting](https://user-images.githubusercontent.com/945841/195980635-fff25821-89d3-495b-8384-6e34de052dbb.jpg)

特にデザイナーの中では現在はコミュニケーションデザイン領域のデザイナーを募集中です。

- [コミュニケーションデザイナー - 株式会社スマートバンク | SmartBank, Inc.](https://smartbank.co.jp/163ba6c98b01465a9ded6b3ebe8088b9)

選考とは別に、「気になるけど選考に進むか悩んでいる」という方向けに会社の雰囲気や気になる職種の業務内容をお話しする「カジュアル面談」も用意しているので、ぜひお気軽にご連絡ください。

- [カジュアル面談への応募 - 株式会社スマートバンク | SmartBank, Inc.](https://smartbank.co.jp/de1f8d3093a64bcd99e2487352cf5f56)
