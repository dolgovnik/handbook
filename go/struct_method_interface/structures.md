[[index.md|<--Go Back]]

# = Structures =
__Структура__ (__С__) - позволяет определить пользовательский тип данных
Пример:
```
package main

import "fmt"

func main() {
	type Product struct {
		name string
		category string
		price float64
	}


	fmt.Println(kayak.name, kayak.category, kayak.price)
	kayak.price = 300
	fmt.Println("Changed price:", kayak.price)
}
```
Результат выполнения:
```
Kayak Watersports 275
Changed price: 300
```

__!!!__ Структуру с `const` определить нельзя
__!!!__ Можно использовать __теги структуры__, с ними работает пакет `reflect`

Можно сделать частичное присвоение. В этом случае неуказанному полю будет присвоено значение по умолчанию, в данном случае `kayak.price = 0`
```
kayak := Product {
	name: "Kayak",
	category: "Watersports",
}
```

Можно использовать литеральный синтаксис, тогда поля присваиваются в том порядке, в которм они определены в __С__:
```
var kayak = Product {"Kayak", "Watersports", 275.00}
```

## == Встроенное поле ==
__Встроенное поле__ - это поле определенное без имени, доступ к нему осуществляется через имя его типа => __один тип - одно поле__
Если нужно два поля одного типа, то второму полю придется дать имя. Тогда только одно поле этого типа будет __встроенным__
Пример:
```
package main

import "fmt"

func main() {
	type Product struct {
		name string
		category string
		price float64
	}

	type stockLevel struct {
		Product                    // <-- Встроенное поле
		count int                  // <-- Регуляное поле
	}

	stockItem := stockLevel {
	    Product: Product {"Kayak", "Watersports", 275},
	    count: 100,
	}

	fmt.Println("Name:", stockItem.Product.name)
	fmt.Println("Count:", stockItem.count)
}
```
Результат выполнения:
```
Name: Kayak
Count: 100
```

__!!!__ Если поля структуры имеют уникальные имена, то можно получить к ним доступ напрямую, а не через тип

## == Сравнение значений структуры ==
Значение __С__ сопоставимы если  можно сравнить ВСЕ их поля
Значения __С__ равны если равны ВСЕ их поля
__С__ нельзя сравнивать если __тип С__ определяет поля с несравнимыми типами - например, если в __С__ одно из полей __срез__
Можно сравнивать __С__ одного типа, т.е. одинаковые __С__. __С__ разных типов сравнить не получится, компилятор выдаст ошибку. Есть способ это сделать с помощью преобразования типа структуры

## == Преобразование между типами структур ==
Пример:
```
package main

import "fmt"

func main() {
	type Product struct {
		name string
		category string
		price float64
	}
	type Product1 struct {
		name string
		category string
		price float64
	}
	p1 := Product {"Aa", "Bb", 12}
	p2 := Product1 {"Aa", "Bb", 12}

	fmt.Println(p1 == Product(p2))
}

```

__!!!__  Поля __С__  должны быть одинаковыми (имена, тип, порядок)

## == Анонимный тип структуры ==
определяется без использования имени
```
func writeName (val struct {              // <-- анонимная структура
                    name string,
					category string,
					price float64
                }) {
    fmt.Println("Name:" val.name)
}

prod = Product {name: "Kayak",
                category: "Watersports",
				price: 275,
       }

writeName(prod)
```
Получается что __Ф__ принимает на вход _любую_ __C__ у которой совпадают имена полей и их типы.

## == Структуры и Указатели ==
Стандартное поведение __Go__: присвоение __C__ новой переменной или использование __С__ в качестве параметра __Ф__ создает новое значение, которое копирует значения полей.
Если копировать не надо, а надо использовать один _объект_, тогда надо использовать __указатель__
Пример:
```
package main

import "fmt"

func main() {
	type Product struct {
		name string
		category string
		price float64
	}
	
	p1 := Product {"Kayak", "Watersports", 300}
	
	p2 := p1     // <-- коприрование
	p3 := &p1    // <-- использование указателя. Тип p3 - *Product, указатель на структуру Product

	p1.name = "Original Kayak"

	fmt.Println("p1.name:", p1.name)
	fmt.Println("p2.name:", p2.name)
	fmt.Println("p3.name:", (*p3).name) // <--Скобки чтобы следовать указателю, а затем считывает из него поле name
}
```
Результат выполнения:
```
p1.name: Original Kayak
p2.name: Kayak
p3.name: Original Kayak
```

