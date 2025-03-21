ollections.md|<--Go Back]]

# = Slice =
__Cрез__ - это последовательность переменной длинны, все элементы которой имеют один и тот же __тип__
__Срез__ - структура данных, которая содержит _несколько_ значений:
    * __Указатель__ на _первый элемент_ [[array.md|массивa]], который доступен через __срез__, но это не обязательно первый элемент _базового массива_
	* __Длинну__ (__Lenght__) среза - кол-во элементов которые он может содержать в данный момент
	* __Ёмкость__ (__Capacity__) среза - это кол-во элементов, которые могут быть сохранены в базовом массиве, прежде чем размер среза должен быть изменен и создан новый массив. Как правило, это кол-во элементов между началом __среза__ и концом _базового_ __массива__
	* Начальный индекс __среза__ в __массиве__ - так как __срез__ может начинаться __НЕ__ с _нулевого_ элемента в __массиве__
__Ёмкость__ всегда НЕ МЕНЬШЕ __Длинны__

| Срез |     |   | Массив     |
|------|-----|---|------------|
| Ptr  | --> | 0 | Kayak      |
| Len  | 3   | 1 | Lifejacket |
| Cap  | 3   | 2 | Paddle     |

Нотация индекса - как у [[array.md|массива]]
| Срез |    |   | Массив       |
|------|----|---|--------------|
| 0    | -> | 0 | Kayak        |
| 1    | -> | 1 | Lifejacket   |
| 2    | -> | 2 | Paddle       |

## == Под капотом ==
Фактически, __срез__ представляет собой структуру, в которой хранится:
- __указатель__ на элемент __массива__ который является _первым_ в __срезе__
- длинна __среза__
- ёмкость __среза__
Т.е.:
```
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

Пример:
```
type sliceHeader struct {
	array unsafe.Pointer
	len int
	cap int
}

func main() {
	array := [6]int{1, 2, 3, 4, 5, 6}
	slice := array[1:3]
	fmt.Println("slice:", slice)

	header := (*sliceHeader)(unsafe.Pointer(&slice))
	fmt.Println("sliceHeader:", header)
}
[12:59:49] ~/Y/go_study/testtest 

```

Результат выполнения:
```
slice: [2 3]
sliceHeader: &{0xc00001a218 2 5}
```



## == Способы определения срезов ==
- __Встроенная функция__ `make`([[../sensitive_shit.md## == MAKE ==|Больше информации о make]]):
__Ф__ создает неименованную переменную __массива__ и возвращает его __срез__.

```
package main

import "fmt"

func main() {
	names := make([]string, 3, 6)

	names[0] = "Kayak"
	names[1] = "Lifejacket"
	names[2] = "Paddle"

	fmt.Println(names)                    // <-- ["Kayak", "Lifejacket", "Paddle"]
	fmt.Println("len:", len(names))       // <-- len: 3
	fmt.Println("cap:", cap(names))       // <-- cap: 6

}
```
`[]string` - тип среза - содержит строковые значения
`3` - __длинна__ среза - это НЕ часть типа среза, потому что длинна среза может изменяться
`6` - __ёмкость__ среза
| Срез |    |   | Массив       |
|------|----|---|--------------|
| 0    | -> | 0 | Kayak        |
| 1    | -> | 1 | Lifejacket   |
| 2    | -> | 2 | Paddle       |
|      | -> | 3 | (zero value) |
|      | -> | 2 | (zero value) |
|      | -> | 5 | (zero value  |

- __Литеральный синтаксис__
```
names := []string {"Kayak", "Lifejacket", "Paddle"}
```
Начальная длинна выводится из кол-ва переданных значений

## == Добавление элементов в срез ==
```
names := []string {"Kayak", "Lifejacket", "Paddle"}
names = append(names, "Hat", "Gloves")
```
Функция `append` применяется для добавления элементов в __срез__. Она принимает __срез__ и один или несколько элементов которые надо добавить.
Функция `append` - __создает массив__, достаточно большой для новых элементов, копирует в него существующий __массив__ и добавляет в него новые значения
Результат работы `append` - __срез__, отображаемый на новый __массив__
| Срез |    |   | Массив     |
|------|----|---|------------|
| 0    | -> | 0 | Kayak      |
| 1    | -> | 1 | Lifejacket |
| 2    | -> | 2 | Paddle     |
| 3    | -> | 3 | Hat        |
| 4    | -> | 4 | Gloves     |

Исходный __срез__ и его __базовый массив__ продолжают существовать и их можно использовать если результат `append` присвоить другой переменной

__!!!__ __Базовый массив__ не заменяется если `append` вызывается для среза с достаточной __ёмкостью__ для размещения новых элементов

```
package main

import "fmt"

