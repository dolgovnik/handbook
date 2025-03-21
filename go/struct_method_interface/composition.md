[[index.md|<-- Go Back]]

# = Composition =

__Композиция__ (__К__) - это процесс создания новых __типов__ путем объединения __структур__ (__С__) и __интерфейсов__ (__И__)
__К__ можно использовать для создания иерархии __типов__
__К__ достигается объединением __типов С__
Пример:
```
##############
### main.go###
##############

package main

import (
	"fmt"
	"composition/store"
)

func main() {
	boats :=  []*store.Boat{
	    store.NewBoat("Kayak", 275, 1, false),
	    store.NewBoat("Canoe", 400, 3, false),
	    store.NewBoat("Tender", 650.25, 2, true),
	}

	for _, b := range boats {
		fmt.Println("Conventional:", b.Product.Name, "Direct:", b.Name) // <-- до поля Встроенного
		                                                                // типа можно добраться
																		// разными способами } }

######################
###store/product.go###
######################
package store type Product struct {
    Name string
	Category string
	price float64
}

func NewProduct(name string, category string, price float64) *Product {
	return &Product{name, category, price}
}

func (p *Product) Price(taxRate float64) float64 {
	return p.price + (p.price * taxRate)
}

###################
###store/boat.go###
###################

package store

type Boat struct {    // <-- Заключенный тип
	*Product          // <-- Встроенный тип
	Capacity int      // <-- Регулярное поле
	Motorized bool    // <-- Регулярное поле
}

func NewBoat(name string, price float64, capacity int, motorized bool) *Boat {
	return &Boat{NewProduct(name, "Watersports", price), capacity, motorized}
}
```
Результат выполнения:
```
Conventional: Kayak Direct: Kayak
Conventional: Canoe Direct: Canoe
Conventional: Tender Direct: Tender
```

__Тип__ `Boat` не определяет поле `Name`, но его можно рассматривать как будто оно _определено_ - это называется __продвижение полей__
Таким же образом __продвигаются__ и __М__.
```
fmt.Println("Conventional:", b.Product.Price(0.2), "Direct:", b.Price(0.2)) // <-- до поля Встроенного
```
__!!!__ __Тип поля__ `Product` - __продвигаются__ любые методы приемников `Product` и `*Product`
__!!!__ __Тип поля__ `*Product` - __продвигаются__ любые методы приемника `*Product`
__!!!__ поля могут __продвигаться__ на несколько уровней выше, т.е. может быть несколько уровней вложенности __С__

## == Несколько встроенных полей ==

__!!!__ __С__ может содержать _несколько_ __встроенных__ полей. В Этом случае, будут __продвигаться__ методы каждого встроенного поля. Например:
```
type RentalBoat struct {
    *Boat                  // <-- Будут продвинуты поля структуры Boat
	IncludeCrew bool
	*Crew                  // <-- Будут продвинуты поля структуры Crew
}
```

## == Продвижение полей не работает ==
__Продвижение__ работет только если в _охватывыющем_ __типе__ НЕТ __поля__ или __метода__ с тем же именем что и во __встроенном__ поле

## == Неоднозначность продвижения ==
Можно сделать так что, будет непонятно из какого __встроенного поля__ брать __М__
В этом случае __Go__ выдаст ошибку

## == Композиция интерфейсов ==
__!!!__ Очень важно для понимания
__Типы__ - это не классы, они ведут себя по-другому!
__К__ очень похожа на классы в других языках программирования. Но есть отличие: составленный __тип__ не может использваться там, где требуются типы, из которых он составлен

Пример:
```
products := map[string]*store.Product {
    "Kayak": store.NewBoat("Kayak", 279, 1, false), // <-- Ошибка, так как *Boat это не *Product
	"Ball": store.NewProduct("Socker ball", "Socker", 19.50)
}
```

__!!!__ __Go__ учитывает __продвигаемые__ __М__ при определении соответствует ли __тип__ __интерфейсу__

Пример:

```
#################
###cat main.go###
#################

package main

import (
	"fmt"
	"composition/store"
)

func main() {
	products := map[string]store.ItemForSale{ // <-- Объединяем разные типы через один интерфейс
		"Kayak": store.NewBoat("Kayak", 279, 1, false),
		"Ball": store.NewProduct("Socker Ball", "Socker", 19.50),
	}

	for key, p := range products {
		fmt.Println("Key:", key, "Price:", p.Price(0.2)) // <-- М Price берется напрямую из
		                                                 // *Product или продвигается из него
														 // в *Boat
	}
}

#######################
###store/forsale.go####
#######################

package store

type ItemForSale interface {        // <- Интерфейс
	Price(taxRate float64) float64
}
```

Результат выполнения:

```
Key: Kayak Price: 334.8
Key: Ball Price: 23.4
```

__!!!__ Могут быть проблемы с оператором `switch - case`, пример:
```
for key, p := range products {
    switch item := p.(type) {
	    case *store.Product, *store.Boat: // <-- утверждение т не выполняется, item имеет тип ItemForSale
		    fmt.Println(item.Name)        // <-- тут будет ошибка, так как ItemForSale
			                              // не имеет метода Name
		default:
		    fmt.Println(....)
	}
}
```
Это можно решить несколькими способами:
```
case *store.Product:
    fmt.Println(...)
case *store.Boat:
    fmt.Println(...)
```
Или так: создать еще один __И__ и заматчить его через `switch - case`
```
type Discribable interface {
    GetName() string
}

func (p *Product) GetName() string {
    return p.Name
}
.
.
.
case store.Discribable:
    fmt.Println(...)
default:
....
```
В этом случае тоже есть проблема - __М__ `Price` доступен только через __утверждение типа__ `ItemForSale` => можно добавить __М__ `Price` в __И__ `Discribable` или использовать __составление И__ 

## == Составление интерфейсов ==
Один __И__ может содержать в себе другой __И__ => __типы__ должны реализовывать все методы, которые оперделены _включающими_ и _вложенными_ __И__
Такое действие называется __Внедрением__
В этом случае, все проще чем с __C__ - результатом является простое объединение методов
```
type Describable interface {
    GetName() string
	ItemForSale
}
```

# = Sensitive shit =
__!!!__ если использовать литеральный синтаксис, то не получится назначать значения __продвигаемым полям__ напрямую. Это можно сделать только через __С__ в которой определено это поле
Т.е вот так нельзя:
```
boat = store.Boat{Name: "Kayak", Category: "Watersports", Capacity: 1, Motorized: false}
```

А вот так можно:
```
boat = store.Boat{Product: &store.Product{Name: "Kayak", Category: "Watersports"}, Capacity: 1, Motorized: false}
```

# = Encapsulation =
__Переменная__ (__М__) называется __инкапсулированной__, если она недостукна клиенту этого объекта. __Инкапсуляция__ иногда называется  _сокрытием_ информации - это ключевой аспект _ООП_
В __Go__ есть только один способ управиль видимостью - использоватьние строчных и прописных букв в названиях _объектов_
_Строчная_ буква - не экспортируется
_Прописаная_буква - экспортируется

__!!!__ Поэтому для инкапсуляции очень хорошо подходят структуры - можно скрывать отдельные поля и методы

__!!!__ Единица инкапсуляции - это пакет, внутри него видны любые поля и методы.
