[[index.md|<--Go Back]]

# = Function Types =
__Ф__ имеют тип данных, следовательно они могут быть назначены переменным и использоваться в качестве параметров других __Ф__

Пример:
```
package main

import "fmt"

func calcWithTax(price float64) float64 {
	return price + (price *0.2)
}

func calcWithoutTax(price float64) float64 {
	return price
}

func main() {
	products := map[string]float64 {
		"Kayak": 275,
		"Lifejacket": 48.95,
	}

	for product, price := range products {
		var calcFunc func(float64) float64     // <-- это тип Ф: указывает тип принимаемых и возвращаемых параметров
		if (price > 100){
			calcFunc = calcWithTax
		} else {
			calcFunc = calcWithoutTax
		}
		totalPrice := calcFunc(price)
	    fmt.Println("Product:", product, "Price:", totalPrice)	
	}
}

```
Результат выполнения:
```
Product: Kayak Price: 330
Product: Lifejacket Price: 48.95
```
__Тип Ф__ указывается так: `func(<типы параметров>)(<типы резульльтатов>)` - это _сигнатура функции_

__!!!__ Можно узнать, была ли __Ф__ присвоена переменной, так как _нулевое значение_ для типов __Ф__ равно `nil`
```
calcFunc == nil
```

__!!!__ __Ф__ может быть параметром другой __Ф__
__!!!__ __Ф__ может быть результатом другой __Ф__
```
func selectCalculator(price float64) func(float64) float64 {
    if (price > 100){
	    return calcWithTax
	}
	return calcWithoutTax
}
```

## == Псевдонимы функциональных типов ==
Простое использование __функциональных типов__ ведет к усложнению кода. Чтобы этого избежать, __Go__ поддерживает __псевдонимы типов__.
Краткий пример:
```
type calcFunc func(float64) float64
...
func printPrice(product string, price float64, calculator calcFunc) {
...
}
func selectCalculator(price float64) calcFunc {
...
}
```

__!!!__ `type` также используется для создания __пользовательских типов__

## == Литеральный синтаксис ==
Литеральный синтаксис позволяет определять функции так, чтобы они были специфичны для области кода:
```
var withTax calcFunc = func (price float64) float64{
    return price + (price * 0.2)
}
```
Фактически получается _анонимная_ ***Ф***
__!!!__ можно использовать _короткий синтаксис_ `:=`
__!!!__ область действия такой переменной - это тот блок кода где она определена

## == Непосредственное использование значений Ф ==
Можно делать так:
```
...
return func (price float64) float64 {
    return price + (price*0.2)
}
```
или таким же образом передавать __Ф__ как аргумент другой __Ф__

# = Замыкание ==
Это __Ф__ и присоединенные к ней данные
Пример:
```
type calcFunc func(float64) float64
func priceCalcFactory(threshold float64, rate float64) caclFunc {
    return func(price float64) float64 {
	    if (price > threshold) {
		    return price + (price * rate)
		}
		return price
	}
}
```
`priceCalcFactory` - _фабричная_ ***Ф***
__!!!__ __Замыкание__ заключается в том, что `threshold` и `rate` попадают в _возвращаемую_ ***Ф*** из окружающего кода - из _фабричной_ ***Ф***.

## == Оценка замыкания ==
__!!!__ Значения из окружающего кода, оцениваются каждый раз когда __Ф__ вызывается. Т.е. если в предыдущем примере вынести их из _фабричной_ ***Ф***, то они могут быть изменены и, следовательно, это окажет влияние на следующие вызовы __Ф__

__!!!__ Чтобы указанный выше эффект был более очевидным - можно использовать __указатели__

__!!!__ Если такое поведение не нужно, то можно скопировать значение внутри _фабричной_ ***Ф*** перед возвращением __Ф-результата__. Или можно добавить параметр в _фабричную_ ***Ф***
