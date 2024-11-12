[[index.md|<--Go Back]]

## == Interfaces ==
__Интерфейс__ (__И__) - описывает набор __М__ без указания из реализации. Если __тип__ реализует все методы, определенные интерфейсом, то значение этого __типа__ можно использовать везде где разрешен даннай __И__
Пример:
```
// ################
// ###product.go###
// ################
package main

type Product struct {
	name string
	category string
	price float64
}

func (p Product) getName() string {        // <-- реализация интерфейса
	return p.name
}

func (p Product) getCost(_ bool) float64 { // <-- реализация интерфейса
	return p.price
}

// ################
// ###service.go###
// ################
package main

type Service struct {
	description string
	durationMonths int
	monthlyFee float64
}

func (s Service) getName() string {              // <-- реализация интерфейса
	return s.description
}

func (s Service) getCost(recur bool) float64 {   // <-- реализация интерфейса
	if (recur) {
		return s.monthlyFee * float64(s.durationMonths)
	}
	return s.monthlyFee
}

// #############
// ###main.go###
// #############
package main

import "fmt"

type Expense interface {           // <-- Определение интерфейса
	getName() string               // <-- сигнатура метода: имя,типы параметров и возвращаемого результата
	getCost(annual bool) float64   // <-- сигнатура метода
}

func main() {
	expenses := []Expense{
	    Product{"Kayak", "Watersports", 275.50},
	    Service{"Boat Cover", 12, 89.0},
	}

	for _, e := range expenses {
		fmt.Println("Name:", e.getName(), "Coast:", e.getCost(true))
	}
}
```
Результат выполнения:
```
Name: Kayak Coast: 275.5
Name: Boat Cover Coast: 1068
```

__!!!__ Все медоды перечисленные в __И__ должны быть определены.
__!!!__ Имена параметров могут быть разные. Должны совпадать _имена_, _типы параметров_ и _типы результатов_

После того как __И__ реализован (т.е созданы все методы, имена и типы совпадают), можно ссылаться на значение через __тип интерфейса__. В примере выше мы ссылаемся на значение `Product` и `Service` через интерфейс `Expense`

Можно передавать __тип И__ в качестве параметра __Ф__:
```
func calcTotal(expenses []Expense) (total float64) {
    for _, item := range expenses {
	    total += item.getCost(true)
	}
	return
}
```

Можно использовать __тип И__ в качестве поля __С__:
```
type Account struct {
    accountNum int
	expenses []Expense
}
```

### === Динамический и Статический тип ==
Переменная, __тип__ которой является __И__ имеют два типа
* _статический_ - Интерфейсный тип, в примере `Expense` - он никогда не меняется
* _динамический_ - тип значения, в примере `Product` и `Service` - может измениться если будет присвоенно значение другого типа реализующего данный __И__

Цикл `for` имеет дело только со _статическим типом_.

__И__ позволяют использовать различные _динамические типы_ в одном месте через реализацию _статического_ интерфейса


### === Приемники, методы, указатели ===
__!!!!!!!!__ __ОЧЕНЬ ВАЖНЫЙ МОМЕНТ__
Пример __#1__:
```
product := Product {"Kayak", "Watersports", 275.0}
var expense Expense = product      // <-- Значение product КОПИРУЕТСЯ
product.price = 100
fmt.Println(product.price)        // <-- 100
fmt.Println(expense.getCost())    // <-- 275
```
Значение `product` скопировалось, поэтому изменение его полей не влияют на значение полей `expense`
Пример __#2__:
```
product := Product {"Kayak", "Watersports", 275.0}
var expense Expense = &product      // <-- используем УКАЗАТЕЛЬ, значение НЕ КОПИРУЕТСЯ
product.price = 100
fmt.Println(product.price)        // <-- 100
fmt.Println(expense.getCost())    // <-- 100
```
Ссылка на `Product` присваетсяет переменной `Expense`, но это __НЕ МЕНЯЕТ тип переменной И__(`expense`), он по прежнему является `Expense`. Выглядит _не логично_ так как переменная всегда `Expense`, а значение может быть либо `Product` либо `*Product`
__!!!__ Т.е. можно выбирать как будет использоваться значение присвоенное переменной __И__

