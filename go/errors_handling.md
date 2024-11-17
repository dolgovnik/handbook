[[../index.md|<-- Go Back]]

# == Ошибки ==
__Go__ предоставляет интерфейс, который позволяет работать/обрабатывать различные __ошибки__ (__О__)
```
type error interface {
    Error() string
}
```
Пример:
```
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

type ProductSlice []*Product

var Products = ProductSlice{
    {"Kayak", "Watersports", 279},
    {"Lifejacket", "Watersports", 49.5},
    {"Socker Ball", "Soccer", 19.5},
    {"Corner Flags", "Soccer", 34.95},
    {"Stadium", "Soccer", 79500},
    {"Thinking Cap", "Chess", 16},
    {"Unsteady Chair", "Chess", 75},
    {"Bling-bling King", "Chess", 1200},
}

func ToCurrency(val float64) string {
	return "$" + strconv.FormatFloat(val, 'f', 2, 64)
}

####################
###operatiions.go###
####################

package main

type CategoryError struct { // <-- определяет неэкспортуруемое поле и метод соответствующий
                            // интерфейсу error
	requestedCategory string
}

func (e *CategoryError) Error() string {    // <-- вот этот метод
	return "Category" + e.requestedCategory + "does not exist"
}

func (slice ProductSlice) TotalPrice(category string) (total float64, err *CategoryError){
	productCount := 0
	for _, p := range slice {
		if (p.Category == category){
			total += p.Price
			productCount++
		}
	}
	if (productCount == 0) {
	    err = &CategoryError{requestedCategory: category} // <-- заполняем ошибку если
		                                                  // нет результата
	}
	return    // <-- возвращает два значения: результат и ошибку
}

#############
###main.go###
#############

package main

import "fmt"

func main(){
    var categories = []string { "Watersports", "Chess", "Running" } 

	for _, cat := range categories {
		total, err := Products.TotalPrice(cat)
		if (err == nil) {                                 // <-- проверяем есть ли ошибка
		    fmt.Println(cat, "Total:", ToCurrency(total))
		} else {
			fmt.Println("Category", cat, "does not exist")
		}
	}
}
```
Результат выполнения:
```
Watersports Total: $328.50
Chess Total: $1291.00
Category Running does not exist
```

##  == Сообщение об ошибке через канал ==
Удобно передавать результат выполнения и __О__ в одной __С__, а в вызывающем коде обрабатывать эти ошибки:
```
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

type ProductSlice []*Product

var Products = ProductSlice{
    {"Kayak", "Watersports", 279},
    {"Lifejacket", "Watersports", 49.5},
    {"Socker Ball", "Soccer", 19.5},
    {"Corner Flags", "Soccer", 34.95},
    {"Stadium", "Soccer", 79500},
    {"Thinking Cap", "Chess", 16},
    {"Unsteady Chair", "Chess", 75},
    {"Bling-bling King", "Chess", 1200},
}

func ToCurrency(val float64) string {
	return "$" + strconv.FormatFloat(val, 'f', 2, 64)
}

####################
###operatiions.go###
####################

package main

type CategoryError struct {
	requestedCategory string
}

func (e *CategoryError) Error() string {
	return "Category" + e.requestedCategory + "does not exist"
}

type ChannelMessage struct {    // <-- структура для передачи результата и ошибки через КН
	Category string
	Total float64
	*CategoryError
}

func (slice ProductSlice) TotalPrice(category string) (total float64, err *CategoryError){
	productCount := 0
	for _, p := range slice {
		if (p.Category == category){
			total += p.Price
			productCount++
		}
	}
	if (productCount == 0) {
	    err = &CategoryError{requestedCategory: category}
	}
	return
}

// Метод для асинхронной работы и отправки результатов через канал
func (slice ProductSlice) TotalPriceAsync (categories []string, channel chan<- ChannelMessage) {
	for _, c := range categories {
		total, err := slice.TotalPrice(c)
		channel <- ChannelMessage{
				       Category: c,
					   Total: total,
					   CategoryError: err,
				   }
	}
	close(channel)
}

#############
###main.go###
#############

package main

import "fmt"

func main(){
    var categories = []string { "Watersports", "Chess", "Running" } 

	channel := make(chan ChannelMessage, 10)

	go Products.TotalPriceAsync(categories, channel)  // запускаем ГР

	for msg := range channel {                        // Читаем сообщения из канала
	    if msg.CategoryError == nil {
    		fmt.Println("Category:", msg.Category, "Total:", msg.Total)
    	} else {
    		fmt.Println("Category:" , msg.Category, "(no such category)")
    	}
	}
}
```
Результат выполнения:
```
Category: Watersports Total: 328.5
Category: Chess Total: 1291
Category: Running (no such category)
```

