[[index.md|<-- Go Back]]

# = Json =
Пакет `encoding/json` обеспечивает поддержку кодирования и декодирования данных в `JSON`

Пакет предоставляет следующие __Ф__ конструктора:
`NewEncoder(writer)` - возвращает `Encoder`, который можно использовать для кодирования данных в `JSON` и записи их в указанный `Writer`
`NewDecoder(reader)` - возвращает `Decoder` который можно использовать для чтения данных `JSON` из указанного  `Reader`

Можно не испльзовать `Reader` и `Writer`, для этого применяются следующие __Ф__:
`Marshal(value)` - кодирует значение в `JSON`, результат - __срез байт__ и __error__
`Unmarshal(byteSlice, val)` - анализирует `JSON` из `byteslice` и присваивает результат переменной `val`

Пример:
```
#############
###main.go###
#############

package main

import (
	"encoding/json"
	"fmt"
	"strings"
)

func main() {
	var b bool = true
	var s string = "Hello!"
	var fval float64 = 99.99
	var ival int = 1500
    var prt *int = &ival

	var writer strings.Builder
	encoder := json.NewEncoder(&writer)

    for _, val := range []interface{}{b, s, fval, ival, prt} {
		encoder.Encode(val)
	}
	fmt.Println(writer.String())
}

```
Результат выполнения:
```
true
"Hello!"
99.99
1500
1500

```

__!!!__ байтовые __массивы__ и байтоыйе __срезы__ кодируются по разному
__!!!__ при кодировании __структур__ __встроенные__ поля _продвигаются_ вперед
__!!!__ c помощью _тегов структуры_ можно настроить способ ее кодирования в `JSON`
__!!!__ при кодировании __интерфейсов__ кодируется __динамический тип__
