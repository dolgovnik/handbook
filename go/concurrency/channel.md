[[concurrency.md|<-- Go Back]]

# = Каналы =
__Канал__ (__КН__) - применяется для передачи данных между __ГР__ - как будто магистраль, через которую можно отправить и получить данные
__КН__ строго типизированы, т.е они переносят только значения указанного __типа__ или __интерфейса__
Пример:
```
var channel chan float64 = make(chan float64)

```
`chan float64` - это __тип__ __канала__
`channel <- someVar` - значение _someVar_ отправляется в __КН__ _channel_. При этом, отправителю не нужно знать как будет получено отправленное значение.
`someTotal += <- channel` - получение значения из __КН__ _channel_. __!!!__ Это __блокирующая__ операция - выполнение не будет продолжаться до тех пор, пока не будет получено значение

Пример:
__ГР__ созданные для вызова `TotalPrice` отправляют свои результаты через __КН__, созданный __Ф__ `CalcStoreTotal`, где они принимаются и обрабатываются.
Программа запускается и начинает выполнять операторы _основной_ __Ф__. Это приводит к вызову `calcStoreTotal`, которая создает __КН__ и запускает несколько __ГР__. __ГР__ выполняют операторы в __М__ `TotalPrice`, котоыре отправляет результаты по __КН__. _Основная_ __Ф__ продолжает выполнять инструкции в `CalcStoreTotal`, которые получают результаты через __КН__ и используют их для создания общей суммы.

```
#############
###main.go###
#############

package main

import (
		"fmt"
		//"time"
)

func main() {
	fmt.Println("main function started")
	CalcStoreTotal(Products)
	//time.Sleep(time.Second * 10)       // <-- Получение из канала - блокирующая операция,
	                                     // поэтому не надо замедлять выполнение - программа
										 // закончит работать только после того как получит все
										 // данные из канала
	fmt.Println("main function complete")
}

###################
###operations.go###
###################

package main

import (
		"fmt"
		"time"
)

func CalcStoreTotal(data ProductData) {
	var storeTotal float64
	var channel chan float64 = make(chan float64)
	for category, group := range data {
		go group.TotalPrice(category, channel)  // <-- передаем канал при вызове метода
	}
	for i := 0; i < len(data); i++{
	    storeTotal += <- channel    // <-- собираем данные из канала
	}

	fmt.Println("Total:", toCurrency(storeTotal))
}

func (group ProductGroup) TotalPrice(category string, resultChannel chan float64) { //<-- добавляем переменную канала, для отправки результатов работы
		var total float64
		for _, p := range group {
			total += p.Price
			fmt.Println(category, "product:", p.Price)
			time.Sleep(time.Millisecond * 100)
		}
		fmt.Println(category, "subtotal:", toCurrency(total))
		resultChannel <- total    // <-- отправляем результаты в канал
}

################
###product.go###
################

package main

import "strconv"

type Product struct {
	Name string
	Category string
	Price float64
}

var ProductList = []*Product{
	{"Kayak", "Watersports", 279},
	{"Lifejacket", "Watersports", 49.95},
	{"Socker Ball", "Socker", 19.50},
	{"Corner Flags", "Socker", 34.95},
	{"Stadium", "Socker", 79500},
	{"Thinking Cap", "Chess", 16},
	{"Unsteady Cheir", "Chess", 75},
	{"Bling-Bling King", "Chess", 1200},
}

type ProductGroup []*Product

type ProductData = map[string]ProductGroup

var Products = make(ProductData)

func toCurrency(val float64) string {
	return "$" + strconv.FormatFloat(val, 'f', 2, 64)
}

func init() {
	for _, p := range ProductList {
        if _, ok := Products[p.Category]; ok {
			Products[p.Category] = append(Products[p.Category], p)
		} else {
		    Products[p.Category] = ProductGroup { p }
	    }
	}
}
```
Результат выполнения
```
main function started
Chess product: 16
Watersports product: 279
Socker product: 19.5
Socker product: 34.95
Watersports product: 49.95
Chess product: 75
Chess product: 1200
Socker product: 79500
Watersports subtotal: $328.95
Socker subtotal: $79554.45
Chess subtotal: $1291.00
Total: $81174.40
main function complete
```

## == Координация каналов ==
__!!!__ Отправка и Получение данных через __КН__ - это __БЛОКИРУЮЩИЕ__ операции:
* __ГР__, которая отправляет данные не будет выполнять никаких действий до тех пор пока другая __ГР__ не получит значение из __КН__
* Если вторая __ГР__ отправляет значение, она будет заблокирована до тех пор, пока канал не будет очищен, что приведет к созданию очереди из __ГР__ ожидающих получения значений.
* Перечисленное выше работает и в обратном направлении - __ГР__ которые получают значения, блокируются до тех пор пока другая __ГР__ не отправит значения.