## == Функции для обработки ошибок ==
Можно использовать __встроенные__ __Ф__ для генерации __О__
```
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

type ProductSlice []*Product

var Products = ProductSlice{
    {"Kayak", "Watersports", 279},
    {"Lifejacket", "Watersports", 49.5},
    {"Socker Ball", "Soccer", 19.5},
    {"Corner Flags", "Soccer", 34.95},
    {"Stadium", "Soccer", 79500},
    {"Thinking Cap", "Chess", 16},
    {"Unsteady Chair", "Chess", 75},
    {"Bling-bling King", "Chess", 1200},
}

func ToCurrency(val float64) string {
	return "$" + strconv.FormatFloat(val, 'f', 2, 64)
}

####################
###operatiions.go###
####################

package main

import "errors"

type ChannelMessage struct {
	Category string
	Total float64
	CategoryError error
}

func (slice ProductSlice) TotalPrice(category string) (total float64, err error){
	productCount := 0
	for _, p := range slice {
		if (p.Category == category){
			total += p.Price
			productCount++
		}
	}
	if (productCount == 0) {
	    err = errors.New("no such category")
	}
	return
}

func (slice ProductSlice) TotalPriceAsync (categories []string, channel chan<- ChannelMessage) {
	for _, c := range categories {
		total, err := slice.TotalPrice(c)
		channel <- ChannelMessage{
				       Category: c,
					   Total: total,
					   CategoryError: err,
				   }
	}
	close(channel)
}

#############
###main.go###
#############

package main

import "fmt"

func main(){
    var categories = []string { "Watersports", "Chess", "Running" } 

	channel := make(chan ChannelMessage, 10)

	go Products.TotalPriceAsync(categories, channel)

	for msg := range channel {
	    if msg.CategoryError == nil {
    		fmt.Println("Category:", msg.Category, "Total:", msg.Total)
    	} else {
    		fmt.Println("Category:" , msg.Category, "(no such category)")
    	}
	}
}
```
Результат выполнения:
```
Category: Watersports Total: 328.5
Category: Chess Total: 1291
Category: Running (no such category)
```

Для красивого форматирования можно использовать пакет `fmt`:
```
err = fmt.Errorf("Cannot find category: %v", category")
```

# = Паника =
Некоторые __О__ не могут быть исправлены, они должны вызывать __панику__ - __Ф__ `panic`
Когда вызывается `panic`, то выполнение _объемлющей_ __Ф__ останавливается, выполняются все __Ф__ `defer` - __паника__ поднимается выше по _стеку вызовов_ и прерывает выполнение всех __Ф__ и вызывает `defer`

__!!!__ Нормальной практикой является предоставлять две версии __Ф__: одна возвращает __ошибку__, а вторая вместо ошибки вызывает __панику__:
`regexp.Compile` - возвращает __ошибку__
`regexp.MustCompile` - вызывает __панику__

```
#############
###main.go###
#############

package main

import "fmt"

func main(){
    var categories = []string { "Watersports", "Chess", "Running" } 

	channel := make(chan ChannelMessage, 10)

	go Products.TotalPriceAsync(categories, channel)

	for msg := range channel {
	    if msg.CategoryError == nil {
    		fmt.Println("Category:", msg.Category, "Total:", msg.Total)
    	} else {
    		panic(msg.CategoryError)    // <-- Вызываем панику, можно передать любое значение
    	}
	}
}
```
Результат выполнения:
```
> go run .
Category: Watersports Total: 328.5
Category: Chess Total: 1291
panic: no such category

goroutine 1 [running]:
main.main()
	/home/nikolay/Y/go_study/errorHandling/main.go:16 +0x289
exit status 2
```

