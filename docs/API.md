### `fromFLType`

`(FLType[, listOfMethodsThatShouldBeGenerated]) → TypeObject`

Полчает тип Fantasy Land — возвращает тип для Static Land.

```js
import {fromFLType} from 'static-land'
import IdFL from 'fantasy-land/id'

const Id = fromFLType(IdFL)

Id.map(x => x + 41, Id.of(1)) // IdFL(42)
```


### `runGenerator`

`(TypeObject, generator) → typeValue`

Что-то типа [вызова do в Хаскеле](https://en.wikibooks.org/wiki/Haskell/do_notation) для монад Static Land основанных на [генераторах](https://developer.mozilla.org/ru/docs/Web/JavaScript/Guide/Iterators_and_Generators).

`TypeObject` должен реализовать [ChainRec](./spec.md#chainrec) или [Chain](./spec.md#chain). Если `ChainRec` доступен, вы можете цикл с большим количеством итераций в генераторе. Функция `generator` должна `yield` и `return` значения, которые `TypeObject` использует как значения.

```js
import {runGenerator} from 'static-land'

const List = {

  of(x) {
    return [x]
  },

  chain(f, list) {
    return list.reduce((acc, input) => acc.concat(f(input)), [])
  },

}

runGenerator(List, function*() {
  const x = yield [1, 2]
  const y = yield [3, 4]
  return [x * y]
}) // -> [3, 4, 6, 8]

runGenerator(List, function*() {
  const x = yield [1, -2, 3]
  return x > 0 ? [x * 2] : []
}) // -> [2, 6]
```

Имейте в виду, что для монад, которые представляют несколько значений (например, List или Observable) функция `generator` может быть выполнена несколько раз, так что убедитесь, что она чистая. Для монад с оним и меньше значений (например, Future или Maybe) можно с уверенностью предположить, что функция `generator` выполняется только один раз.

Предупреждение: из-за того как работают генераторы в JavaScript, сложно реализовать `runGenerator()`, которая одновременно работает для любой монады и эффективна по ресурсам. Эта реализация может потреблять много памяти и ресурсов процессора, в некоторых случаях. Рекомендуется использовать её только для образовательных целей и не использовать ее в реальных программах. 
