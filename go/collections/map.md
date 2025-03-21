[[collections.md|<--Go Back]]
_
# = Map =_
Хеш-таблица - неупорядоченная коллекция пар _ключ-значение_: все _ключи_ различны, а _значение_ можно обновить за_ константное время. 
__Отображение__ - ссылка на хеш-таблицу, в которой все _ключи_ имеют один и тот же __тип__, и _значения_ тоже имеют один и тот же __тип__.
__Тип__ _ключа_ должен быть _сравнимым_ с помощью оператора `==`.
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
	fmt.Println("Price", )
}
```
Результат выполнения:
```
Map size: 2
Price 279
Price 0
```

_Значение_ - это НЕ переменная, поэтому нельзя получить её адрес, т.е. вот так делать НЕЛЬЗЯ:
```
_ = &products["Hat"] //<-- Невозможно получить адрес в отображении
```
Это происходит потому что при росте __отображения__ может происзодить повторное хеширование, следовательно адреса станут не действительными.


Нельзя присваивать _значение_ _ключам_ в _нулевом_ __отображении__:
```
var ages map[string]int
fmt.Println(ages == nil)     //<-- true
fmt.Println(len(ages) == 0)  //<-- true
ages["carol"] = 21           //<-- Авария!!!
```

__!!!__ _Ключи_ __отображения__ можно использовать как `set`, так как они уникальны

## == Способы определения отображений ==
- __Встроенная функция__ `make`([[../sensitive_shit.md## == MAKE ==|Больше информации о make]]):
```
products := make(map[string]float64, 10)
```
_ключ_ типа `string`, _данные_ типа `float64`, начальный размер `10`
__!!!__ Размер __Отображения__ изменяется автоматически, поэтому его можно не указывать при создании 
__!!!__ Если __отображение__ не содержит _ключа_, то тогда возворащается __нулевое значение__ _данных_
__!!!__ `len` - возвращает кол-во _ключей_ в __отображении__

- __Литеральный синтаксис__
```
product := map[string]float64 {
    "Kayak": 279,
	"Lifejacket": 48.95,
}
```
__!!!__ После _значения_ должна быть либо запятая либо фигурная скобка.
__!!!__ _ Ключи_ в литеральном синтаксисе должны быть __уникальными__

## == Проверка ключей в отображении ==
Если _ключ_ есть в __отображении__, но ему соответствуют нулевое _значение_, то это сложно отличить от случая если _ключа_ совсем нет - потому что в обоих случаях __отображение__ вернет нулевое значение
__!!!__ на самом деле, __отображения__ возвращают два значения:
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

## == Удаление ключей из отображения ==
Есть втроенная функция `delete`
```
delete(products, "Hat")
```

## == Перечисление содержимого отображения ==
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

## == Сравнение отображений ==
__Отображения__ нельзя сравнивать друг с другом. Единственное что можно - сравнение со значением `nil`
Если надо сравнить, тогда придется это делать через цикл.
