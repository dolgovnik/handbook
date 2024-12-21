[[index.md|<--Go Back]]

# = Methods =
__Метод__ (__М__) - это __Ф__, которую можно вызвать через значение (тип)
Пример:
```
package main

import "fmt"

type Product struct {
	name string
	category string
	price float64
}

func newProduct(name string, category string, price float64) *Product {
	return &Product{name, category, price}
}

func (product *Product) printDetails() {
		fmt.Println("Name:", product.name, "Category:", product.category, "Price:", product.price)
}

func main() {
	products := [3]*Product {
		newProduct("Kayak", "Watersports", 275.00),
		newProduct("LIfejacket", "Watersports", 30),
		newProduct("Soccer Ball", "Socker", 48.56),
	}

	for _, p := range products {
		p.printDetails()
	}
}
```
Результат выполнения:
```
Name: Kayak Category: Watersports Price: 275
Name: LIfejacket Category: Watersports Price: 30
Name: Soccer Ball Category: Socker Price: 48.56
```
Это определение __М__:
```
func (product *Product) printDetails() {
		fmt.Println("Name:", product.name, "Category:", product.category, "Price:", product.price)
}
```
`product` - __приемник__ - тип, к которому относится __М__, его можно использовать внутри __М__

__!!!__ __М__ вызывается через значение, __тип__ которого соответствект __приемнику__
__!!!__ Всё работает как у __Ф__, т.е. можно указать параметры и __тип__ возвращаемого значения.

__!!!__ __Go__ не поддерживает перегрузку __М__, т.е. комбинация имени __М__ и __типа приемника__ должна быть _уникальной_. Одинаковые имена __М__ недопустимы независимо от того какие параметры они принимают.

__!!!__ можно хранить __типы__ и __М__ в разных фалах

## == Приемники указателей и значений ==
Если __приемник__ является __типом указателя__, то __М__ может быть вызван через обычное значение __базового типа__. Т.е. __М__ типа `*Product` может использоваться со значением `Product`
В обратную сторону это тоже работает

Т.е. приведенные ниже __М__ - это одно и тоже. Соответственно, перегрузка на них не будет работать
```
func (product *Product) printDetails() {...}
```
```
func (product Product) printDetails() {...}
```

## == Методы для псевдонимов типов ==
__М__ применяются не только к __С__, а ко всем сущностся определяемым с помощью `type`, т.е. например для __псевдонимов типов__
Пример:
```
package main

import "fmt"

type Product struct {
	name string
	category string
	price float64
}

type productList []Product    // <-- создание псевдонима типа

func (products *productList) calcCategoryTotal() map[string]float64 {  // <-- Определение метода
	totals := make(map[string]float64)
	for _, p := range *products {
		totals[p.category] = totals[p.category] + p.price
	}
	return totals
}

func main() {
	products := productList {
	    {"Kayak", "Watersports", 275.0},
	    {"LIfejacket", "Watersports", 30},
	    {"Soccer Ball", "Soccer", 44.5},
	}

	for k, v := range products.calcCategoryTotal(){
			fmt.Println("Category:", k, "Total:", v)
	}
}

```
Результат выполнения:
```
Category: Watersports Total: 305
Category: Soccer Total: 44.5
```
