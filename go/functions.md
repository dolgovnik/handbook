[[../index.md|<--Go Back]]

# = Functions =
__Функция__ (__Ф__) - это группа операторов, которые можно использовать (сколько угодно раз) как одно действие
Пример:
```
package main

import "fmt"

func printPrice(product string, price float64, taxRate float64) {
	taxAmount := price * taxRate
    fmt.Println(product, "price:", price, "Tax:", taxAmount)
}

func main() {
	printPrice("Kayak", 279.0, 0.2)
	printPrice("LIfejacket", 48.95, 0.2)
}
```

__!!!__ __Go__ не поддерживает необязательные параметры или значения по умолчанию для параметров __Ф__
__!!!__ Тип можно не указывать если смежные параметры имеют одинаковый тип: `price` и `taxRate` в примере ниже:
```
func printPrice(product string, price, taxRate float64) {
```
__!!!__ Можно использовать `_` для пропуска передаваемого параметра, т.е. он не будет использоваться внутри __Ф__
    Можно опустить __ВСЕ__ имена параметров, даже не используя `_`
    Это может быть полезно при реализации __методов__ требуемых __интерфейсом__
	
## == Вариационные параметры == 
Пример:
```
package main

import "fmt"

func printSuppliers(product string, suppliers ...string){
	for _, supplier := range suppliers {
		fmt.Println("Product:", product, "Supplier:", supplier)
	}
}

func main(){
	printSuppliers("Kayak", "Acme Kayaks", "Bob's Boats", "Crazy Canoes")
	printSuppliers("Lifejacket", "Sail Safe Go")
}
```
Результат выполнения:
```
Product: Kayak Supplier: Acme Kayaks
Product: Kayak Supplier: Bob's Boats
Product: Kayak Supplier: Crazy Canoes
Product: Lifejacket Supplier: Sail Safe Go
```
__Вариативный параметр__ (__ВП__) опеделяется так: `...<тип>`, например `...string`
__!!!__ __ВП__ должен быть последним параметром в функции
__!!!__ Значения __ВП__ содержатся в __срезе__, т.е. в нашем примере тип параметра `suppliers` - `[]string`
__!!!__ __ВП__ можно заменить __срезом__, но это не очень удобно по сравнению с __ВП__
__!!!__ Если ничего не передать, то значение __ВП__ будет `nil`

Можно распаковать срез в __ВП__, пример ниже:
```
package main

import "fmt"

func printSuppliers(product string, suppliers ...string){
	for _, supplier := range suppliers {
		fmt.Println("Product:", product, "Supplier:", supplier)
	}
}

func main(){
    names := []string {"Acme Kayaks", "Bob's Boats", "Crazy Canoes"}
	printSuppliers("Kayak", names...)
}
```

## == Указатель в качестве параметра функции ==
__!!!__ По умолчания __Go__ _копирует_ значения, которые используются в качестве аргументов. Следовательно все измения ограничиваются телом __Ф__
Можно передать __указатель__, тогда изнутри __Ф__ будет обращение к настоящему значению
Пример:
```
package main

import "fmt"

func swapValues(first *int, second *int) {
	fmt.Println("Before swap:", *first, *second)
	temp := *first
	*first = *second
	*second = temp
	fmt.Println("After swap:", *first, *second)
}

func main() {
	val1, val2 := 10, 20
	fmt.Println("Before call:", val1, val2)
	swapValues(&val1, &val2)
	fmt.Println("After call:", val1, val2)
}

```
Результат выполнения:
```
Before call: 10 20
Before swap: 10 20
After swap: 20 10
After call: 20 10

```

## == Возврат результатов ==
Пример:

```
func swapValues(first int, second int) (int, int) {
    return second, first
}
```
__!!!__ Удобно возвращать два значения: результат и __ok__ (__true__ или __false__)

Пример __именованных результатов__:
Суть - присваиваем имя возвращаемому результату, когда функция доходит до __return__, то возвращает текущее значение переменной с определенным ранее именем
Пример:
```
func testExapmle(val1 int, val2 int) (t1 int, t2 int){
    t1 = val1
	t2 = val2
	return
}
```

## == Defer ==
__defer__ - планирует вызов функции, который будет выполнен непосредственно перед возвратом из текущей функции
Используется для освобождения ресурсов - закрытие файлов, соединений и т.п.
Пример:
```
package main

import "fmt"

func main() {
	fmt.Println("Start")
	defer fmt.Println("Defer 1")
	defer fmt.Println("Defer 2")
	fmt.Println("Stop")
}
```
Результат выполнения:
```
Start
Stop
Defer 2
Defer 1
```

## == Sensetive shit ==

