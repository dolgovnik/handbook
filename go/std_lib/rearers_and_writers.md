[[index.md|<--Go back]]

# = Общая инфа =

`Reader` и `Writer` - это __интерфейсы__ (из пакета `io`), которые предоставляют абстрактные способы чтения и записи данных _без привязки_ к источнику данных. Как правило, они реализуются для __указателей__

# = Чтение - Reader =
__И__ `Reader` предоставляет единственный __М__ - `Read(byteSlice)` - считывает данные в указанный `[]byte` и возвращает кол-во прочитанных байтов (`int`, `err`)
Пример:
```
################
###printer.go###
################

package main

import "fmt"

func Printfln(template string, values ...interface{}) {
	fmt.Printf(template + "\n", values...)
}

#############
###main.go###
#############

package main

import (
	"io"
	"strings"
)

func processData(reader io.Reader) {    // <-- принимает Интерфейс Reader
	b := make([]byte, 2)
	for {                               // <-- повторяем чтение в цикле
		count, err := reader.Read(b)    // <-- используем единственный метод Read
		                                // читаем по 2 байта - именно такой размер Среза
		if (count > 0) {
			Printfln("Read %v bytes: %v count", count, string(b[0:count]))
		}
		if err == io.EOF {    // <-- Если ошибка, то уходим
			break	
		}
	}
}

func main() {
	r := strings.NewReader("Kayak")    // <-- Новый Ридер, который читает из строки,
	                                   // т.е. строку "Kayak"
	processData(r)
}
```
Результат выполнения:
```
Read 2 bytes: Ka count
Read 2 bytes: ya count
Read 1 bytes: k count
```

# = Запись - Writer =
`Writer` определяет единственный __М__ `Write(byteSlice)` - записывает данные из указанного `byte` среза, возвращает кол-во записанных байтов (`int`, `err`): ошибка будет если кол0во записанных байтов меньше длинны среза
Пример:
```
################
###printer.go###
################

package main

import "fmt"

func Printfln(template string, values ...interface{}) {
	fmt.Printf(template + "\n", values...)
}

#############
###main.go###
#############

package main

import (
	"io"
	"strings"
)

func processData(reader io.Reader, writer io.Writer) {
	b := make([]byte, 2)
	for {
		count, err := reader.Read(b)
		if (count > 0) {
			writer.Write(b[0:count])
			Printfln("Read %v bytes: %v count", count, string(b[0:count]))
		}
		if err == io.EOF {
			break	
		}
	}
}

func main() {
	r := strings.NewReader("Kayak")
	var builder strings.Builder    // <-- реализует интерфейс io.Writer
	processData(r, &builder)       // <-- как правило методы REader и Writer реализуются
	                               // для указателей => не создается копия.
	Printfln("Strings builder content: %v", builder.String())
}

```
Результат выполнения:
```
Read 2 bytes: Ka count
Read 2 bytes: ya count
Read 1 bytes: k count
Strings builder content: Kayak
```

# = Другие функции =
Пакет `io` содержит и другие функции для чтения и записи: `Copy`, `ReadAll`, `Pipe()`

# = Буферизация данных =
Пакет `bufio` позволяет добавить _буфер_ для чтения и записи.
В _буфер_ считывается большой объем данных для обслуживания более мелких _чтений_ => меньше накладных расходов.
