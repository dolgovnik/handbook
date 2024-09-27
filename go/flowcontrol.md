[[../index.md|<--Go Back]]

# = Flowcontrol =

Функция __main__ - это __точка входа__, с нее начинается выполнение программы

## == IF ==
__!!!__ `{` - обязательно в той строке где и условие
__!!!__ в выражениях можно применять круглые скобки, как в примерах ниже
__!!!__ У каждой ветки в операторе `if` своя область видимости => можно использовать одинаковые имена переменных в каждой ветке

Пример `if-else if-else`:
```
package main

import "fmt"

func main() {
	kayakPrice := 275.00
    if (kayakPrice > 500) {
	    fmt.Println("Price is greater than 500")
	} else if (kayakPrice < 100) {
	    fmt.Println("Price is less than 100")
	} else {
	    fmt.Println("Price not matched by earlier expressions")
    }
}
```

### === Оператор инициализации ===
Можно использовать оператор инициализации который будет выполняться перед оператороам `if`
Пример:
```
package main

import (
	"fmt"
	"strconv"
)

func main() {
	priceString := "275"

	if kayakPrice, err := strconv.Atoi(priceString); (err == nil) {
		fmt.Println("Price:", kayakPrice)
	} else {
		fmt.Println("Error:", err)
    }
}
```
Синтаксис:
`if kayakPrice, err := strconv.Atoi(priceString); (err == nil) {...`
Область видимости для переменных __kayakPrice__, __err__- весь оператор `if`

## == FOR ==
Оператор `for` нужен для создания циклов
Пример:
```
package main

import (
	"fmt"
)

func main() {
	counter := 0
	for {
		fmt.Println("Counter:", counter)
		counter++
		if (counter > 3) {
			break
		}
	}
}

```

Условие можно включить в цикл:
```
counter := 0
for (counter <= 3) {...
```

Можно использовать операторы _инициализации_ и _пост-оператор_
```
for counter :=0; counter <= 3; counter++ {...
```
_Инициализация_ выполняется один раз, затем оценка, затем действия в скобках, затем _пост-оператор_, затем оценка

Можно испльзовать `break` и `continue`

### === Перечисление последовательностей ===
Пример:
```
package main

import (
	"fmt"
)

func main() {
	product := "Kayak"

    for index, character := range product {
        fmt.Println("Index:", index, "Character:", string(character))		
	}
}
```
`character` имеет тип `rune`

__!!!__ Можно получить только `index` или `character` c помощью `_` - пустого идентификатора

## == SWITCH ==
__Это краткая альтернатива оператору `if`__

Пример:
```
package main

import (
	"fmt"
)

func main() {
	product := "Kayak"

    for index, character := range product {
		switch (character) {
			case 'K':
				fmt.Println("K at position", index)
			case 'k':
				fmt.Println("k at position", index)
			case 'y':
				fmt.Println("y at position", index)
			default:
				fmt.Println("Character", string(character), "at poosition", index)
		}
	}
}
```


__!!!__ можно использовать `break`
```
package main

import (
	"fmt"
)

func main() {
	product := "Kayak"

    for index, character := range product {
		switch (character) {
			case 'K', 'k':
				if (character == 'k') {
				    fmt.Println("k at position", index)
				break
				}
				fmt.Println("K at position", index)
			case 'y':
				fmt.Println("y at position", index)
		}
	}
}
```

__!!!__ `fallthrough` продолжает выполнение операторов из следующего оператора `case`:
```
package main

import (
	"fmt"
)

func main() {
	product := "Kayak"

    for index, character := range product {
		switch (character) {
			case 'K':
				fmt.Println("Uppercase character")
				fallthrough
			case 'k':
				fmt.Println("k at position", index)
			case 'y':
				fmt.Println("y at position", index)
		}
	}
}
```
Результат выполнения:
```
Uppercase character
k at position 0
y at position 2
k at position 4

```

__!!!__ в операторе `switch` можно использовать оператор _инициализации_
```
switch val := counter / 2; val {
    case 2, 3, 5:
	...
```

__!!!__ Можно так - значение оператора `switch` опущено, значит в `case` должны быть условия:
```
switch {
    case counter == 0:
	    .....
	case counter < 3:
	    ....
```

__!!!__ `switch` можно использовать для различения типов данных

## == GOTO ==
Пример:
``` 
target: fmt.Println("SMTH")
if (counter < 5) {
    goto target
}
```

## == Sensetive shit ==