Грубо говоря, передача может быть осуществлена __ТОЛЬКО__ если _отправитель_ и _получатель_ доступны, а сообщения обрабатываются _последовательно_

Для решения этой проблемы можно использовать __КН__ с __буффером__ - _отправитель_ кладет сообщение в такой __КН__ и продолжает работу. _Получатель_ может прочитать инфу из такого __КН__ когда будет готов. Если __буфер__ переполнен, то _отпавителю_ придется подождать пока _получатель_ не прочитает сообщения и в __буфере__ не появится свободное место.
```
var channel chan float64 = make(chan float64, 2) // <-- 2 - это размер буффера
len(channel)    // <-- Кол-во значений в буфере
cap(channel)    // <-- Размер буфера
```

### === Закрытие канала ===
Пример:
```
#############
###main.go###
#############

package main

import (
		"fmt"
)

func main() {
	dispatchChannel := make(chan DispatchNotification)
	go DispatchOrders(dispatchChannel)
	for {    // <-- Будет пытаться получить значеня из канала, после
	         // того как они все будут отправлены, единственная ГР
	         // попадет в дедлок и среда выполнения завершит программу
		details := <- dispatchChannel
		fmt.Println("Dispatch to", details.Cuctomer, ":", details.Quantity, "x", details.Product.Name)
	}
}

######################
###orderdispatch.go###
######################

package main

import (
		"fmt"
		"math/rand"
		"time"
)

type DispatchNotification struct {
    Cuctomer string
	*Product
	Quantity int
}

var Customers = []string{"Alice", "Bob", "Charlie", "Dora"}

func DispatchOrders(channel chan DispatchNotification) { // <-- Генерирует произвольное кол-во
                                                         // сообщений в канал
	rand.Seed(time.Now().UTC().UnixNano())
	orderCount := rand.Intn(3) + 2
	fmt.Println("Order Count:", orderCount)
	for i := 0; i < orderCount; i++ {
		channel <- DispatchNotification{
				       Cuctomer: Customers[rand.Intn(len(Customers)-1)],
					   Quantity: rand.Intn(10),
					   Product: ProductList[rand.Intn(len(ProductList)-1)],
				   }
	}
}
```
Результат выполнения:
```
Order Count: 4
Dispatch to Charlie : 9 x Unsteady Cheir
Dispatch to Bob : 9 x Socker Ball
Dispatch to Alice : 0 x Thinking Cap
Dispatch to Bob : 7 x Stadium
fatal error: all goroutines are asleep - deadlock!    // <-- Дедлок, описанный выше!

goroutine 1 [chan receive]:
main.main()
	/home/nikolay/Y/go_study/concurrency/main.go:11 +0x90
exit status 2
```

__!!!__Решить проблему __дедлока__ можно с помощью закрытия __КН__: 

```
#############
###main.go###
#############

package main

import (
		"fmt"
)

func main() {
	dispatchChannel := make(chan DispatchNotification)
	go DispatchOrders(dispatchChannel)
	for {
		if details, open := <- dispatchChannel; open {    // <-- проверяем, что канал открыт
		                                                  // перед тем как прочитать из него
		    fmt.Println("Dispatch to", details.Cuctomer, ":", details.Quantity, "x", details.Product.Name)
		} else {
			fmt.Println("Channel has been closed")
			break
		}
	}
}

######################
###orderdispatch.go###
######################

package main

import (
		"fmt"
		"math/rand"
		"time"
)

type DispatchNotification struct {
    Cuctomer string
	*Product
	Quantity int
}

var Customers = []string{"Alice", "Bob", "Charlie", "Dora"}

func DispatchOrders(channel chan DispatchNotification) {
	rand.Seed(time.Now().UTC().UnixNano())
	orderCount := rand.Intn(3) + 2
	fmt.Println("Order Count:", orderCount)
	for i := 0; i < orderCount; i++ {
		channel <- DispatchNotification{
				       Cuctomer: Customers[rand.Intn(len(Customers)-1)],
					   Quantity: rand.Intn(10),
					   Product: ProductList[rand.Intn(len(ProductList)-1)],
				   }
	}
	close(channel)    // <-- Закрываем канал, когда больше нечего отправить
}
```
Результат выполнения:
```
Order Count: 4
Dispatch to Alice : 2 x Stadium
Dispatch to Charlie : 9 x Thinking Cap
Dispatch to Charlie : 6 x Lifejacket
Dispatch to Bob : 6 x Socker Ball
Channel has been closed
```

__!!!__ Можно использовать `for range`, тогда не надо проверять закрыт __КН__ или нет, это делается автоматически внутри цикла:
```
for details := range(dispatchChannel) {
    fmt.Println("Dispatch to", details.Cuctomer, ":", details.Quantity, "x", details.Product.Name)
}
fmt.Println("Channel has been closed")
```

