[[../../index.md|<-- Go Back]]

# = Concurrency =

* [[gorutine.md|Gorutine]] - __Горутина__ (__ГР__) - _облегченный поток_ созданный средой выполнения __Go__
* [[channel.md|Channel]] - __Канал__ (__КН__) - применяется для передачи данных между __ГР__

Код:
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

type ProductGroup []*Product    // <-- Псевдоним типа, для среза из *Product

type ProductData = map[string]ProductGroup //<-- Псевдоним типа, для хранения содержимого
                                           // содержимого категорий:
										   // "Watersports":{"Kayak", Lifejacket"}

var Products = make(ProductData)   // <--Создаем пустую переменную типа ProductData

func toCurrency(val float64) string {
	return "$" + strconv.FormatFloat(val, 'f', 2, 64)
}

func init() {                       // <-- Ф инициализации
	for _, p := range ProductList {
        if _, ok := Products[p.Category]; ok { 
			Products[p.Category] = append(Products[p.Category], p)  // <-- Наполняем переменную
			                                                        // Products
		} else {
		    Products[p.Category] = ProductGroup { p }
	    }
	}
}

###################
###operations.go###
###################

package main

import "fmt"

func CalcStoreTotal(data ProductData) {
	var storeTotal float64
	for category, group := range data {
		storeTotal += group.TotalPrice(category)
	}
	fmt.Println("Total:", toCurrency(storeTotal))
}

func (group ProductGroup) TotalPrice(category string) (total float64) { //<-- М для псевдонима
                                                                        //из products.go
		for _, p := range group {
			total += p.Price
			fmt.Println(category, "product:", p.Price)
		}
		fmt.Println(category, "subtotal:", toCurrency(total))
		return
}

#############
###main.go###
#############

package main

import "fmt"

func main() {
	fmt.Println("main function started")
	CalcStoreTotal(Products)
	fmt.Println("main function complete")
}
```
Результат выполнения:
```
main function started
Watersports subtotal: $328.95
Socker subtotal: $79554.45
Chess subtotal: $1291.00
Total: $81174.40
main function complete
```
