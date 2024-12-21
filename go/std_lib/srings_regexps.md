[[index.md|<-- Go Back]]

# = Модуль strings =
`strings` отвечает за работу со стороками: поиск по строке, изменение регистра, проверка регистра, поиск индекса, различные проверки с использование пользовательских функций, разбиение сток по полям, обрезка строк, изменение строк, генерация строк

```
#############
###main.go###
#############
package main

import (
	"fmt"
	"strings"
)

func main() {
    product := "Kayak"

	fmt.Println("Product:", product)
	fmt.Println("Contains:", strings.Contains(product, "yak"))
	fmt.Println("ContainsAny:", strings.ContainsAny(product, "abc"))
	fmt.Println("ContainsRune:", strings.ContainsRune(product, 'K'))
	fmt.Println("EqualFold:", strings.EqualFold(product, "KAYAK"))
	fmt.Println("HasPrefix:", strings.HasPrefix(product, "Kay"))
	fmt.Println("HasSuffix:", strings.HasSuffix(product, "yak"))
}
```
Результат выполнения:
```
Product: Kayak
Contains: true
ContainsAny: true
ContainsRune: true
EqualFold: true
HasPrefix: true
HasSuffix: true
```

## == Builder ==
Позволяет _строить_ строки гораздо эффективней чем конкатенация.
Общий подход следующий:
    - Создать `Builder`
	- Составить строку с помощью __Ф__ `WriteString`, `WriteRune` и `WriteByte`
	- Получить строку с помощью `String`
Пример:
```
#############
###main.go###
#############

package main

import (
	"fmt"
	"strings"
)

func main() {
    text := "It was a boat. A small boat"
	var builder strings.Builder

	for _, sub := range strings.Fields(text){
	    if (sub == "small"){
			builder.WriteString("very ")
		}
        builder.WriteString(sub)
		builder.WriteRune(' ')
	}
	fmt.Println(builder.String())
}
```
Результат выполнения:
```
It was a boat. A very small boat 
```

# = Модуль regexp =
Пакет `regexp` - обеспечивает поддержку __регулярных выражений__
```
#############
###main.go###
#############

package main

import (
	"fmt"
	"regexp"
)

func main() {
		descriptions := [2]string{ "A boat for one person", "Car for one person"}
		for _, d := range descriptions{
				match, err := regexp.MatchString("[A-z]oat", d)
				if (err==nil){
						fmt.Println("Match:", match)
				} else {    // <-- ошибка если шаблон не может быть обработан
						fmt.Println("Error:", err)
				}
		}
}
```
Результат выполнения:
```
Match: true
Match: false
```

Можно находить подстроки:
```
#############
###main.go###
#############
package main

import (
	"fmt"
	"regexp"
)

func main() {
	pattern := regexp.MustCompile("K[a-z]{4}|[A-z]oat")
	description := "Kayak. A boat for one person."

	firstMatch := pattern.FindString(description)
    allMatches := pattern.FindAllString(description, -1)

    fmt.Println("First Match:", firstMatch)

    for i, match := range allMatches {
        fmt.Println("Match",i, "=", match)
    }
}
```
Результат выполнения:
```
First Match: Kayak
Match 0 = Kayak
Match 1 = boat
```

`pattern.FindStringSubmatch` позволяет делать _бекреференс_
_подвыражения_ можно именовать, например так:
```
pattern := regexp.MustCompile("A (?P<type[A-z]*>) for (?P<capacity>[A-z]*) person")
```

Есть методы для замены подстрок: `REplaceAllStrings` и т.п.