## == Направление канала ==
По умолчанию, __КН__ внутри __Ф__ используется как для _отправки_ так и для _получения_ данных. Это поведение можно ограничить:
```
func DispatchOrders(channel chan<- DispatchNotification) {    // <-- канал только для ОТПРАВКИ
    .....
}
```
```
func DispatchOrders(channel <-chan DispatchNotification) {    // <-- канал только для ПОЛУЧЕНИЯ
    .....
}
```
Если __КН__ используется не по назначению, то тогда будет _ошибка времени выполнения_

Строго говоря, `chan<- DispatchNotification` - это __тип__, но переменной такого __типа__ можно назначить канал _двунаправленного_ __типа__ и таким образом __явно__ ограничить использование канала в определенном месте для определенной цели. Пример:
```
dispatchChannel := make(chan DispatchNotification)
var sendOnlyChannel chan<- DispatchNotification = dispatchChannel
var receiveOnlyChannel <-chan DispatchNotification = dispatchChannel
```
Также можно использовать прямое преобразование:
```
dispatchChannel := make(chan DispatchNotification)
go DispatchOrders(chan<- DispatchNotification(dispatchChannel))
go receiveDispatches(<-chan DispatchNotification(dispatchChannel))
```

## == Оператор select ==
Оператор `select` имеет структуру аналогичную оператору `switch`, но только операторы `case` - это операции __КН__
Когда выполняется оператор `select`, то каждая операция __КН__ (в операторе `case`) оценивается до тех пор, пока не будет достигнута операция, которую можно выполнить без блокировки.
`default` - это операция _по умолчанию_, выполняется когда другие операции _не проходят_ оценку. Если оператор `default` отсутствует, тогда `select` будет заблокирован до тех пор пока не будет получено сообщение из канала
Оператор `select` оценивает операции __один__ раз, поэтому его можно использовать внутри оператора `for`

### === Получение без блокировки ===
```

#############
###main.go###
#############

package main

import (
	"fmt"
	"time"
)

func main() {
	dispatchChannel := make(chan DispatchNotification)
	go DispatchOrders(chan<- DispatchNotification(dispatchChannel))
	for {                                           // <-- постоянно проверяем канал
	    select {
		    case details, ok := <- dispatchChannel: // <-- Работа канала
			    if ok {                             // <-- Если канал открыт
				    fmt.Println("Dispatch to", details.Cuctomer, ":", details.Quantity, "x", details.Product.Name)
		        } else {                            // <-- Если канал закрыт
					fmt.Println("Channel has been closed")
					goto alldone                    // <-- выходим из цикла
		        }
			default:                                // <-- Если в канале пусто...
			    fmt.Println("-- No Message ready to be received") 
				time.Sleep(time.Millisecond * 500)  // <-- ...то подождем
		}
	}
	alldone: fmt.Println("All values received")
}

######################
###orderdispatch.go###
######################

package main

import (
		"fmt"
		"math/rand"
		"time"
)

type DispatchNotification struct {
    Cuctomer string
	*Product
	Quantity int
}

var Customers = []string{"Alice", "Bob", "Charlie", "Dora"}

func DispatchOrders(channel chan<- DispatchNotification) {
	rand.Seed(time.Now().UTC().UnixNano())
	orderCount := rand.Intn(5) + 5
	fmt.Println("Order Count:", orderCount)
	for i := 0; i < orderCount; i++ {
		channel <- DispatchNotification{
				       Cuctomer: Customers[rand.Intn(len(Customers)-1)],
					   Quantity: rand.Intn(10),
					   Product: ProductList[rand.Intn(len(ProductList)-1)],
				   }
	    time.Sleep(time.Microsecond * 750) // <-- Добавляем задержку отправки, чтобы было видно
		                                   // как оно работает
	}
	close(channel)
}
```
Результат выполнения
```
-- No Message ready to be received
Order Count: 5
Dispatch to Bob : 6 x Kayak
-- No Message ready to be received
Dispatch to Alice : 1 x Lifejacket
-- No Message ready to be received
Dispatch to Bob : 1 x Thinking Cap
-- No Message ready to be received
Dispatch to Bob : 6 x Stadium
-- No Message ready to be received
Dispatch to Bob : 0 x Lifejacket
-- No Message ready to be received
Channel has been closed
All values received
```

### === Прием с нескольких каналов ===
Если через `select` использовать несколько __КН__, то работа будет гораздо эффективней.

__!!!__ Каждый раз когда работает `select`, он проходит через операторы `case` и создает список каналов из которых можно прочитать значение без блокировки. Один из операторов `case` выбирается __случайным__ образом выполняется. Если ни один из `case` не может быть выполнен, тогда выполняется `default`