## == Восстановление после паники == 
__Go__ предоставляет __Ф__ `recover`, которая при вызове не позволит __панике__ подняться выше
__!!!__ `recover` должна выполняться в `defer`, удобно дельть это с помощью _анонимной_ __Ф__
```
#############
###main.go###
#############

package main

import "fmt"

func main(){


   defer func(){                          // <-- определяем анонимную Ф, оборачиваем в defer
       if arg := recover(); arg != nil {  // <-- вызываем recover
			if err, ok := arg.(error); ok {        // <-- действуем в зависимости от того
			                                       // какое значение вернет recover()
				fmt.Println("Error:", err.Error())
		    } else if str, ok := arg.(string); ok {
		        fmt.Println("Message:", str )
		    } else {
			    fmt.Println("Panic recovered")
		    }
	   }
   }()                                   // <-- вызов анономной Ф

    var categories = []string { "Watersports", "Chess", "Running" } 

	channel := make(chan ChannelMessage, 10)

	go Products.TotalPriceAsync(categories, channel)

	for msg := range channel {
	    if msg.CategoryError == nil {
    		fmt.Println("Category:", msg.Category, "Total:", msg.Total)
    	} else {
    		panic(msg.CategoryError)
    	}
	}
}
```
Результат выполнения:
```
Category: Watersports Total: 328.5
Category: Chess Total: 1291
Error: no such category    // <-- recover вернул error и мы ее обработали, паники нет
```

Можно вызвать __панику__ после обработки  - ну мало ли?
```
#############
###main.go###
#############

package main

import "fmt"

func main(){


   defer func(){
       if arg := recover(); arg != nil {
			if err, ok := arg.(error); ok {
				fmt.Println("Error:", err.Error())
				panic(err)
		    } else if str, ok := arg.(string); ok {
		        fmt.Println("Message:", str )
		    } else {
			    fmt.Println("Panic recovered")
		    }
	   }
   }()

    var categories = []string { "Watersports", "Chess", "Running" } 

	channel := make(chan ChannelMessage, 10)

	go Products.TotalPriceAsync(categories, channel)

	for msg := range channel {
	    if msg.CategoryError == nil {
    		fmt.Println("Category:", msg.Category, "Total:", msg.Total)
    	} else {
    		panic(msg.CategoryError)
    	}
	}
}
```
Результат выполнения:
```
Category: Watersports Total: 328.5
Category: Chess Total: 1291
Error: no such category
panic: no such category [recovered]
	panic: no such category

goroutine 1 [running]:
main.main.func1()
	/home/nikolay/Y/go_study/errorHandling/main.go:12 +0x1b3
panic({0x48c0a0?, 0xc000014070?})
	/usr/local/go/src/runtime/panic.go:770 +0x132
main.main()
	/home/nikolay/Y/go_study/errorHandling/main.go:31 +0x2c5
exit status 2
[22:09:01] ~/Y/go_study/errorHandling 
```

