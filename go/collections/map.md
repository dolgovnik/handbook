[[collections.md|<--Go Back]]

# = Map =
__Карта__ - это встроенная структура данных, которая связывает _значения данных_ с _ключами_
Пример:
```
package main

import "fmt"

func main() {
	products := make(map[string]float64, 10)

	products["Kayak"] = 279
	products["Lifejacket"] = 48.95

	fmt.Println("Map size:", len(products))
	fmt.Println("Price", products["Kayak"])
	fmt.Println("Price", products["Hat"])
}
```
Результат выполнения:
```
Map size: 2
Price 279
Price 0
```

## == Способы определения карт ==
- __Встроенная функция__ `make`([[../sensitive_shit.md## == MAKE ==|Больше информации о make]]):
```
products := make(map[string]float64, 10)
```
_ключ_ типа `string`, _данные_ типа `float64`, начальный размер `10`
__!!!__ Размер __Карты__ изменяется автоматически, поэтому его можно не указывать при создании 
__!!!__ Если __карта__ не содержит _ключа_, то тогда возворащается __нулевое значение__ _данных_
__!!!__ `len` - возвращает кол-во _ключей_ в __карте__

- __Литеральный синтаксис__
```
product := map[string]float64 {
    "Kayak": 279,
	"Lifejacket": 48.95,
}
```
__!!!__ После _значения_ должна быть либо запятая либо фигурная скобка.
__!!!__ _ Ключи_ в литеральном синтаксисе должны быть __уникальными__

## == Проверка ключей в карте ==
Если _ключ_ есть в карте, но ему соответствуют нулевое _значение_, то это сложно отличить от случая если _ключа_ совсем нет - потому что в обоих случаях __карта__ вернет нулевое значение
__!!!__ на самом деле, __карты__ возвращают два значения:
```
product := map[string]float64 {
    "Kayak": 279,
	"Lifejacket": 48.95,
	"Hat": 0,
}

value, ok := products["Hat"]

if (ok) {
....
} else {
....
}
```

`ok` == __true__ если _ключ_ существует и __false__ если не существует

## == Удаление ключей из карты ==
Есть втроенная функция `delete`
```
delete(products, "Hat")
```

## == Перечисление содержимого карты ==
- __for range__:
__!!!__ Ключи перечисляются в произвольном порядке
```
for key, value := range products {
    fmt.Println("Key:", key, "Value:" value)
}
```

- с использованием __среза__ из _ключей_
__!!!__ Ключи перечисляются в определенном порядке
```
package main

import (
    "fmt"
	"sort"
)

func main() {
	products := map[string]float64 {
		"Kayak": 279,
		"Lifejacket": 48.95,
		"Hat": 0,
    }
	keys := make([]string, 0, len(products))
	for key, _ := range products {
		keys = append(keys, key)
	}
	sort.Strings(keys)
	for _, key := range keys {
		fmt.Println("Key:", key, "Value:", products[key])
	}
}
```
