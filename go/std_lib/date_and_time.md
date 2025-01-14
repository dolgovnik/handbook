[[index.md|<-- Go Back]]

# = Даты и Время =
Пакет `time` предоставляет __тип__ `Time`. Есть разные __Ф__, которые создают значения `Time`: `Now`, `Date`, `Unix`
Доступ к _компонентам_ `Time` осуществляется с помощью методов: `Date`, `Clock`, `Year` и т.п.
`time` также определяет __типы__ `Month` и `Weekday`

__!!!__ При работе с _датами_ используется предопределенный _layout_ (эталонное время) - 02.01.2006 15:04:05 (MST)

__!!!__ `Duration` - это псевдоним типа `int64`, используется для представления определенного кол-ва миллисекунд: `Hour`, `Minute`, `Second` и т.п.

# == Разбор значений времени и строк ==
`time.Parse` и `time.ParseInLocation` анализируют строки в соответствии с _layout_ и создают `Time` 
# == Управление значениями времени ==
`time` предоставляет методы для работы со значениями `Time`: `Add`, `Before`, `In`, `Equal` и т.п. часть изних использует __тип__ `Duration` 
Есть методы `Since` и `Until`
`ParseDuration` - парсит `Duration` из строк.

# = Функции времени для горутин и каналов =
`Sleep(duration)` - приостанавливает текущую __ГР__
`AfterFunc(duration, func)` - выполняет указанную __Ф__ по истечении указанного времени, возвращает `*Timer` у которого есть метод `Stop` для отмены выполнения
`After(duration)` - возвращает __канал__, который блокируется на указанное время, а затем возвращает `Time`
`Tick(duration)` - возвращает __канал__, который периодически отправляет значение `Time`
