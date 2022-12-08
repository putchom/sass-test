Sass のテストを書く

こちらは [CSS Advent Calendar 2022](https://qiita.com/advent-calendar/2022/cascading_style_sheets) 22 日目の記事です。

こんにちは、[家計簿プリカ B/43](https://b43.jp/)を運営する[株式会社スマートバンク](https://smartbank.co.jp/)でデザイナーをしている putchom です。

前職は GMO ペパボ株式会社というところで [Inhouse](https://design.pepabo.com/inhouse/) という共通基盤デザインシステムのコンポーネントライブラリの設計をしており、わりと複雑な Sass の関数を書いていました。

そのような場合、テストコードを書いて手動確認の工数を減らしたり、何か変更が合った場合にバグを検知したくなります。

そんなときに便利なパッケージが @gyugyu さんの [sassunit](https://github.com/gyugyu/sassunit) です。

# 基本的な使い方

公式の [README](https://github.com/gyugyu/sassunit/blob/master/README.md) にあるとおりです。

例えば以下ような単純な 2 つの引数を足し合わせる Sass の関数が合った場合、

```sass
@function sum($a, $b) {
  @return $a + $b;
}
```

このようなテストコードを書いて

```sass
@use 'functions';
@use '@gyugyu/assert-sass' as assert;

@function test-sum() {
  @return assert.equals(
    functions.sum(2, 3),
    5
  );
}
```

`$ npx sassunit` を実行するとテストが成功していることがわかります。

```bash
$ npx sassunit

> sass-test@1.0.0 test
> sassunit

.
passed: 1
failed: 0
```

反対にわざと落ちるコードに修正してみます。

```sass
@use 'functions';
@use '@gyugyu/assert-sass' as assert;

@function test-sum() {
  @return assert.equals(
    functions.sum(2, 3),
    10
  );
}
```

するとこのようになるはずです。

```bash
$ npx sassunit

> sass-test@1.0.0 test
> sassunit

x
passed: 0
failed: 1
src/avatar/_functions.test.scss:
": assert-equal failed. expected: 5 actual: 10"
  ╷
5 │     @return assert.equals(
  │ ┌───────────^
6 │ │     functions.sum($a: 2, $b: 3),
7 │ │     10
8 │ │   );
  │ └───^
  ╵
  src/avatar/_functions.test.scss 5:11  test-sum()
  stdin 12:14                           root stylesheet
```

すごくわかりやすいですね！

# どのようなときに役に立つか？

## 輝度のような複雑な計算をする場合

## デザイントークンを別パッケージから取得する場合

# 気をつけるポイント

## Sass の型を意識する