func main() {
	names := make([]string, 3, 6)

	names[0] = "Kayak"
	names[1] = "Lifejacket"
	names[2] = "Paddle"
	
	appendedNames := append(names, "Hat", "Gloves")

    names[0] = "Canoe"

	fmt.Println("names:", names)                    // <-- names: [Canoe Lifejacket Paddle]
	fmt.Println("appendedNames:", appendedNames)    // <-- appendedNames: [Canoe Lifejacket Paddle Hat Gloves]
}
```
__Срезы__ `names` и `appendedNames` поддерживаются одним тем же __базовым массивом__
| names |    |   | Массив       |    | appendedNames |
|-------|----|---|--------------|----|---------------|
| 0     | -> | 0 | Canoe        | <- | 0             |
| 1     | -> | 1 | Lifejacket   | <- | 1             |
| 2     | -> | 2 | Paddle       | <- | 2             |
|       |    | 3 | Hat          | <- | 3             |
|       |    | 4 | Gloves       | <- | 4             |
|       |    | 5 | (zero value) |    |               |

__!!!__ Срезание за пределами `cap` вызывает аварию:
```
fmt.Println(names[:20]) //<-- выход за диапазон, аварийная ситуация
someMoreNames := names[:4] //<-- расширение среза в педелах диапазона
fmt.Println(someMoreNames) // "[Canoe, Lifejacket, Paddle, Hat]"
```

## == Добавление одного среза к другому ==
```
package main

import "fmt"

func main() {
	names := make([]string, 3, 6)

	names[0] = "Kayak"
	names[1] = "Lifejacket"
	names[2] = "Paddle"
	
	moreNames := []string {"Hat", "Gloves"}
	
	appendedNames := append(names, moreNames...)    // <-- '...'  - это распаковка среза

	fmt.Println("appendedNames:", appendedNames)    // <-- appendedNames: [Canoe Lifejacket Paddle Hat Gloves]
}
```
__!!!__ `...` обязательны чтобы `append` распаковал срез 


## == Срез из Массива ==
```
products := [4]string {"Kayak", "Lifejacket", "Paddle", "Hat"}
someNames := products[1:3]        // <-- ["Lifejacket", "Paddle"]
allNames := products[:]
```
__Срезы__ `someNames` и `allNames` поддерживаются одним __массивом__ `products`
| someNames |    |   | Массив     |    | allNames |
|-----------|----|---|------------|----|----------|
|           |    | 0 | Canoe      | <- | 0        |
| 0         | -> | 1 | Lifejacket | <- | 1        |
| 1         | -> | 2 | Paddle     | <- | 2        |
|           |    | 3 | Hat        | <- | 3        |

__!!!__ Первый индекс среза `someNames` указывает НЕ на первый индекс массива
```
len(someNames) = 2
cap(someNames) = 3 # !!!
len(allNames) = 4
cap(allNames) = 4
```
__!!!__ Если __добавить__ элемент в __срез__ `someNames` то он изменит __срез__ `allNames` так как они опираются на один __массив__

__!!!__ При __создании среза__ из __массива__, можно указать __ёмкость__ среза
```
products := [4]string {"Kayak", "Lifejacket", "Paddle", "Hat"}
someNames := products[1:3:3]
allNames := products[:]
```
__!!!__ __Ёмкость__ в данном случае определяется вычитанием __нижнего значения__ (1) из __максимального__ (вторая цифра 3)


## == Срез из среза == 
```
products := [4]string {"Kayak", "Lifejacket", "Paddle", "Hat"}
allNames := products[1:]
someNames := allNames[1:3]
```
__Принципиальная схема__ -  как в коде
| someNames |    | allNames |    |   | Массив     |
|-----------|----|----------|----|---|------------|
|           |    |          |    | 0 | Kayak      |
|           |    | 0        | -> | 1 | Lifejacket |
| 0         | -> | 1        | -> | 2 | Paddle     |
| 1         | -> | 2        | -> | 3 | Hat        |

__Действительная схема__ - срезы _указывают_ на диапазоны одного __массива__
| allNames |    |   | Массив     |    | someNames |
|----------|----|---|------------|----|-----------|
|          |    | 0 | Kayak      |    |           |
| 0        | -> | 1 | Lifejacket |    |           |
| 1        | -> | 2 | Paddle     | <- | 0         |
| 2        | -> | 3 | Hat        | <- | 1         |

## == COPY ==
Ф-ция `copy` копирует элементы между __срезами__, новый __срез__ будет ссылаться на новый __массив__
__!!!__ Копирование идет до тех пор пока не закончится __целевой срез__ или __срез источник__
__!!!__ Размер __целевого среза__ не изменяется: даже если в __целевом срезе__ хватает __ёмкости__, то все равно копируется только кол-во элементов равное __длинне__
```
package main

