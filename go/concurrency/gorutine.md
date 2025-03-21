[[concurrency.md|<-- Go Back]]

# = Gorutime =
__Горутина__ (__ГР__) - _облегченный поток_ созданный средой выполнения __Go__
Любая программа на __Go__ использует минимум одну __ГР__ - в ней выполняется код __Ф__ `main` - от первого оператора до последнего, по очереди.
__ГР__ выполняет операторы _синхронно_ - ожидает завершения оператора, перед тем как перейти к следующему.

## == Стек ==
Размер стека __ГР__ - 2Кбайта - хранит локальные переменные функций. Этот стек может расти или уменьшаться. Максимальный размер 1Гбайт

## == Планирование ГР ==
Среда выполнения __GO__ содержит свой планировщик, отдельный от планировщика ОС. Он использует свой метод планирования __m:n-планирование__ - планирует выполнение __m__ __ГР__ на __n__ потоках ОС.
Планировщик вызывается неявно некоторыми конструкциями языка Go.
Параметр __GOMAXPROCS__ определяет кол-во потоков ОС, которые могут одновременно активно выполнять код __Go__, по умолчанию его значение равно кол-ву процессоров на компе.

## == Создание дополнительных Gorutine ==
Можно создать дополнительную __ГР__, которая будет выполнять код одновременно с _основной_ __ГР__
Пример:
```
###################
###operations.go###
###################
.
.
.
func CalcStoreTotal(data ProductData) {
	var storeTotal float64
	for category, group := range data {
		go group.TotalPrice(category)     // <-- создание горутины для КАЖДОГО вызова TotalPrice
	}
	fmt.Println("Total:", toCurrency(storeTotal))
}
.
.
.
```
__ГР__ создается ключевым словом `go` за которым следует __Ф__ или __М__ который должен выполняться _асинхронно_. Т.е. __Ф__ будет выполняться в другой __ГР__, в другом 'потоке' и __ОДНОВРЕМЕННО__ c _основной_ __ГР__
Результат выполнения:
```
main function started
Total: $0.00
main function complete
```
Это происходит потому что _основная_ __ГР__ выполняет все операторы в `main` до того как выполнятся дополнительные __ГР__. Если замедлить выполнение `main`, то дополнительные __ГР__ успеют выполнить `TotalPrice`
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
	fmt.Println("main function started")
	CalcStoreTotal(Products)
	time.Sleep(time.Second * 5)    // <-- замедляем выполнение main, теперь ГР в которых
	                               // выполняется TotalPrice успевают выполниться до конца
	fmt.Println("main function complete")
}
```
Результат выполнения:
```
main function started
Total: $0.00
Socker product: 19.5
Watersports product: 279
Watersports product: 49.95
Socker product: 34.95
Watersports subtotal: $328.95
Socker product: 79500
Socker subtotal: $79554.45
Chess product: 16
Chess product: 75
Chess product: 1200
Chess subtotal: $1291.00
main function complete
```
__!!!__ Можно замедлить и `TotalPrice`, тогда станет видно что __ГР__  выполняются _одновременно_
```
func (group ProductGroup) TotalPrice(category string) (total float64) {
		for _, p := range group {
			total += p.Price
			fmt.Println(category, "product:", p.Price)
			time.Sleep(time.Millisecond * 100)
		}
		fmt.Println(category, "subtotal:", toCurrency(total))  // <-- Заамееедляяяеееем
		return
}
```
Результат выполнения - категории перечисляются в разном порядке:
```
main function started
Total: $0.00
Watersports product: 279
Chess product: 16
Socker product: 19.5
Socker product: 34.95
Watersports product: 49.95
Chess product: 75
Watersports subtotal: $328.95
Socker product: 79500
Chess product: 1200
Chess subtotal: $1291.00
Socker subtotal: $79554.45
main function complete
```

## == Возврат результата из Горутины ==
Получение результата из __Ф__, которая выполняется _асинхронно_ требует координации между _вызывающей_ и _вызываемой_ __ГР__
Для решения этой проблемы в __Go__ созданы [[channel.md|каналы]] (__КН__)