__Синтаксический сахар__ при следовании указателю __С__ - можно не использовать `*`, но только при доступе к полям
Пример:
```
package main

import "fmt"

type Product struct {
	name string
	category string
	price float64
}	

func calcTax(product *Product) {               // <-- тип Указатель на структуру Product
	if (product.price > 100){                  // <-- Следование указателю БЕЗ * и ()
		product.price += product.price * 0.2
	}
}
 	
func main() {
 	   
 	p := Product {"Kayak", "Watersports", 300}

	calcTax(&p)                               // <-- передаем в Ф указатель

	fmt.Println("p.name:", p.price)           // <-- изменяется оригинальная структура
}
```
Результат выполнения:
```
p.name: 360

```

БОЛЬШЕ __Сахара__!!!! Можно сразу создавать указатель на структуру. Пример выше станет таким:
```
package main

import "fmt"

type Product struct {
	name string
	category string
	price float64
}	

func calcTax(product *Product) {
	if (product.price > 100){
		product.price += product.price * 0.2
	}
}
 	
func main() {
 	   
 	p := &Product {"Kayak", "Watersports", 300}    // <-- Сразу создаем указатель на структуру

	calcTax(p)                                     // <-- передаем его в Ф

	fmt.Println("p.name:", p.price)
}
```
Результат выполнения:
```
p.name: 360
```

# == Конструктор Структуры ==
__Функция-Конструктор__(__ФК__) - отвечает за создание значений __С__ c использование значений полученных через переданные параметры
__!!!__ __ФК__ возвращают __указатели__

Пример:
```
package main

import "fmt"

type Product struct {
	name string
	category string
	price float64
}	

func newProduct(name string, category string, price float64) *Product {  // <-- возвращает тип Указателя
	return &Product{name ,category, price}  // <-- возвращает тип Указателя
}
 	
func main() {
    products := [2]*Product {
		newProduct("Kayak", "Watersport", 300),
		newProduct("Ball", "Football", 30),
	}

	for _, p := range products {
		fmt.Println("Name:", p.name)
	}
}
```
Результат выполнения:
```
Name: Kayak
Name: Ball
```
__ФК__ обычно называются `new<ИмяСтруктуры>` или `New<ИмяСтруктуры>`

## == Тип указателя как поле для структуры ==
Пример:
```
package main

import "fmt"

type Product struct {
	name string
	category string
	price float64
	*Supplier
}

type Supplier struct {
	name string
	city string
}

func newProduct(name string, category string, price float64, supplier *Supplier) *Product {
	return &Product{name, category, price, supplier}
}
 	
func main() {
	acme := &Supplier {"Acme Co", "New York"}
    products := [2]*Product {
		newProduct("Kayak", "Watersport", 300, acme),
		newProduct("Ball", "Football", 30, acme),
	}

	for _, p := range products {
			fmt.Println("Name:", p.name, "Supplier:", p.Supplier.name, "City:", p.Supplier.city)
	}
}
```
Результат выполнения:
```
Name: Kayak Supplier: Acme Co City: New York
Name: Ball Supplier: Acme Co City: New York
```

__!!!__ Доступ к полям через имя __типа С__, так как в данном случае это __встроенное поле__

__!!!__ При копировании такой __С__, __указатель__ тоже скопируется, но останется __указателем__, а не значением - Т.е. это __поверхностная копия__ как в __Python__ `copy`
__!!!__ В __Go__ нет __глубокого копирования__, как в __Python__ `deepcopy`

В данном случае, можно скопировать вот так:
```
func copyProduct(product *Product) Product {   // <-- получает указатель
    p := *product                              // <-- следует указателю, копирует его в p
    s := *product.Supplier                     // <-- следует указателю на Supplier, в s
	p.Supplier = &s                            // <-- указатель на s присваетсяет в p
	return p
}

p2 = copyProduct(p1)
```

## == Нулевое значение структуры ==
__!!!__ Если полям присвоен _нулевой тип_, то __С__ имеет _нулевое_ значение
```
type Product struct {
	name string
	category string
	price float64
}

var prod Product         // prod.name = "", prod.category = "", prod.price = 0
var prodPtr *Product     // <-- nil
```
Проблема возникнет, если в __C__ `Product` будет указатель на другую структуру
```
type Product struct {
	name string
	category string
	price float64
	*Supplier
}

type Supplier struct {
    name string
}

var prod Product         // <-- prod.Supplier.name - вызовет ошибку, так как prod.Supplier - nil
var prodPtr *Product     // <-- nil
```
Можно починить вот так:
```
var prod Product = Product{ Supplier: &Supplier{} }
```

## == NEW ==
Можно использовать встроенную __Ф__ `new`:
```
var lifejacket = new(Product)
```
Это эквивалентно такому подходу
```
var lifejacket = &Product{}
```