import "fmt"

func main() {
    products := [4]string {"Kayak", "Lifejacket", "Paddle", "Hat"} 
	allNames := products[1:]
	someNames := make([]string, 2)
	copy(someNames, allNames)

	fmt.Println("someNames:", someNames)
	fmt.Println("allNames:", allNames)
}
```
Результат выполнения:
```
someNames: [Lifejacket Paddle]
allNames: [Lifejacket Paddle Hat]
```

Можно указать диапазон, в который `copy` будет копировать элементы:
```
package main

import "fmt"

func main() {
    products := [4]string {"Kayak", "Lifejacket", "Paddle", "Hat"} 
	allNames := products[1:]
	someNames := []string {"Boots", "Canoe"}
	copy(someNames[1:], allNames[2:3])           // <-- копируемые элементы будут вставляться в позиции начаная с 1

	fmt.Println("someNames:", someNames)
	fmt.Println("allNames:", allNames)
}
```
Результат выполнения:
```
someNames: [Boots Hat]
allNames: [Lifejacket Paddle Hat]
```

## == Неинициализированный срез ==
__Неинициализированный срез__ - это когда переменная создана, но __срез__ не создан через `make`
`var someNames []string` - переменная определяется, но не иницилизируется
Ф-ция `copy` не добавит элементы в такой срез - так как его __длинна__ равна 0 (и __ёмкость__ тоже)

## == Удаление элементов среза ==
Встроенной функции для удаления элементов из __среза__ - нет, поэтому удаление можно сделать с помощью диапазонов
```
package main

import "fmt"

func main() {
    products := [4]string {"Kayak", "Lifejacket", "Paddle", "Hat"} 

	deleted := append(products[:2], products[3:]...)       // <-- append объединяет все элементы среза, кроме того, который надо удалить

	fmt.Println("Deleted:", deleted)
}
```
Результат выполнения:
```
Deleted: [Kayak Lifejacket Hat]
```

## == Перечисление срезов ==
Надо использовать `for` и `range`
За подробностями смотри [[../flowcontrol.md### === Перечисление последовательностей ===|FlowControl]]

## == Сортировка элементов срезов==
С помощью пакета `sort`
TODO добавить ссылку на конспект про этот пакет `reflect`

## == Сравнение срезов ==
__Срезы__ можно сравнивать только с нулевым значением, т.е. с `nil`

Нулевым значением __типа среза__ является значени `nil`. Такой __срез__ не имеет базового массива.__Длинна__ и __ёмкость__ нулевого массива равны нулю. Но нулевой __длинной__ и __емкостью__ могут обладать и не нулевые срезы: `[]int{}`, `make([]int,3)[3:]`. Поэтому, если нужно проверить является ли __срез__ пустым, то надо проверять его __длинну__: `len(s) == 0`


Сравнение двух __срезов__ вызовет ошибку при компиляции
Можно сравнивать срезы при помощи пакета `reflect`
TODO добавить ссылку на конспект про пакет `reflect`

## == Получение массива лежащего в основе среза ==
```
package main

import "fmt"

func main() {
	p1 :=  []string {"Kayak", "lifejacket", "Paddle", "Hat"}
	arrayPtr := (*[3]string)(p1)        // <-- перобразование типа среза
	array := *arrayPtr                  // <-- следование указателю

	fmt.Println(array)
}
```
`*[3]string` - __указатель на массив__ который содержит три строковых элемента.
__!!!__ В данном случае, __длинна массива__ должна быть НЕ БОЛЬШЕ __длинны среза__, т.е. `*[5]string` - не прокатит:
```
panic: runtime error: cannot convert slice with length 4 to array or pointer to array with length 5
```
__!!!__ `*[4]int` тоже не прокатит:
```
./main.go:7:24: cannot convert p1 (variable of type []string) to type *[4]int
```

## == Передача среза в функцию ==
Если передаем __срез__ по _значению_ то внутри __Ф__ будет новый __срез__,  со своей _длинной_ и _ёмкостью_, но указывать он будет на тот же массив (__!!!__) что и __срез__ снаружи. Следовательно:1) если изменить элементы __среза__ - тогда это отобразится в __срезе__ снаружи __Ф__
2) если изменить _длинну_ или _ёмкость_, тогда это не отразится на __срезе__ снаружи - так как у него останется его собственная _длинна_ и _ёмкость_

# == Sensetive shit ==
__!!!__ `len` и `cap` - работают и для __массивов__  - они возвращают длинну __массива__
__!!!__ Если несколько __срезов__ указывают на __ОДИН массив__, то при изменении размера одного из __срезов__ будет создан новый __массив__ и, следовательно, __срезы__ станут указывать на __РАЗНЫЕ массивы__
