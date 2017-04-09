# <img width="80" height="50" src="./logo/logo.png" /> Static Land

Спецификация для общих алгебраических типов в JavaScript на основе [Fantasy Land](https://github.com/devSchacht/fantasy-land).

* [Спецификация](docs/spec.md)

### Отличие от Fantasy Land

Fantasy Land использует методы как основу для типов. Экземпляр типа в Fantasy Land
это объект с определенными методами. Например, экземпляр типа Functor должен быть объектом, у которого есть метод `map`.

В Static Land тип — просто набор статических функций и экземпляром типа может быть любое значение, в том числе примитивы (Number, Boolean, и т.д.)

Например, мы можем реализовать тип Addition, который используечт числа как экземпляры и удовлетворяет закону моноид:

```js
const Addition = {

  empty() {
    return 0
  },

  concat(a, b) {
    return a + b
  },

}
```

#### Плюсы

  - Нет конфликтов имен. Поскольку тип — это просто набор функций, которые не имеют общих имен, у нас нет проблем с совпадением имен.
  - Мы можем реализовать много типов для того же значения. Например, мы можем реализовать два Monoid'a для чисел: Addition и Multiplication.
  - Мы можем рализовать тип со значением в виде примитива (Number, Boolean, и т.д.).
  - Мы можем рализовать бесшовный тип. Например, мы можем реализовать тип со значением в виде массива и пользователю не надо будет оборачивать/разворачивать значения в классы обёртки с методавми Fantasy Land.

#### Минусы

  - Нам прийдётся пробрасывать типы чаще. В Fantasy Land какой-то обычный код может быть написан с использованием только методов, мы должны прокинуть только `of` и `empty`. В Static Land мы должны прокидывать типы для любого кода.

### Как добавить совместимость с Static Land для вашей библиотеки

Просто раскройте некоторые [Типы](docs/spec.md#type), которые работают с типами, которые предоставляет ваша библиотека или с типами объявленными в другой библиотеке или с нативными типам, такими как Array.

Тип не должны быть простыми объектами JavaScript; они также могут быть конструкторами при желании. Единственным требованием является:

- этот объект содержит несколько статических методов из Static Land; и
- если он содержит метод с именем как в Static Land, то этот метод должен быть методом Static Land (подчиняясь закону и т.д.).

#### Пример 1. Тип Static Land для Array

```js
const SArray = {

  of(x) {
    return [x]
  },

  map(fn, arr) {
    return arr.map(fn)
  },

  chain(fn, arr) {
    // ...
  },

}

export {SArray}
```

#### Пример 2. Тип Static Land как Class

```js
class MyType = {

  constructor() {
    // ...
  }

  someInstanceMethod() {
    // ...
  }

  static someNonStaticLandStaticMethod() {
    // ...
  }


  // Static Land methods

  static of(x) {
    // ...
  }

  static map(fn, value) {
    // ...
  }

}

export {MyType}
```

### Совместимые библиотеки

У нас есть список в вики. Не стесняйтесь добавлять туда свою библиотеку.

- [Совместимые библиотеки](https://github.com/rpominov/static-land/wiki/Compatible-libraries)


## Утилиты [![Build Status](https://travis-ci.org/rpominov/static-land.svg?branch=master)](https://travis-ci.org/rpominov/static-land) [![Coverage Status](https://coveralls.io/repos/github/rpominov/static-land/badge.svg?branch=master)](https://coveralls.io/github/rpominov/static-land?branch=master)

У нас также есть пакет `static-land` в npm, который предоставляет некоторые полезные утилиты (не много на данный момент).

* [Справочник по API](docs/API.md)

### Установка

```console
npm install static-land
```

```js
// современный JavaScript
import {fromFLType} from 'static-land'

// классический JavaScript
var fromFLType = require('static-land').fromFLType
```

Или используя CDN:

```html
<script src="https://unpkg.com/static-land/umd/staticLand.js"></script>
<script>
  var fromFLType = window.StaticLand.fromFLType
</script>
```

### Разработка

```console
npm run lobot -- --help
```

Выполняйте команды [lobot](https://github.com/rpominov/lobot) так `npm run lobot -- args...`.
