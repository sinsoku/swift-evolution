# 付属型を許容するために `typealias` キーワードを associatedtype に置き換える

* 提案: [SE-0011](https://github.com/apple/swift-evolution/blob/master/proposals/0011-replace-typealias-associated.md)
* 提案者: [Loïc Lecrenier](https://github.com/loiclec)
* 状況: **Swift 2.2 承認済み** ([Bug](https://bugs.swift.org/browse/SR-511))
* レビュー責任者: [Doug Gregor](https://github.com/DougGregor)

## 背景

`typealias` キーワードは今のところ2種類の型の宣言に使われています:

1. タイプエイリアス (既存の型の別名)
2. 付属型 (プロトコルの一部として使われる型のプレースホルダー)

これらの2つの宣言は異なり、個別のキーワードを使うべきです。これにより違いが明確になり、付属型の使用に関するいくつかの混乱を軽減するだろう。

提案された新しいキーワードは `associatedtype` です。

## Motivation

Re-using `typealias` for associated type declarations is confusing in many ways.

1. It is not obvious that `typealias` in protocols means something else than in
 other places.
2. It hides the existence of associated types to beginners, which allows them
 to write code they misunderstand.
3. It is not clear that concrete type aliases are forbidden inside protocols.

In particular, **2 + 3** leads to programmers writing

```swift
protocol Prot {
    typealias Container : SequenceType
    typealias Element = Container.Generator.Element
}
```

without realizing that `Element` is a new associated type with a default value
of `Container.Generator.Element` instead of a type alias to
`Container.Generator.Element`.

However, this code

```swift
protocol Prot {
    typealias Container : SequenceType
}
extension Prot {
    typealias Element = Container.Generator.Element
}
```

declares `Element` as a type alias to `Container.Generator.Element`.

These subtleties of the language currently require careful consideration to
understand.

## Proposed solution

For declaring associated types, replace the `typealias` keyword with `associatedtype`.

This solves the issues mentioned above:

1. `typealias` can now only be used for type aliases declaration.
2. Beginners are now forced to learn about associated types when creating protocols.
3. An error message can now be displayed when someone tries to create a type alias
inside a protocol.

This eliminates the confusion showed in the previous code snippets.

```swift
protocol Prot {
    associatedtype Container : SequenceType
    typealias Element = Container.Generator.Element // error: cannot declare type alias inside protocol, use protocol extension instead
}
```

```swift
protocol Prot {
    associatedtype Container : SequenceType
}
extension Prot {
    typealias Element = Container.Generator.Element
}
```

Alternative keywords considered: `type`, `associated`, `requiredtype`, `placeholdertype`, …

## Proposed Approach

For declaring associated types, I suggest adding `associatedtype` and deprecating 
`typealias` in Swift 2.2, and removing `typealias` entirely in Swift 3.

## Impact on existing code

As it simply replaces one keyword for another, the transition to `associatedtype`
could be easily automated without any risk of breaking existing code.

## Mailing List

- [Original](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151130/000470.html)
- [Alternative Keywords](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151214/003551.html)
