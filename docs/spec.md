# Спецификация Static Land

Это спецификация для общих алгебраических типов в JavaScript на основе [спецификации Fantasy Land](https://github.com/devSchacht/fantasy-land).


## Тип

Тип в Static Land это словарь (JavaScript объект) со статическими функциями в качестве значений. 'Статический' означает что функции не используют `this`, они могут быть убраны у объекта типа. Объект типа просто контейнер для функций.

```js
const {of, map} = MyType

// Должно работать
map(x => x + 1, of(41)) // MyType(42)
```

Функции из объекта типа часто называются "методами" типа. Но запомните что они не методы в JS представлении (они не используют `this`).


## Описание типа

Каждый метод в этой спецификации имеет описание типа, оно выглядит так.

```
map :: Functor f => Type f ~> (a → b, f a) → f b
```

Мы используем синтаксис похожий на Хаскель. Вы можете узнать о нём отсюда [Ramda's wiki](https://github.com/ramda/ramda/wiki/Type-Signatures) или из книги ["Professor Frisby's Mostly Adequate Guide to Functional Programming"](https://drboolean.gitbooks.io/mostly-adequate-guide/content/ch7.html).

Эта спецификация использует расширения для синтаксиса описания типов:

  1. `(a, b) → c` обозначает булеву функцию, которая не каррирована. Тоже самое для большего числа аргументов.
  2. `Type a` обозначает [типизированный словарь](#Тип) типа `a`.
     Например, функция с описанием `(Type f, f a) → f a` может быть вызвана так `fn(F, F.of(1))`.
  3. `~>` обозначает доступ к свойствам JavaScript объекта.
     Например, `fn :: Type f ~> (f a) → f a` может быть применена так `F.fn(F.of(1))`.

Если метод вызывается с неправильными типами, поведение неопределено. Также, если метод принимает функцию, он должен применять функцию только в соответствии с описанием типа, т.е. обеспечивать правильное количество аргументов и правильные типы.


## Параметризация

Все реализации методов должны использовать только информацию о типе аргумента, изветсную из описания типа метода. не разрешено проверять аргументы или значения, которые они возвращают или содержат для получения большей информации о их типах. Другими словами методы должны быть [параматрически полиморфны](https://ru.wikipedia.org/wiki/Параметрический_полиморфизм).

Например, давайте рассмотрим описание метода функтора `map`:

```
map :: Functor f => Type f ~> (a → b, f a) → f b
```

В нём есть три типа переменных: `f`, `a`, and `b`. Также у нас есть некоторые ограничения:

  1. `Functor f` говорит что `f a` — значение Functor.
  2. `Type f ~>` означает, что мы пишем реализацию `map` для [типизированного словаря](#Тип) `Type f`. В этом случае мы знаем, что определенный тип `Type f` работает с методом, поэтому мы знаем все о `f`.

У нас нет никаких ограничений для типов `a` и `b`, так что мы ничего не знаем о них. И нам нельзя проверять их.

Вот реализация Maybe, которая нарушает требование параметризации, хотя вписывается в описание типа: 

```js
Maybe.Nothing = {type: 'Nothing'}

Maybe.of = x => {
  if (x === undefined) { // проверка неразрешена
    return Maybe.Nothing
  }
  return {type: 'Just', value: x}
}

Maybe.map = (f, v) => {

  // это законная проверка `f a`, потому что мызнаем структуру `f`
  if (v.type === 'Nothing') {
    return v
  }

  const a = v.value
  const b = f(a)

  if (b === undefined) { // проверка неразрешена
    return Maybe.Nothing
  }

  return Maybe.of(b)
}
```


## Эквивалентность

Эквивалентности для данного значения должна соответствоавать определению, что два значения могут быть безопасно поменяны местами в программе, что подтверждает абстракцию.

Например:

 - Два списка эквивалентны, если они эквивалентны по всем показателям.
 - Два обычных JavaScript объекта, представленные как словари, эквивалентны, когда они эквивалентны по всем ключам.
 - Два промиса эквивалентны, когда они возвращают эквивалентные значения.
 - Две функции эквивалентны, если они дают эквивалентные результаты для эквивалентных входных данных.

мы используем символ `≡` в правилах для обозначения эквивалентности.


## Алгебры

Алгебра представляет собой набор значений(экземпляров типа и других значений), набор операторов(методов типа), которые зависимы и должны подчиняться некоторым правилам.

Каждая алгебра — это отдельная спецификация. Алгебра может иметь зависимости от реализаций других алгебр.

Алгебра также может содержать дргие методы алгебр, которые могут быть получены из новых методов. Если тип имеет метод, который может быть получен, его поведение должно быть эквивалентно тому, из которого он получен.

* [Setoid](#setoid)
* [Semigroup](#semigroup)
* [Monoid](#monoid)
* [Functor](#functor)
* [Bifunctor](#bifunctor)
* [Contravariant](#contravariant)
* [Profunctor](#profunctor)
* [Apply](#apply)
* [Applicative](#applicative)
* [Alt](#alt)
* [Plus](#plus)
* [Alternative](#alternative)
* [Chain](#chain)
* [ChainRec](#chainrec)
* [Monad](#monad)
* [Foldable](#foldable)
* [Extend](#extend)
* [Comonad](#comonad)
* [Traversable](#traversable)



## Setoid

#### Методы

  1. `equals :: Setoid s => Type s ~> (s, s) → Boolean`

#### Правила

  1. Рефлексивность: `S.equals(a, a) === true`
  1. Симметрия: `S.equals(a, b) === S.equals(b, a)`
  1. Транзитивность: если `S.equals(a, b)` и `S.equals(b, c)`, тогда `S.equals(a, c)`



## Semigroup

#### Методы

  1. `concat :: Semigroup s => Type s ~> (s, s) → s`

#### Правила

  1. Ассоциативность: `S.concat(S.concat(a, b), c) ≡ S.concat(a, S.concat(b, c))`



## Monoid

#### Зависимости

  1. Semigroup

#### Методы

  1. `empty :: Monoid m => Type m ~> () → m`

#### Правила

  1. Точный справа: `M.concat(a, M.empty()) ≡ a`
  1. Точный слева: `M.concat(M.empty(), a) ≡ a`



## Functor

#### Методы

  1. `map :: Functor f => Type f ~> (a → b, f a) → f b`

#### Правила

  1. Точный: `F.map(x => x, a) ≡ a`
  1. Композиция: `F.map(x => f(g(x)), a) ≡ F.map(f, F.map(g, a))`



## Bifunctor

#### Зависимости

  1. Functor

#### Методы

  1. `bimap :: Bifunctor f => Type f ~> (a → b, c → d, f a c) → f b d`

#### Правила

  1. Точный: `B.bimap(x => x, x => x, a) ≡ a`
  1. Композиция: `B.bimap(x => f(g(x)), x => h(i(x)), a) ≡ B.bimap(f, h, B.bimap(g, i, a))`

#### Может быть получен

  1. Functor's map: `A.map = (f, u) => A.bimap(x => x, f, u)`



## Contravariant

#### Методы

  1. `contramap :: Contravariant f => Type f ~> (a → b, f b) → f a`

#### Правила

  1. Точный: `F.contramap(x => x, a) ≡ a`
  1. Композиция: `F.contramap(x => f(g(x)), a) ≡ F.contramap(g, F.contramap(f, a))`


## Profunctor

#### Зависимости

  1. Functor

#### Методы

  1. `promap :: Profunctor f => Type f ~> (a → b, c → d, f b c) → f a d`

#### Правила

  1. Точный: `P.promap(x => x, x => x, a) ≡ a`
  1. Композиция: `P.promap(x => f(g(x)), x => h(i(x)), a) ≡ P.promap(g, h, P.promap(f, i, a))`

#### Может быть получен

  1. Functor's map: `A.map = (f, u) => A.promap(x => x, f, u)`



## Apply

#### Зависимости

  1. Functor

#### Методы

  1. `ap :: Apply f => Type f ~> (f (a → b), f a) → f b`

#### Правила

  1. Композиция: `A.ap(A.ap(A.map(f => g => x => f(g(x)), a), u), v) ≡ A.ap(a, A.ap(u, v))`



## Applicative

#### Зависимости

  1. Apply

#### Методы

  1. `of :: Applicative f => Type f ~> a → f a`

#### Правила

  1. Точный: `A.ap(A.of(x => x), v) ≡ v`
  1. Гомоморфизм: `A.ap(A.of(f), A.of(x)) ≡ A.of(f(x))`
  1. Перестановка: `A.ap(u, A.of(y)) ≡ A.ap(A.of(f => f(y)), u)`

#### Может быть получен

  1. Functor's map: `A.map = (f, u) => A.ap(A.of(f), u)`



## Alt

#### Зависимости

  1. Functor

#### Методы

  1. `alt :: Alt f => Type f ~> (f a, f a) → f a`

#### Правила

  1. Ассоциативность: `A.alt(A.alt(a, b), c) ≡ A.alt(a, A.alt(b, c))`
  2. Распределённость: `A.map(f, A.alt(a, b)) ≡ A.alt(A.map(f, a), A.map(f, b))`



## Plus

#### Зависимости

  1. Alt

#### Методы

  1. `zero :: Plus f => Type f ~> () → f a`

#### Правила

  1. Точный справа: `P.alt(a, P.zero()) ≡ a`
  2. Точный слева: `P.alt(P.zero(), a) ≡ a`
  3. Упразднение: `P.map(f, P.zero()) ≡ P.zero()`



## Alternative

#### Зависимости

  1. Applicative
  2. Plus

#### Методы

  1. Распределённость: `A.ap(A.alt(a, b), c) ≡ A.alt(A.ap(a, c), A.ap(b, c))`
  2. Упразднение: `A.ap(A.zero(), a) ≡ A.zero()`



## Chain

#### Зависимости

  1. Apply

#### Методы

  1. `chain :: Chain m => Type m ~> (a → m b, m a) → m b`

#### Правила

  1. Ассоциативность: `M.chain(g, M.chain(f, u)) ≡ M.chain(x => M.chain(g, f(x)), u)`

#### Может быть получен

  1. Apply's ap: `A.ap = (uf, ux) => A.chain(f => A.map(f, ux), uf)`



## ChainRec

#### Зависимости

  1. Chain

#### Методы

  1. `chainRec :: ChainRec m => Type m ~> ((a → c, b → c, a) → m c, a) → m b`

#### Правила

  1. Эквивалентность: `C.chainRec((next, done, v) => p(v) ? C.map(done, d(v)) : C.map(next, n(v)), i) ≡ (function step(v) { return p(v) ? d(v) : C.chain(step, n(v)) }(i))`
  2. Использование `C.chainRec(f, i)` должно быть максимально подобным самостоятельному вызову `f`.



## Monad

#### Зависимости

  1. Applicative
  1. Chain

#### Правила

  1. Точный слева: `M.chain(f, M.of(a)) ≡ f(a)`
  1. Точный справа: `M.chain(M.of, u) ≡ u`

#### Может быть получен

  1. Functor's map: `A.map = (f, u) => A.chain(x => A.of(f(x)), u)`




## Foldable

#### Методы

  1. `reduce :: Foldable f => Type f ~> ((a, b) → a, a, f b) → a`

#### Правила

  1. `F.reduce ≡ (f, x, u) => F.reduce((acc, y) => acc.concat([y]), [], u).reduce(f, x)`



## Extend

#### Методы

  1. `extend :: Extend e => Type e ~> (e a → b, e a) → e b`

#### Правила

  1. Associativity: `E.extend(f, E.extend(g, w)) ≡ E.extend(_w => f(E.extend(g, _w)), w)`



## Comonad

#### Зависимости

  1. Functor
  1. Extend

#### Методы

  1. `extract :: Comonad c => Type c ~> c a → a`

#### Правила

  1. `C.extend(C.extract, w) ≡ w`
  1. `C.extract(C.extend(f, w)) ≡ f(w)`
  1. `C.extend(f, w) ≡ C.map(f, C.extend(x => x, w))`



## Traversable

#### Зависимости

  1. Functor
  1. Foldable

#### Методы

  1. `traverse :: (Traversable t, Applicative f) => Type t ~> (Type f, (a → f b), t a) → f (t b)`

#### Правила

  1. Нормализованность: `f(T.traverse(A, x => x, u)) ≡ T.traverse(B, f, u)` для любого `f` такого что `B.map(g, f(a)) ≡ f(A.map(g, a))`
  2. Точный: `T.traverse(F, F.of, u) ≡ F.of(u)` для любого Applicative `F`
  3. Композиция: `T.traverse(ComposeAB, x => x, u) ≡ A.map(v => T.traverse(B, x => x, v), T.traverse(A, x => x, u))` для `ComposeAB` определённого ниже и для любого Applicatives `A` и `B`

```js
const ComposeAB = {

  of(x) {
    return A.of(B.of(x))
  },

  ap(a1, a2) {
    return A.ap(A.map(b1 => b2 => B.ap(b1, b2), a1), a2)
  },

  map(f, a) {
    return A.map(b => B.map(f, b), a)
  },

}
```

#### Может быть получен

  1. Foldable's reduce:

    ```js
    F.reduce = (f, acc, u) => {
      const of = () => acc
      const map = (_, x) => x
      const ap = f
      return F.traverse({of, map, ap}, x => x, u)
    }
    ```

  2. Functor's map:

    ```js
    F.map = (f, u) => {
      const of = (x) => x
      const map = (f, a) => f(a)
      const ap = (f, a) => f(a)
      return F.traverse({of, map, ap}, f, u)
    }
    ```
