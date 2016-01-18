# 外部引数名として(大部分の)キーワードを許容する

* 提案: [SE-0001](https://github.com/apple/swift-evolution/blob/master/proposals/0001-keywords-as-argument-labels.md)
* 提案者: [Doug Gregor](https://github.com/DougGregor)
* 状況: **承認済み**, [Swift 2.2 実装済み](https://github.com/apple/swift/commit/c8dd8d066132683aa32c2a5740b291d057937367) ([Bug](https://bugs.swift.org/browse/SR-344))

## 背景

外部引数名は Swift の関数のインターフェースで重要な役割をもち、関数の個々の引数の特徴を表現し、可読性を向上させます。ときには、引数に最も自然なラベルが `in`, `repeat`, `defer` のように予約語と一致します。このようなキーワードはより良いインターフェースの表現を可能にする外部引数名として許可されるべきです。

## 動機

いくつかの関数で、個々の引数の最も良い外部引数名が予約語と一致することが起きます。例えば、配列の特定の値を見つけるモジュールスコープの関数を考える。これの自然な名前は `indexOf(_:in:)` になるだろう:

	indexOf(value, in: collection)

しかしながら、 `in` はキーワードなので、実際には `in` をエスケープのためにバッククウォートを使う必要があるだろう。例:

	indexOf(value, `in`: collection)

Swift に新しいAPIを定義する際、開発者はキーワードではない単語(例: `within` for this example)を選びがちであり、それは理想的ではありません。一方で、この問題は the "omit needless words" heuristics に従って Objective-C API をインポートするときにも発生し、これらのAPIを使うためにエスケープが必要です。例えば:

	event.touchesMatching([.Began, .Moved], `in`: view)
	NSXPCInterface(`protocol`: SomeProtocolType.Protocol)


## 提案された解決方法

Allow the use of all keywords except `inout`, `var`, and `let` as argument labels. This affects the grammar in three places:

* Call expressions, such as the examples above. Here, we have no grammatic ambiguities, because "<keyword> \`:\`" does not appear in any grammar production within a parenthesized expression list. This is, by far, the most important case.

* Function/subscript/initializer declarations: aside from the three exclusions above, there is no ambiguity here because the keyword will always be followed by an identifier, ‘:’, or ‘_’. For example:

```swift
func touchesMatching(phase: NSTouchPhase, in view: NSView?) -> Set<NSTouch>
```

  Keywords that introduce or modify a parameter—-currently just
"inout", "let", and "var"—-will need to retain their former
meanings. If we invent an API that uses such keywords, they will still
need to be back-ticked:

```swift
func addParameter(name: String, `inout`: Bool)
```

* Function types: these are actually easier than #2, because the parameter name is always followed by a ‘:’:

```swift
(NSTouchPhase, in: NSView?) -> Set<NSTouch>
(String, inout: Bool) -> Void
```

## 既存コードへの影響

This functionality is strictly additive, and does not break any existing
code: it only makes some previously ill-formed code well-formed, and
does not change the behavior of any well-formed code.

## 検討された代案

The primarily alternative here is to do nothing: Swift APIs will
continue to avoid keywords for argument labels, even when they are the
most natural word for the label, and imported APIs will either
continue to use backticks or will need to be renamed. This alternative
leaves a large number of imported APIs (nearly 200) requiring either
some level of renaming of the API or backticks at the call site.

A second alternative is to focus on `in` itself, which is by far the
most common keyword argument in imported APIs. In a brief survey of
imported APIs, `in` accounted for 90% of the conflicts with existing
keywords. Moreover, the keyword `in` is only used in two places in the
Swift grammar--for loops and closures--so it could be made
context-sensitive. However, this solution is somewhat more complicated
(because it requires more context-sensitive keyword parsing) and less
general.