Пример __#3__:
```
func (p *Product) getName() string {   // <-- Приемник - тип указателя!
    return p.name
}

func (p *Product) getCost(_ bool) float64 {
    return p.price
}
```
__!!!__ Теперь `Product` НЕ реализует __И__ `Expense`, так как необходимые __М__ не определены для __С__ `Product`. __И__ `Expense` реализуется __типом__ `*Prodiuct` - т.е. __указатели__ на значение `Product` можно рассматривать как занчения `Expense`. Следовательно код ниже работать не будет:
```
product := Product {"Kayak", "Watersports", 275.0}
var expense Expense = product             // Product не реализует интерфейс (тип) Expense
```

### === Сравнение значений интерфейса ===
Два значения __И__ равны если: они имеют одинаковый __динамический тип__ и все их поля равны.

__!!!__ Надо быть осторожным с __указателями__: если __динамический тип__ будет __типом указателя__, тогда сравнение может быть неудачным, так как __указатели__ равны только тогда, когда они указывают в одно и тоже место в памяти.

__!!!__ Могут быть проблемы если __динамический тип__ не сопоставим, например если в __С__ есть __срез__ или другой _несравнимый_ __тип__



### === Утверждения типа ===
Часто надо иметь возможность из __И__ получить доступ к __динамическому типу__. Это называется - __сужение типа__: процесс перехода от _менее_ точного __типа__ к _более_ точному.
__Утверждение типа__ используется для доступа к _динамическому_ __типу__ значения __И__
Пример:
```
package main

import "fmt"

type Expense interface {
	getName() string
	getCost(annual bool) float64
}

func main() {
	expenses := []Expense{
	    Service{"Boat Cover", 12, 89.0},
	    Service{"Paddle Protect", 6, 7.5},
		&Product{"Kayak", "Watersports", 275.0},
	}

	for _, e := range expenses {
		if s, ok := e.(Service); ok {  // <-- Утверждение типа, далее можно использовать
		                               // любые поля из типа Service, а не только методы
									   // определенные в интерфейсе Expense
	   	    fmt.Println("Service:", s.description)
        } else {
	   	    fmt.Println("Expense:", e.getName())
		}
	}
}
```
Результат выполнения:
```
Service: Boat Cover
Service: Paddle Protect
Expense: Kayak
```

__!!!__ НЕ ПУТАТЬ __утверждение типа__ и __переобразование типа__:
__утверждение__ применяется только к __И__ - сообщает компилятору, что значентие __И__ имеет определенный __динамический тип__
__преобразование__ применяется к определенным __типам__, а не к __И__, только в том случае если структура этих __типов__ совместима

### === Switch ===
Можно использовать оператора `switch`
```
for _, expense := range expenses {
    switch value := expense.(type){   // <-- специальный тип утверждения
	    case Service:
		    fmt.Println("Service")
		case *Product: 
		    fmt.Println("*Product")
		default:
		    fmt.Printlnervico("Something")
	}
}
```
__!!!__ внутри оператора `case` работает __утверждение__, поэтому там можно использовать ВСЕ поля и методы __типа__.

### === Пустой интерфейс ===
_Пустой_ __И__ - это __И__, который не имеет __М__, он может представлять ЛЮБОЙ __тип__. Удобно для группировки __типов__, которые не имеют ничего общего между собой.
Пример:
```
data := []interface{}{    // <-- срез пустых интерфейсов
    "This is a string",
	100,
	true
}
```
Дальше можно обрабатывать такой __срез__ с помощью `switch`

Можно использовать так:
```
func processItem(item interface{}){
    switch val := item.(type) {
	    case Product:
		.
		.
		.
	}
}
```

Можно использовать для переменных параметров - позволяет вызывать __Ф__ с любым кол-вом аргументовлюбого типа:
```
func processItem(items ...interface{}){
    for _, item := range items {
	    switch value := item.(type){
		    case Product:
			.
			.
			.
		}
	}
}
```
