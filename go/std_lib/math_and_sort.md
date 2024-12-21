[[index.md|<--Go Back]]

# = Математические функуции =
Пакет `math` предоставляет большой набор математических __Ф__, которые работают с `float64` - т.е. надо явно преобразовывать тип в `float64`
__Ф__: поиск максимального и минимального значений, различные виды округления, остаток от деления, возведение в степень.
`math` предоставляет набор констант, например `MaxInt8`, `MinInt8` и т.п.

## == Случайные числа ==
`math/rand` - обеспечивает поддержку генерации _случайных_ (на самом деле __псевдослучайных__) чисел.
Пример:
```
################
###printer.go###
################

package main

import "fmt"

func Printfln(template string, values ...interface{}) {
    fmt.Printf(template + "\n", values...)
}

#############
###main.go###
#############

package main

import "math/rand"

func main() {
	for i := 0; i < 5; i++ {
		Printfln("Value %v: %v", i, rand.Int())
	}
}
```
Результат выполнения
```
Value 0: 693933934736260748
Value 1: 7360027427896559695
Value 2: 4701053459956130760
Value 3: 5081076558949267867
Value 4: 5532682413673547661
```

__!!!__ Можно генерировать значения в определенном диапазоне с помощью `Intn`

# = Сортировка =
Пакет `sort` обеспечивает сортировку, например `sort.Float64s` сортирует (__на месте__) срез значений `float64`, `sort.Float64sAreSorted` возвращает `true` если срез отстортирован
__Ф__ `sort.SearchFloat64s` ищет значение в отсортированном срезе - выдаст индекс значения или индекс по которому значение может быть вставлено без потери сортировки
Можно сортировать пользовательские __типы__ данных - для этого определен __И__ `sort.Interface` - надо опеределить _три_ метода: `Len`, `Less`, `Swap`. Таким образом можно делать различные сортивовки пользовательских __типов__