__!!!__ Написанное выше справедливо и для отправки - будет создан список каналов, в которые не заблокирована отправка и один из них (произвольный) будет использован для отправки сообщения

__!!!__ Если скомбинировать отправку и получение в разных `case`, тогда будет создан общий список доступных каналов и выбран произвольный с которым продолжится работа.

__!!!__ `select` НЕ распреляет сообщения по каналам равномерно, каждый раз используется произвольный канал!

__!!!__ Закрытый __КН__ возвращает `nil` для каждой операции приема которая происходит после закрытия канала и поэтому `case` для закрытого канала всегда будут выбираться оператором `select`, потому что они всегда готовы предоставить значение `nil` => надо запретить оператору `select` выбирать закрытий канал:
    - присваиваем переменной закрытого __КН__ значение `nil`. __КН__ `nil` никогда не будет готов и никогда не будет выбран.
	- выйти из `for` когда все __КН__ будут закрыты. Без этого, `select` бесконечно заходил бы в `default`

Пример:
```
#############
###main.go###
#############

package main

import (
	"fmt"
	"time"
)

func enumerateProduct(channel chan<- *Product){ // <-- Ф для работы со вторым каналом
    for _, p := range ProductList[:3]{
		channel <- p
		time.Sleep(time.Millisecond * 800)
	}
	close(channel)                              // <-- Закроет КН когда все отправит
}

func main() {
	dispatchChannel := make(chan DispatchNotification, 100)
	go DispatchOrders(chan<- DispatchNotification(dispatchChannel))

	productChannel := make(chan *Product, 100)
	go enumerateProduct(chan<- *Product(productChannel))

	openChannels := 2   // <-- счетчик открытых каналов, чтобы выйти из цикла for

	for {
	    select {
		    case details, ok := <- dispatchChannel: //<-- получение через один канал
			    if ok {
				    fmt.Println("Dispatch to", details.Cuctomer, ":", details.Quantity, "x", details.Product.Name)
		        } else {
					fmt.Println("dispatchChannel has been closed")
                    dispatchChannel  = nil         // <-- если канал закрыт, блокируе его выбор
					openChannels--                 // <-- уменьшаем счетчик открытых каналов
		        }
			case product, ok := <- productChannel: //<-- получение через второй канал
				if ok {
				    fmt.Println("Product:", product.Name)
				} else {
					fmt.Println("productChannel has been closed")
                    productChannel  = nil          // <-- если канал закрыт, блокируе его выбор
					openChannels--                 // <-- уменьшаем счетчик открытых каналов
				}
			default: 
			    if (openChannels == 0){           // <-- если открытых КН нет, то выходим 
					goto alldone
				}
			    fmt.Println("-- No Message ready to be received")
				time.Sleep(time.Millisecond * 500)
		}
	}
	alldone: fmt.Println("All values received")
}

######################
###orderdispatch.go###
######################

package main

import (
		"fmt"
		"math/rand"
		"time"
)

type DispatchNotification struct {
    Cuctomer string
	*Product
	Quantity int
}

var Customers = []string{"Alice", "Bob", "Charlie", "Dora"}

func DispatchOrders(channel chan<- DispatchNotification) {
	rand.Seed(time.Now().UTC().UnixNano())
	orderCount := rand.Intn(5) + 5
	fmt.Println("Order Count:", orderCount)
	for i := 0; i < orderCount; i++ {
		channel <- DispatchNotification{
				       Cuctomer: Customers[rand.Intn(len(Customers)-1)],
					   Quantity: rand.Intn(10),
					   Product: ProductList[rand.Intn(len(ProductList)-1)],
				   }
	    time.Sleep(time.Microsecond * 750)
	}
	close(channel)
}
```
Результат выполнения:
```
-- No Message ready to be received
Order Count: 5
Product: Kayak
Dispatch to Bob : 7 x Socker Ball
Dispatch to Charlie : 0 x Unsteady Cheir
Dispatch to Bob : 8 x Corner Flags
Dispatch to Charlie : 8 x Thinking Cap
Dispatch to Bob : 6 x Corner Flags
dispatchChannel has been closed
-- No Message ready to be received
Product: Lifejacket
-- No Message ready to be received
-- No Message ready to be received
Product: Socker Ball
-- No Message ready to be received
productChannel has been closed
All values received
```

### === Отправка без блокировки, отправка в несколько каналов ===
`select` можно использовать для отправки в канал, или в несколько накалов:
```
select {
    case ch1 <- msg:
	    fmt.Println('test 1') // <-- Строго говоря, это можно опустить, без него тоже будет норм
    case ch2 <- msg:
	    fmt.Println('test 12)
}
```