## == Паника в горутинах ==
__Паника__ поднимается вверх по стеку только до вершины текущий __ГР__. Т.е. __паники__ должны восстанавливаться внутри кода, выполняемого __ГР__
```
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

type ProductSlice []*Product

var Products = ProductSlice{
    {"Kayak", "Watersports", 279},
    {"Lifejacket", "Watersports", 49.5},
    {"Socker Ball", "Soccer", 19.5},
    {"Corner Flags", "Soccer", 34.95},
    {"Stadium", "Soccer", 79500},
    {"Thinking Cap", "Chess", 16},
    {"Unsteady Chair", "Chess", 75},
    {"Bling-bling King", "Chess", 1200},
}

func ToCurrency(val float64) string {
	return "$" + strconv.FormatFloat(val, 'f', 2, 64)
}

####################
###operatiions.go###
####################

package main

import "errors"

type ChannelMessage struct {
	Category string
	Total float64
	CategoryError error
}

func (slice ProductSlice) TotalPrice(category string) (total float64, err error){
	productCount := 0
	for _, p := range slice {
		if (p.Category == category){
			total += p.Price
			productCount++
		}
	}
	if (productCount == 0) {
	    err = errors.New("no such category")
	}
	return
}

func (slice ProductSlice) TotalPriceAsync (categories []string, channel chan<- ChannelMessage) {
	for _, c := range categories {
		total, err := slice.TotalPrice(c)
		channel <- ChannelMessage{
				       Category: c,
					   Total: total,
					   CategoryError: err,
				   }
	}
	close(channel)
}

#############
###main.go###
#############

package main

import "fmt"

type CountCategoryMessage struct {
	Category string
	Count int
}


func processCategories(categories []string, outChan chan<- CountCategoryMessage){
	defer func(){            // <-- Восстановленеи после паники
	                         // но выполнение Ф не продолжится до конца
              if arg := recover(); arg != nil {
                  fmt.Println(arg)
			  }
	      }()

    channel := make(chan ChannelMessage, 10)
    go Products.TotalPriceAsync(categories, channel) 
	for message := range channel {
		if message.CategoryError == nil {
            outChan <- CountCategoryMessage {
                           Category: message.Category,
						   Count: int(message.Total),
					   }
		} else {
            panic(message.CategoryError)    // <-- Паника, если вернули ошибку
		}
	}
	close(outChan) // <-- После восстаноления после паники этот кусок НЕ будет выполняться,
	               // канал так и останется открытым и будет дедлок.
}

func main(){
    categories := []string{"Watersports", "Chess", "Running"}
    channel := make(chan CountCategoryMessage)
	
	go processCategories(categories, channel)

	for message := range channel{
        fmt.Println("Category:", message.Category, "Total:", message.Count)
	}
}
```
Результат выполнения:
```
Category: Watersports Total: 328
Category: Chess Total: 1291
no such category
fatal error: all goroutines are asleep - deadlock! // <-- Восстановление после паники НЕ
                                                   // возбновляет выполнение функции и поэтому
												   // канал не закроется и будет дедлок
goroutine 1 [chan receive]:
main.main()
	/home/nikolay/Y/go_study/errorHandling/main.go:39 +0x1c8
exit status 2
```

__!!!__ Чтобы избежать подобного _дедлока_, необходимо передавать через канал инфу ошибке и о статусе канала.
Пример:	
```
> cat main.go
package main

import "fmt"

type CountCategoryMessage struct {
	Category string
	Count int
	TerminalError interface{}    // <-- добавляем поле в структуру для передачи информации об
	                             // состоянии канала
}

func processCategories(categories []string, outChan chan<- CountCategoryMessage){
	defer func(){
              if arg := recover(); arg != nil {
                  fmt.Println(arg)
				  outChan <- CountCategoryMessage{
						         TerminalError: arg, // <-- заполняем поле с ошибкой, если
								                     // была паника и надо закрыть канал
						     }
			  }
			  close(outChan)                         // <-- закрываем канал во после обработку
			                                         // паники
	      }()

    channel := make(chan ChannelMessage, 10)
    go Products.TotalPriceAsync(categories, channel) 
	for message := range channel {
		if message.CategoryError == nil {
            outChan <- CountCategoryMessage {
                           Category: message.Category,
						   Count: int(message.Total),
					   }
		} else {
            panic(message.CategoryError)
		}
	}
	close(outChan)
}

func main(){
    categories := []string{"Watersports", "Chess", "Running"}
    channel := make(chan CountCategoryMessage)
	
	go processCategories(categories, channel)

	for message := range channel {
		if message.TerminalError == nil {    // <-- проверяем если ли ошибка
            fmt.Println("Category:", message.Category, "Total:", message.Count)
        } else {
			fmt.Println("Terminal error has occured")
		}
	}
}
```
Результат выполнения:
```
Category: Watersports Total: 328
Category: Chess Total: 1291
no such category
Terminal error has occured  // <-- программа штатно закончила работу
```
